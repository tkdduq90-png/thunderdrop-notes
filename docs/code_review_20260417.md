# ThunderDrop 코드리뷰 — 2026-04-17 세션 커밋 7건

> 대상 브랜치: master (unpushed, origin 대비 7 commits ahead)  
> 검토 기준: 의도한 버그만 정확히 타겟, 주변 코드 일관성, 스코프/변수 안전성, 회귀 리스크  
> 수정·커밋 금지

---

## 커밋 1: `090bece` — fix(thumbnail): Thumnail C2 → C 파일명 수정

### diff 요약
```
thumbnail_service.py:49        C2.png → C.png
yacha_thumbnail_variants.py:26 C2.png → C.png
```

### 발견 이슈

없음.

두 파일에 독립적으로 정의된 `YACHA_C_INIT_IMAGE` 상수가 동시에 수정됨.  
(`thumbnail_service.py` = os.path.join 방식, `yacha_thumbnail_variants.py` = raw string 방식 — 기존 아키텍처 문제, 이번 커밋 범위 외)  
`Thumnail C.png` 실제 존재 확인 ✅ (636×356, RGBA, 438KB)

### 안전성 평가
🟢 **안전** — 1줄 × 2파일, 의미 명확, FileNotFoundError 직접 해소. py_compile PASS.

### 권고
없음. (C-3 진단에서 확인된 `Thumnail C.png` 자체 테두리 문제는 별도 수정 이슈)

---

## 커밋 2: `0dc8332` — fix(3crow): villain arc purposes/style/descriptions 제거

### diff 요약
```python
# purposes 리스트에서 "villain arc" 제거
- "purposes": ["night drive","night drive","night drive","car music","villain arc"],
+ "purposes": ["night drive","night drive","night drive","car music"],

# purpose_style에서 "villain arc" 키/값 제거
- "villain arc": "dark rise, power surge, cursed awakening, dominant force",

# descriptions에서 "villain arc" 키/값 제거
- "villain arc": "💀 MEGA BASS for the dark side — no rules, no mercy.\n...",
```

### 발견 이슈

없음.

3군데 (purposes, purpose_style, descriptions) 완전 제거. `hashtags` dict에 villain arc 키 원래 없었음 (확인됨). 관련 코드에서 `purpose_style[purpose]`, `descriptions[purpose]` 조회 시 KeyError 발생 조건 없음 — purposes 리스트에서 선택된 값이 style/desc dict에 반드시 존재함을 전제로 하는데, 제거 후 purposes와 style/desc가 일치 ✅

### 안전성 평가
🟢 **안전** — 3줄 삭제, 순환 참조 없음. py_compile PASS.

### 권고
없음.

---

## 커밋 3: `c6c1b73` — fix(3crow): upload_schedule.json villain arc 설명 교체

### diff 요약
- purpose: `"gym motivation"` → `"night drive"`
- description: gym motivation 문구 → 3CROW night drive 표준 문구
- title: `"🎵 TECHNO MIX 🎧 | GYM MOTIVATION..."` → `"Late Night Techno 2026 🌃 | MEGA BASS | 3CROW"`
- genre: `"techno"` → `"industrial techno"`
- upload_time: `"2026-03-25 21:00:00"` → `"2026-04-14 21:00:00"`
- 추가: `hashtags`, `title_B`, `title_C`, `suno_style_prompt` 필드

### 발견 이슈

**주의 사항 (이슈 아님, 메모)**:  
`suno_style_prompt` 값에 `"dark rise, power surge, cursed awakening, dominant force"` 포함.  
이 필드는 Suno 음악 생성 힌트로, YouTube 메타데이터(title/description/hashtags)에 직접 노출되지 않음.  
업로드 파이프라인에서 suno_style_prompt를 SEO 필드로 사용하지 않는 구조 확인 ✅ (session 기록 참조)  
villain arc SEO 오염 경로 없음.

### 안전성 평가
🟢 **안전** — JSON 구조 유효, 파이프라인 필드 매핑 호환, EOL 정상화 (기존 no-newline-at-EOF → 정상).

### 권고
없음. (upload_time 2026-04-14가 이미 과거이므로 해당 영상 스케줄 재검토는 마스터 오 판단 사항)

---

## 커밋 4: `76b3878` — fix(ab-tester): Step G Popen stdout/stderr 리다이렉트 ★ 최고 리스크

### diff 요약
```python
# 추가된 코드
ab_log_path = rf"C:\ThunderDrop\logs\ab_bg_{channel_key}.log"
ab_log_file = open(ab_log_path, "a", encoding="utf-8", buffering=1)
ab_env = {**os.environ, "PYTHONUTF8": "1"}
_sp.Popen(
    ["python", "-c",
     f"import time,subprocess,sys; time.sleep(600); "
     f"subprocess.run([...'--register'], "
     f"stdout=sys.stdout, stderr=sys.stderr)"],
    cwd=r"C:\ThunderDrop",
    stdout=ab_log_file,
    stderr=ab_log_file,
    env=ab_env,
)
```

### 발견 이슈

#### 이슈 1 — `ab_log_file` 미닫힘 (경미한 코드 품질 문제, 기능 영향 없음)

`ab_log_file`이 `open()`된 후 `close()`하지 않음.  
예외 발생 경로(`_sp.Popen` 실패): `except` 블록에서도 `ab_log_file.close()` 없음.  
파이프라인은 배치 프로세스로 종료 후 OS가 핸들 회수하므로 **기능 장애 없음**.  
그러나 컨텍스트 매니저(`with open(...) as f`) 패턴 미사용은 코드 품질 관점 아쉬움.

```python
# 권장 패턴 (수정 시 참고)
with open(ab_log_path, "a", encoding="utf-8", buffering=1) as ab_log_file:
    _sp.Popen(..., stdout=ab_log_file, stderr=ab_log_file, env=ab_env)
# with 블록 종료 후 부모 핸들은 닫히지만 자식 핸들은 독립적으로 유지됨 (Windows 동작)
```

#### 이슈 2 — 파일 핸들 부모→자식 상속 동작 확인 (정상 확인)

Windows에서 `Popen(stdout=ab_log_file)` 호출 시:
- Python은 `DuplicateHandle`로 자식에게 독립된 OS 핸들 복사본 전달
- 부모 프로세스 종료 / 부모 핸들 GC 후에도 **자식 핸들은 독립 유효** ✅
- 10분 대기(`time.sleep(600)`) 동안 부모가 종료되어도 자식 쓰기 정상 작동

#### 이슈 3 — sys.stdout 체인 확인 (정상 확인)

```
[부모] stdout=ab_log_file
  └─ [자식 python -c] sys.stdout = ab_log_file (상속됨)
       └─ subprocess.run(studio_ab_tester.py, stdout=sys.stdout, stderr=sys.stderr)
            └─ [손자] stdout = ab_log_file ✅
```

studio_ab_tester.py 출력이 ab_bg_{channel}.log에 기록됨 ✅

#### 이슈 4 — E012 UnicodeEncodeError 교훈 반영 확인

| 항목 | 이전 (April 7) | 현재 |
|------|--------------|------|
| 파일 오픈 | 없음 (stdout 버림) | `open(..., encoding="utf-8")` ✅ |
| 환경변수 | 없음 | `PYTHONUTF8="1"` ✅ |
| buffering | 해당 없음 | `buffering=1` (line buffer) ✅ |

#### 이슈 5 — ab_log_path 경로 안전성

`channel_key` = "yacha" 또는 "3crow" (알파벳+숫자만, 특수문자·공백 없음) ✅  
`C:\ThunderDrop\logs\` 디렉토리 존재 확인됨 ✅

#### 이슈 6 — 내부 subprocess.run check=False (기존 동작 유지, 신규 문제 아님)

`subprocess.run([...studio_ab_tester.py...])` — `check=True` 없음, 실패 시 예외 미발생.  
그러나 stdout/stderr가 로그 파일로 redirect되므로 실패 출력은 ab_bg_{channel}.log에서 확인 가능 ✅ (기존 대비 개선됨)

### 안전성 평가
🟡 **검증 필요** — 기능적으로 올바르나 `ab_log_file` 미닫힘 패턴 존재.  
다음 파이프라인 run 후 `logs/ab_bg_yacha.log`에 studio_ab_tester.py 출력 기록 여부로 검증 권장.

### 권고
- 다음 YACHA 파이프라인 run 10분 후 `logs/ab_bg_yacha.log` 내용 확인
- 기능 검증 완료 시 추후 리팩토링으로 `with open(...)` 패턴 전환 검토 (필수 아님)

---

## 커밋 5: `9b64f00` — fix(shorts): raw_bg A_raw 하드코딩 → variant_key_raw

### diff 요약
```python
- raw_bg = thumb_variants.get("A_raw", "")
+ raw_bg = thumb_variants.get(f"{variant_key}_raw", "")
```

### 발견 이슈

없음.

**variant_key 스코프 확인:**  
`master_pipeline.py:1050`: `variant_key = ["A","B","C"][short_sub_idx] if short_sub_idx < 3 else "A"`  
`master_pipeline.py:1112`: `raw_bg = thumb_variants.get(f"{variant_key}_raw", "")` — 동일 함수 내, 62라인 후 참조 ✅

**thumb_variants B_raw / C_raw 키 존재 확인:**  
`thumbnail_adapter._variants_dict()`: `"B_raw": ts.B.raw_path if ts.B.success else ""`  
생성 실패 시 `""` 반환 → `raw_bg = ""` → line 1118 fallback `raw_bg = bg_image` ✅

**variant_key 범위:**
- short_sub_idx=0 → "A" → `"A_raw"` ✅
- short_sub_idx=1 → "B" → `"B_raw"` ✅
- short_sub_idx=2 → "C" → `"C_raw"` ✅
- short_sub_idx≥3 → "A" (방어 코드) ✅

### 안전성 평가
🟢 **안전** — 1줄 수정, f-string 변수 스코프 확인, 키 누락 fallback 정상.

### 권고
없음.

---

## 커밋 6: `5d4af76` — fix(shorts): min(2) → min(3)

### diff 요약
```python
- top_wavs = _select_top_rms_wavs(wav_files, n=min(2, len(wav_files)))
+ top_wavs = _select_top_rms_wavs(wav_files, n=min(3, len(wav_files)))
```

### 발견 이슈

없음.

**`_select_top_rms_wavs` n 처리 확인 (`master_pipeline.py:1009–1033`):**  
`scores[:n]` — n=2이면 2개, n=3이면 3개, n > len(scores)이면 전체 반환 (Python 슬라이싱 안전) ✅

**for 루프 동적 대응:**  
`for short_idx, wav in enumerate(top_wavs)` — top_wavs 크기에 자동 대응  
`total_shorts=len(top_wavs)` — Short 1/2/3 표시에 올바르게 사용 ✅

**short_sub_idx=2 (Short C) 처리:**  
`variant_key = ["A","B","C"][2]` = "C" → thumbnail "C" variant 선택 ✅  
`build_shorts(short_sub_idx=2, total_shorts=3)` — 정상 호출 ✅

**wav 파일 2개만 있는 경우:**  
`n=min(3, 2)` = 2 → `_select_top_rms_wavs(wav_files, n=2)` → 2개 반환 → Short A/B만 생성 (정상)

### 안전성 평가
🟢 **안전** — 1줄 수정, 동적 처리 확인, 경계 조건(wav < 3) 안전.

### 권고
없음.

---

## 커밋 7: `0c03df8` — chore(yacha): VILLAIN ARC MOOD_FALLBACKS 제거

### diff 요약
```python
- "VILLAIN ARC", "DEMON MODE", "DARK SIDE", "BEAST MODE", "NIGHT HUNTER",
+ "DEMON MODE", "DARK SIDE", "BEAST MODE", "NIGHT HUNTER",
  "HAN UNLEASHED", "CURSED BLOOD", "SHADOW REIGN", "IRON WILL", "WRATH",
```

### 발견 이슈

없음.

**구문 유효성:**  
`"NIGHT HUNTER",` ← 쉼표 있음 → 두 번째 줄과 정상 연결 ✅  
`"WRATH",` ← trailing comma 있음 → valid Python ✅

**리스트 크기:** 10개 → 9개.  
사용처 `thumbnail_builder.py:179`: `fallbacks = YACHA_MOOD_FALLBACKS`  
→ `random.choice(fallbacks)` — 9개 비어있지 않은 리스트, IndexError 없음 ✅

**py_compile:** PASS ✅

### 안전성 평가
🟢 **안전** — 1줄 수정, 구문 확인, 사용처 IndexError 불가.

### 권고
없음.

---

## 종합 판정

### 커밋별 요약

| 커밋 | 설명 | 판정 |
|------|------|------|
| `090bece` | C2 → C 파일명 수정 (2파일) | 🟢 안전 |
| `0dc8332` | villain arc purposes/style/desc 제거 | 🟢 안전 |
| `c6c1b73` | upload_schedule.json 설명 교체 | 🟢 안전 |
| `76b3878` | Popen stdout/stderr → ab_bg 로그 리다이렉트 | 🟡 검증 필요 |
| `9b64f00` | raw_bg A_raw 하드코딩 → variant_key_raw | 🟢 안전 |
| `5d4af76` | Shorts 생성 수 2 → 3 | 🟢 안전 |
| `0c03df8` | VILLAIN ARC MOOD_FALLBACKS 제거 | 🟢 안전 |

### 최종 판정: **푸시 OK ✅**

🔴 이슈: **0건**  
🟡 이슈: **1건** — `76b3878` (`ab_log_file` 미닫힘, 기능 장애 아님)

🟡 항목 검증 방법:
- 다음 YACHA 파이프라인 완료 10분 후 `logs/ab_bg_yacha.log` 파일 생성 및 내용 확인
- studio_ab_tester.py 출력이 파일에 기록되면 ✅

조건: 마스터 오 명시적 승인 후 push 진행.

---

*생성: 2026-04-17 | 수정·커밋 금지*
