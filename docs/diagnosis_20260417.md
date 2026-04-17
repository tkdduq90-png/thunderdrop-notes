# ThunderDrop 진단 보고서 — 2026-04-17

> 대상 로그: `logs/2026-04-16_yacha.log` (파이프라인 22:39~01:07)  
> AB 로그: `logs/ab_bg_yacha.log` (타임스탬프 Apr 7 — 오늘 로그 아님)  
> 조사 기준: 수정 금지, 커밋 금지

---

## ① Thumnail C2.png FileNotFoundError ❌ 확정 버그

### 로그 증거
```
[ERROR] Leonardo v2 실패: [Errno 2] No such file or directory:
        'C:\ThunderDrop\YACHA\assets\Thumnail C2.png'
```
총 4회 발생: playlist (file_idx=0), single (file_idx=100), shorts 1 (file_idx=200), shorts 2 (file_idx=201)

결과: 모든 video_type에서 `success=False`, `✅ ThumbnailSet 완료: ... success=False`

### 추가 조사
- `YACHA/assets/` 실제 파일 목록 확인: **`Thumnail C.png`는 존재, `Thumnail C2.png`는 없음**
- `thumbnail_service.py:49`:
  ```python
  YACHA_C_INIT_IMAGE = os.path.join(YACHA_ASSETS_DIR, "Thumnail C2.png")
  ```
  파일명 불일치 (코드에 "C2", 실제 파일은 "C")

### 원인 가설
파일을 `Thumnail C.png` → `Thumnail C2.png`로 교체하거나 반대로 코드 수정이 필요한 상태.  
이전 어느 시점에 파일명이 변경되었거나, 코드에서 잘못 참조됨.

### 영향
- C 변형 썸네일 생성 불가 → AB 테스트에서 C 타입 없이 A/B만 존재
- Shorts는 C 실패 시 raw 이미지 폴백: `🖼️ Shorts 배경: raw 이미지 사용 → A_xxx_raw.jpg`
- Playlist/Single은 raw 이미지 사용: `🖼️ 배경: 썸네일 → A_0_raw.jpg`

---

## ② LEONARDO_API_KEY 미설정 ⚠️ WARNING

### 로그 증거
```
[WARNING] ⚠️ LEONARDO_API_KEY env var 미설정 → fallback 사용  (라인 273)
```
1회 발생 (파이프라인 시작 시 전역 체크로 추정).

### 추가 조사
Fallback 동작 중: B 썸네일은 정상 생성됨.  
`B_0.jpg (1344, 768)`, `B_100.jpg (1344, 768)`, `B_200.jpg (768, 1344)`, `B_201.jpg (768, 1344)` 모두 저장 확인.

### 원인 가설
환경변수 `LEONARDO_API_KEY` 미설정. Fallback이 존재하므로 A/B 썸네일 생성은 정상이나,  
C 썸네일 실패(이슈 ①)와 합쳐져 AB 테스트 불완전.

---

## ③ Shorts B 썸네일 해상도 — 정상 ✅ (이전 수정 확인)

### 로그 증거
```
raw 저장: B_200.jpg (768, 1344)   ← 9:16 Shorts 1
raw 저장: B_201.jpg (768, 1344)   ← 9:16 Shorts 2
```
이전 수정(fix: Shorts A 썸네일 16:9→9:16)이 정상 적용됨.  
이슈 없음.

---

## ④ Shorts 썸네일 업로드 스킵 ⚠️ 주의

### 로그 증거
```
Shorts 썸네일 업로드 스킵 — 첫 프레임 사용 (video_id=Uv2pDOvA1DM)  ← Short A
Shorts 썸네일 업로드 스킵 — 첫 프레임 사용 (video_id=kho2z4aVPlY)  ← Short B
```

### 추가 조사
로그 불충분 — 코드에서 Shorts 썸네일 스킵 조건 확인 필요.

### 원인 가설
YouTube Shorts는 커스텀 썸네일 적용 시 첫 프레임 우선이거나,  
코드 내 `video_type == "shorts"` 분기에서 의도적으로 스킵 처리.  
썸네일 파일 존재 여부와 무관하게 스킵됨 → 의도된 동작인지 버그인지 확인 필요.

---

## ⑤ AB 테스터 로그 — 오래된 데이터, 오늘 결과 불명

### 로그 증거
`ab_bg_yacha.log` 타임스탬프: **Apr 7 20:10** (10일 전)  
내용: UnicodeDecodeError + TimeoutExpired (timeout=300초)

### 추가 조사
현재 `master_pipeline.py:1483-1489`:
```python
_sp.Popen(
    ["python", "-c",
     f"import time,subprocess; time.sleep(600); "
     f"subprocess.run(['python', r'C:\\ThunderDrop\\scripts\\studio_ab_tester.py', "
     f"'--channel', '{channel_key}', '--register'])"],
    cwd=r"C:\ThunderDrop",
)
```
`capture_output=True, text=True, timeout=300` **없음** → ab_bg_yacha.log에 오늘 출력 기록 안 됨.

파이프라인 완료: 01:07:50 → AB 테스터 실행 예정: 01:17:50  
실행 후 결과 로그 없음 (Popen에 stdout/stderr 미연결).

### 원인 가설
- Apr 7 로그는 이전 코드 버전의 오류 (그 당시 capture_output+timeout 사용)
- 현재 코드는 Popen으로 변경되어 오류 없이 실행되나, 결과 확인 불가
- AB 테스터 실제 등록 성공 여부: **Studio 직접 확인 필요**

---

## ⑥ 3Crow villain arc 코드 잔류 ❌ SEO 위험

### 로그 증거
로그 없음 (3Crow 파이프라인 미실행).

### 추가 조사
**`master_pipeline.py:103`**:
```python
"purposes": ["night drive","night drive","night drive","car music","villain arc"],
```
**`master_pipeline.py:107, 114`**:
```python
"villain arc": "dark rise, power surge, cursed awakening, dominant force",
"villain arc": "💀 MEGA BASS for the dark side — no rules, no mercy.\n
               Heavy drops built for those who chose the villain path.\nEmbrace the darkness.",
```
**`3Crow/upload_schedule.json`** (현재 예약 영상):
```json
{
  "purpose": "villain arc",
  "description": "💀 MEGA BASS for the dark side — no rules, no mercy.\n
                  Heavy drops built for those who chose the villain path.\nEmbrace the darkness.",
  "upload_time": "2026-04-14 21:00:00"
}
```
YACHA SEO 클린업(2026-04-14) 당시 3Crow 코드/데이터는 처리되지 않음.

### 원인 가설
YACHA 전용 클린업이었고, 3Crow purposes/descriptions는 독립 설정이라 동시 수정 누락.  
현재 upload_schedule.json의 villain arc 영상은 이미 예약 완료됐을 가능성 있음 (2026-04-14 21:00 예약).

---

## ⑦ Hook 제목 0개 생성 (일부) ⚠️

### 로그 증거
```
🎣 Hook 제목 생성: gym phonk mix (type=B)
📦 캐시 히트: f682efdf632b6d5867012b0fab0c797d.json
✅ Hook 제목 0개 생성 완료   ← Short 1 type B
🎣 Hook 제목 생성: gym phonk mix (type=C)
📦 캐시 히트: f682efdf632b6d5867012b0fab0c797d.json
✅ Hook 제목 0개 생성 완료   ← Short 1 type C
```
Short 2 type B는 1개 정상 생성됨.

### 추가 조사
로그 불충분 — Claude 응답 내용 미기록.  
`HOOK_CLICHE_BLACKLIST` 항목: `never quit, no pain no gain, push harder, feel the bass, built for, made for`

### 원인 가설
캐시 히트 상태에서 동일 niche 중복 요청 시 Claude가 blacklist 기준 초과로 반환 0개 처리.  
Short 1 B/C 제목 없음 → AB 테스트에서 해당 variant 제목 누락.

---

## ⑧ SEO 금지 키워드 — mythology 위험 가능성 ⚠️

### 로그 증거
`prompt_builder.py:429`:
```python
"powerful/dark/demonic Korean mythology energy" if is_yacha
```

### 추가 조사
- `YACHA_BLACKLIST`: `BRAZILIAN, FUNK, MURDER, METAMORPHOSIS, SLOWED, TOP X, SONGS MIX, TRACKS MIX`  
  → **mythology 블랙리스트 미포함**
- 이 문자열은 YACHA 곡 제목 generation의 `title_theme` 변수로 사용  
- 실제 생성된 트랙명 확인: `IRON DOKKAEBI, GWISIN RAGE, SHADOWBEAST RISING, FORGOTTEN SHAMAN, BLOOD HAETAE, ETERNAL JEOSEUNG, DARK MUYEOK, BONE CRUSHER DEITY, POSSESSED WARRIOR, SAMSHIN WRATH, VENGEFUL CHEONYEO, UNDERWORLD GRIND, SAVAGE INMANG, NINE TAILED FURY, CURSED MUSIN, DEATH DRUM RISE, FALLEN SINSEON, IMMORTAL GWISHIN, IRON SOUL RITUAL, DEMON BAEK`  
- 곡명에 mythology 단어 직접 노출 없음. 하지만 SEO 제목/설명에 "mythology" 텍스트가 삽입될 가능성은 코드상 잠재적.

### 원인 가설
title_theme은 Claude 프롬프트의 스타일 힌트로만 사용되고, 출력에 직접 복사되지 않음.  
오늘 생성된 트랙명 기준으로는 mythology 단어 미포함. **단기 위험 낮음**, 장기 모니터링 필요.

---

## 요약 판정

| 이슈 | 심각도 | 상태 |
|------|--------|------|
| ① Thumnail C2.png 없음 | 🔴 HIGH | 즉시 수정 필요 |
| ② LEONARDO_API_KEY 미설정 | 🟡 MED | 환경변수 추가 필요 |
| ③ Shorts B 해상도 | 🟢 OK | 이전 fix 정상 적용 |
| ④ Shorts 썸네일 업로드 스킵 | 🟡 MED | 의도/버그 확인 필요 |
| ⑤ AB 테스터 결과 불명 | 🟡 MED | Studio 직접 확인 필요 |
| ⑥ 3Crow villain arc 코드 잔류 | 🔴 HIGH | 코드+데이터 수정 필요 |
| ⑦ Hook 제목 0개 (일부) | 🟡 MED | 캐시 충돌, 재생성 고려 |
| ⑧ mythology 키워드 위험 | 🟢 LOW | 오늘 출력 기준 안전, 모니터링 |

---

*생성: 2026-04-17 | 수정·커밋 금지*
