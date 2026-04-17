# ThunderDrop 진단 보고서 — 2026-04-17 (Part 2)

> 대상: B-1 (Step G AB 자동 실행 실패), B-2 (YACHA Shorts A/B/C 동일 raw_bg + Short C 미생성)  
> 조사 기준: 수정 금지 (진단만)

---

## B-1: master_pipeline Step G (AB 자동 실행) 실패 원인

### Step G 전체 코드 (`master_pipeline.py:1477–1490`)

```python
# 5) 10분 후 AB 테스트 자동 등록 (백그라운드)
try:
    import subprocess as _sp
    logger.info(f"\n🔬 [{cfg['name']}] AB 테스트 등록 예약 (10분 후)...")
    _sp.Popen(
        ["python", "-c",
         f"import time,subprocess; time.sleep(600); "
         f"subprocess.run(['python', r'C:\\ThunderDrop\\scripts\\studio_ab_tester.py', "
         f"'--channel', '{channel_key}', '--register'])"],
        cwd=r"C:\ThunderDrop",
    )
    logger.info(f"   ✅ AB 테스트 등록 프로세스 시작됨 (10분 후 실행)")
except Exception as e:
    logger.error(f"   ⚠️ AB 테스트 예약 실패: {e}", exc_info=True)
```

### Popen 구조 점검

| 항목 | 현재 값 | 문제 |
|------|---------|------|
| `stdout=` | 없음 (상속) | 출력이 어디로도 기록되지 않음 |
| `stderr=` | 없음 (상속) | 에러도 어디로도 기록되지 않음 |
| `env=` | 없음 (상속) | PYTHONUTF8=1 등 파이프라인 환경변수 전달됨 (문제 없음) |
| `cwd=` | `C:\ThunderDrop` | 정상 |
| timeout | 없음 | Popen은 타임아웃 파라미터 없음 (무기한 대기) |
| 내부 subprocess.run | `capture_output=` 없음 | studio_ab_tester.py 출력 캡처 안 됨 |

### 실행 조건 점검

- `IS_TEST` 플래그: Sheets 로깅 (line 1435, 1445, 1461)은 스킵하지만 **AB Popen은 스킵 조건 없음**
- dry-run 분기: 존재하지 않음 (Step G는 항상 실행됨)
- video_type 필터: 없음 (모든 타입 후 호출)
- 채널 조건: `channel_key` 를 인자로 전달할 뿐, 채널별 스킵 로직 없음

### 로그 증거 (`logs/2026-04-16_yacha.log`)

```
541: 🔬 [YACHA] AB 테스트 등록 예약 (10분 후)...
542: 2026-04-17 01:07:50 [INFO] ✅ AB 테스트 등록 프로세스 시작됨 (10분 후 실행)
```

- **Step G 도달: 확인됨** (line 541–542)
- 파이프라인 완료: 01:07:50
- AB 테스터 예정 실행: 01:17:50

### ab_bg_yacha.log 불일치 원인

```
logs/ab_bg_yacha.log 타임스탬프: Apr 7 20:10
내용: UnicodeDecodeError + subprocess.TimeoutExpired (timeout=300)
```

Apr 7 당시 코드는 `capture_output=True, text=True, timeout=300` 로 subprocess.run을 사용.  
현재 코드는 `Popen(bare)` 으로 변경 → **stdout/stderr 어디에도 redirect 없음** → `ab_bg_yacha.log` 미갱신.  
Apr 7 로그는 이전 버전의 흔적이며, 오늘 실행 결과는 **어떤 파일에도 기록되지 않음**.

### Step G 실행 흐름 (의사코드)

```
[Pipeline 완료 01:07:50]
  └─ Popen 호출 → 즉시 리턴 (비동기)
       ✅ "AB 테스트 등록 프로세스 시작됨" 로그 기록
       [파이프라인 종료]

[10분 후, 별도 Python 프로세스 01:17:50]
  └─ time.sleep(600) 완료
  └─ subprocess.run(studio_ab_tester.py --channel yacha --register)
       stdout: DISCARDED (no capture)
       stderr: DISCARDED (no capture)
       → 성공/실패 여부 확인 불가
```

### 가장 유력한 실패 원인

1. **로그 리다이렉션 없음 (확정)**: 현재 Popen에 `stdout=`/`stderr=` 미설정. 오늘 실행 결과는 어디에도 남지 않음. 실제 등록 성공 여부는 YouTube Studio 직접 확인 필요.

2. **내부 subprocess.run 에러 무시**: `subprocess.run(...)` 은 `check=False` (기본값). studio_ab_tester.py가 예외 발생해도 오류 없이 종료됨.

### 수정 방향 (수정 금지, 참고만)

```python
# 권장: stdout/stderr를 ab_bg_yacha.log로 리다이렉트
log_file = open(r"C:\ThunderDrop\logs\ab_bg_yacha.log", "a", encoding="utf-8")
_sp.Popen(..., stdout=log_file, stderr=log_file, env={**os.environ, "PYTHONUTF8": "1"})
```

---

## B-2: YACHA Shorts raw_bg 동일 (A_raw 고정) + Short C 미생성

### 버그 1 — raw_bg 항상 A_raw 고정 (`master_pipeline.py:1111–1119`)

```python
# ── raw 이미지 경로 (텍스트 없는 원본) — variants_dict에서 가져옴
if channel_key == "yacha":
    raw_bg = thumb_variants.get("A_raw", "")   # ← BUG: variant_key 무시
else:
    raw_bg = ""
if raw_bg and not os.path.exists(raw_bg):
    ...
if not raw_bg:
    raw_bg = bg_image
```

`raw_bg = thumb_variants.get("A_raw", "")` — `variant_key` (A/B/C) 와 관계없이 항상 A_raw 키를 참조.

**로그 증거 (`2026-04-16_yacha.log`):**
```
399: 🖼️ Shorts 배경: raw 이미지 사용 → A_200_raw.jpg   ← Short 1 variant A (정상)
446: 🖼️ Shorts 배경: raw 이미지 사용 → A_201_raw.jpg   ← Short 2 variant B (버그: B_201_raw 이어야 함)
```

Short B 비디오의 배경 이미지가 B_raw가 아닌 A_raw를 사용.

### A/B/C init 파일 경로 비교표

| 구분 | 플리/단곡 | Shorts |
|------|---------|--------|
| **A init** | `base_closeup/rear/front.jpg` (day_idx % 9 로테이션) | 동일 |
| **B init** | `Thumnail B.png` | 동일 |
| **C init** | `Thumnail C.png` (수정 1로 C2→C 픽스됨) | 동일 |
| **비디오 raw_bg** | N/A (영상 없음) | **A_raw 고정 (버그)** |

thumbnail_service.py 의 init_image 선택 자체는 A/B/C 별로 올바르게 분기됨.  
버그는 `master_pipeline.py:1112` — **비디오 배경 이미지 선택 시** variant_key를 무시.

### 버그 2 — Short C 미생성 (`master_pipeline.py:695`)

```python
top_wavs = _select_top_rms_wavs(wav_files, n=min(2, len(wav_files)))
```

주석: "RMS 에너지 상위 3곡 각각 A/B/C variant"  
코드: `n=min(2, ...)` → **최대 2곡 선택**, Short C(short_sub_idx=2) 호출 자체가 없음.

**로그 증거:**
```
378: 🏆 RMS Top 2: ['005_BLOOD HAETAE_v1_230404.wav', '013_SAVAGE INMANG_v2_234306.wav']
425: ✅ Short A done: short_01_01_BLOOD_HAETAE_A.mp4
470: ✅ Short B done: short_01_02_SAVAGE_INMANG_B.mp4
   (Short C 로그 없음 — 선택 자체 안 됨)
```

업로드 결과: `Uv2pDOvA1DM (short A)`, `kho2z4aVPlY (short B)` — Short C 업로드 없음 ✅ (예상과 일치).

### 버그 요약

| # | 파일:라인 | 내용 | 영향 |
|---|----------|------|------|
| B-2-1 | `master_pipeline.py:1112` | `thumb_variants.get("A_raw")` 고정 → variant_key 무시 | Short B/C 비디오 배경이 A_raw 사용 |
| B-2-2 | `master_pipeline.py:695` | `n=min(2, ...)` → Short C 선택 안 됨 | Short C 영상+업로드 미생성 |

### 수정 방향 (수정 금지, 참고만)

```python
# B-2-1 수정
raw_bg = thumb_variants.get(f"{variant_key}_raw", "")

# B-2-2 수정
top_wavs = _select_top_rms_wavs(wav_files, n=min(3, len(wav_files)))
```

두 수정 모두 1줄씩, 독립적. 별도 커밋 권장.

---

## 추가 확인 (B-3): Shorts 썸네일 업로드 스킵

진단 v2 이슈 ④ 추적 결과:

```python
# master_pipeline.py:1299-1302
if is_shorts:
    logger.info(f"Shorts 썸네일 업로드 스킵 — 첫 프레임 사용 (video_id={video_id})")
```

코드 확인: Shorts는 `is_shorts=True` 분기에서 커스텀 썸네일 업로드를 의도적으로 스킵.  
YouTube Shorts는 커스텀 썸네일 API 제한으로 인한 의도된 동작으로 판단됨.  
버그 아님 ✅

---

*생성: 2026-04-17 | 수정·커밋 금지*
