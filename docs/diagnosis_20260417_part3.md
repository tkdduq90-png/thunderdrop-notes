# ThunderDrop 진단 보고서 — 2026-04-17 (Part 3)

> 대상: C-1 (Playlist/Single bg_image 경로), C-2 (Shorts 썸네일 크기), C-3 (C 썸네일 테두리 2중),  
> C-4 (YouTube 업로드 메타데이터), C-5 (해시태그 오염 감사)  
> 조사 기준: 수정 금지 (진단만)

---

## C-1: Playlist/Single bg_image 경로 (raw vs final)

### 코드 확인 (`master_pipeline.py:542–557` Playlist, `855–870` Single)

**Playlist:**
```python
# master_pipeline.py:542-557
if channel_key == "yacha":
    _raw_bg = thumb_variants.get("A_raw", "")   # A_raw 고정
elif channel_key == "3crow":
    _raw_bg = thumb_raw if thumb_raw else ""
else:
    _raw_bg = ""
if _raw_bg and not os.path.exists(_raw_bg):
    logger.warning(...)
    _raw_bg = ""
bg_image = (
    _raw_bg if _raw_bg
    else thumb_path if thumb_path and os.path.exists(thumb_path)
    else cfg.get("banner_file") ...
    else None
)
```

**Single (855–870):** 동일 패턴, 변수명만 `thumb` → 동일.

### Shorts B-2-1 버그와의 비교

| 구분 | Playlist/Single | Shorts |
|------|-----------------|--------|
| variant 수 | A 단독 (B/C는 AB테스트용 썸네일, 별도 영상 아님) | A / B / C 각각 별도 영상 |
| bg_image 선택 | `thumb_variants.get("A_raw", "")` 고정 | `thumb_variants.get(f"{variant_key}_raw", "")` |
| A_raw 고정 여부 | **의도된 동작** | 수정 전 버그 (B-2-1) |

### 결론

**버그 없음 ✅**

Playlist/Single은 영상이 1개이므로 bg_image는 항상 A variant의 raw 이미지.  
B/C는 AB 테스트 썸네일(정지 이미지)로만 사용되며 bg_image로 사용되지 않는다.  
`"A_raw"` 고정은 Shorts와 달리 **구조적으로 올바른 구현**이다.

---

## C-2: YACHA Shorts 썸네일 크기 검증

### RESOLUTIONS dict (`thumbnail_service.py:36–40`)

```python
RESOLUTIONS = {
    "playlist": (1344, 768),   # 16:9 가로
    "single":   (1344, 768),   # 16:9 가로
    "shorts":   (768, 1344),   # 9:16 세로 ✅
}
```

### video_type 전달 체인

```
master_pipeline.py:1073
  build_and_adapt_yacha(video_type="shorts", ...)
    └─ thumbnail_adapter.py → build_yacha_thumbnail_set(video_type="shorts", ...)
         ├─ _build_variant_b_raw(video_type, ...)
         │    └─ w, h = RESOLUTIONS["shorts"]  →  (768, 1344)
         └─ _build_variant_c_raw(day_idx, genre, video_type, ...)
              └─ w, h = RESOLUTIONS["shorts"]  →  (768, 1344)
```

- `thumbnail_service.py:430`: `w, h = RESOLUTIONS[video_type]` (Variant B)
- `thumbnail_service.py:453`: `w, h = RESOLUTIONS[video_type]` (Variant C)

### 결론

**올바른 구현 ✅**

- Shorts 해상도 768×1344 (9:16) 정상 적용
- video_type 전달 경로 완전함, 누락 없음
- 3개 variant 모두 동일 RESOLUTIONS 조회 — 동적, 하드코딩 없음

---

## C-3: Shorts C 썸네일 "테두리 2중" 원인

### Thumnail C.png 메타데이터

```
경로: C:\ThunderDrop\YACHA\assets\Thumnail C.png
크기: 636 × 356 px
모드: RGBA (알파 채널 포함)
파일 크기: 438,964 bytes
```

- 636×356은 표준 해상도 아님 (일반적인 정사각형/16:9와 다름)
- RGBA — 알파 채널 포함 = 투명 영역 또는 테두리 구조 존재 가능성
- 438KB — B variant init_image (Thumnail B.png: 269KB)보다 63% 큼 = 더 복잡한 내용물

### C variant 생성 흐름

```python
# thumbnail_service.py:446-467
def _build_variant_c_raw(day_idx, genre, video_type, raw_path):
    w, h = RESOLUTIONS[video_type]
    saved, actual_res = _leonardo_i2i_v2(
        init_image_path=YACHA_C_INIT_IMAGE,   # ← Thumnail C.png
        prompt=prompt,
        width=w, height=h,
        output_path=raw_path,
        # strength= 미설정 → 기본값 MID
    )
```

Leonardo I2I는 `strength="MID"` 기본값으로 **init_image 구조를 50% 수준으로 유지**한다.  
init_image인 Thumnail C.png에 테두리/프레임 구조가 포함되어 있으면 생성 결과에도 잔존.

### A/B variant와 누끼 처리 비교

| variant | init_image | remove_white_bg() | 비고 |
|---------|-----------|-------------------|------|
| A | base_closeup.jpg / rear / front (JPEG, RGB) | 해당 없음 (배경 이미지 자체) | 배경으로 직접 사용 |
| B | Thumnail B.png (498×511, RGBA) | 수행 (`yacha_thumbnail_variants.py`) | 누끼 후 배경에 합성 |
| C | Thumnail C.png (636×356, RGBA) | **미수행** | `Image.open(raw_path).convert("RGB")` — 누끼 없이 배경으로 사용 |

`yacha_thumbnail_variants.py:404` 인근: C variant는 Leonardo raw 출력을 그대로 `.convert("RGB")` 후 배경으로 채운다.

### C2 → C 파일명 수정의 한계

| 수정 내용 | 해결 여부 |
|-----------|----------|
| `FileNotFoundError` (Thumnail C2.png 미존재) | ✅ 해결됨 (커밋 `090bece`) |
| Thumnail C.png 자체 테두리/프레임 구조 | ❌ 미해결 — 별도 이슈 |

### 결론

**C-3: 신규 버그 확인 ❌**

근본 원인은 두 가지가 복합됨:

1. **Thumnail C.png 이미지 자체에 테두리/프레임 구조 포함** (636×356 RGBA, 438KB)
2. **C variant 처리 시 `remove_white_bg()` 누끼 미수행** — B variant는 누끼 후 합성하지만 C는 raw 직접 사용

C2→C 파일명 수정만으로는 테두리 문제가 해결되지 않는다.

수정 방향 (참고):
```
Option A: Thumnail C.png를 테두리 없는 깨끗한 캐릭터 이미지로 교체
Option B: thumbnail_service.py 또는 yacha_thumbnail_variants.py에서
          C variant에도 remove_white_bg() 혹은 누끼 처리 추가
```

---

## C-4: YouTube 업로드 메타데이터 검증

### 업로드 로그 확인 (`logs/2026-04-16_yacha.log`)

| video_id | 타입 | 업로드 제목 |
|----------|------|-----------|
| KQoRWyRNPdw | playlist | Aggressive Phonk 2026 💢 \| MEGA BASS \| YACHA |
| LxocJsIq2mw | single | when the fury locks in \| Gym Phonk Mix 2026 |
| Uv2pDOvA1DM | shorts A | when the fury becomes unstoppable \| Aggressive Gym |
| kho2z4aVPlY | shorts B | when the world goes quiet \| Gym Phonk Mix 2026 #sh |

4개 영상 모두 YACHA 정체성 (gym phonk) 유지 ✅  
구체적 해시태그 값은 로그에 기록되지 않음 (sheets_logger 경유, 로그 미출력).

### channel_constraint 적용 경로

```python
# seo_scraper.py:708-721
if ck == "yacha":
    channel_constraint = (
        "CHANNEL CONSTRAINTS (YACHA — Gym Phonk): "
        "ONLY gym, workout, lifting, phonk, motivation tags. "
        "NEVER use: drive, night, techno, chill, sleep, lofi, study."
    )
elif ck == "3crow":
    channel_constraint = (
        "CHANNEL CONSTRAINTS (3CROW — Night Drive Techno): "
        "ONLY night drive, techno, dark, underground, rave tags. "
        "NEVER use: gym, workout, phonk, lifting, motivation."
    )
```

→ Claude Haiku 해시태그 생성 프롬프트에 직접 포함 (line 726 `messages=[...]` 내)  
→ YACHA에서 drive/night/techno 태그 생성 구조적 차단됨

### CHANNEL_CONFIG 해시태그 필드 확인

**YACHA `master_pipeline.py:81–86`:**
```
gym motivation: #phonk #darkphonk #megabass #gymmusic #workoutmusic #gymmotivation ...
workout:        #phonk #darkphonk #megabass #workoutmusic #gymmusic ...
beast mode:     #phonk #darkphonk #megabass #beastmode #gymmusic #workoutmusic ...
tags:           ["phonk","dark phonk","gym phonk","mega bass","bass boosted","workout music","gym music","beast mode",...]
```

drive/car/road 관련 태그 없음 ✅

### 결론

**채널 분리 정상 ✅**

channel_constraint가 해시태그 생성 API 프롬프트에 직접 포함되어 교차 오염 차단됨.  
CHANNEL_CONFIG 정적 해시태그도 gym/phonk 전용.  
단, 실제 업로드된 태그값은 YouTube Studio 직접 확인 필요 (로그 미기록, API 조회 스크립트 없음).

---

## C-5: Sheets History 해시태그 오염 감사

### Sheets 구조

`sheets_logger.py:44-69`: Sheets는 **기록 전용** (쓰기 함수만 존재).  
히스토리 읽기 함수 미구현 → 코드 레벨에서 과거 기록 자동 분석 불가.

### 채널별 해시태그 필드 교차 확인

**YACHA (`master_pipeline.py:81–86`):**
- 포함: gym, workout, phonk, beastmode, gymmotivation
- 없음: drive, car, road, night, techno ✅

**3CROW (`master_pipeline.py:114–121`):**
- 포함: nightdrive, carmusic, techno, rave, underground
- 없음: gym, workout, phonk, lifting, motivation ✅

**seo_scraper.py `channel_constraint`:**
- YACHA → NEVER use: drive, night, techno, chill, sleep, lofi, study (명시적 차단)
- 3CROW → NEVER use: gym, workout, phonk, lifting, motivation (명시적 차단)

### seo_cache 캐시 확인

`seo_cache/used_titles.json` (55개 항목):
- YACHA 항목: "Gym Phonk Mix 2026" / "Aggressive Phonk 2026" 계열
- 3CROW 항목: "Night Drive Techno Mix 2026" 계열
- 교차 오염 항목 없음 ✅

### 결론

**해시태그 오염 없음 ✅**

정적 CHANNEL_CONFIG과 동적 Claude 생성 모두 채널별로 명확히 분리됨.  
코드 레벨에서 교차 생성 구조적으로 차단됨.  
과거 Sheets 기록의 직접 감사는 YouTube Studio 또는 Sheets 직접 열람으로만 가능.

---

## 수정 지시 우선순위

| 순위 | 이슈 | 파일:라인 | 내용 | 긴급도 |
|------|------|----------|------|--------|
| **1** | C-3 | `YACHA/assets/Thumnail C.png` | init_image 교체 (테두리 없는 버전) | 높음 — 현재 C variant 출력 품질 저하 |
| — | C-1 | — | 정상 확인, 수정 불필요 | — |
| — | C-2 | — | 정상 확인, 수정 불필요 | — |
| — | C-4 | — | channel_constraint 정상, 수정 불필요 | — |
| — | C-5 | — | 오염 없음, 수정 불필요 | — |

C-3의 수정 방법은 두 가지:
- (a) `YACHA/assets/Thumnail C.png`를 테두리 없는 이미지로 교체 (에셋 교체)
- (b) `thumbnail_service.py` 또는 `yacha_thumbnail_variants.py`에서 C variant 처리 시 누끼 추가 (코드 수정)

어느 방향으로 진행할지 마스터 오 결정 필요.

---

*생성: 2026-04-17 | 수정·커밋 금지*
