# ThunderDrop 에러 로그
*최종 업데이트: 2026-04-09*
*이전 대화기록 전수 조사 기반 정리*

---

## 사용법
- 에러 발생 시 이 파일 먼저 검색
- 새 에러 해결 후 아래 형식으로 추가
- 형식: `에러명 | 원인 | 해결 | 파일`

---

## ✅ 해결된 에러 목록

---

### [E001] Docker suno-api hCaptcha 실패
- **증상:** Suno API 서버가 음원 생성 요청 거부
- **원인:** Docker 환경에서 hCaptcha Enterprise 서버사이드 감지 → 렌더링 자체 차단
- **해결:** pyautogui 방식으로 완전 전환 (`suno_bot.py`). Docker 방식 완전 포기
- **파일:** `suno_bot.py`
- **재발 여부:** ❌ (근본 방식 변경)

---

### [E002] FFmpeg Motion Clip - hwaccel + zoompan 충돌
- **증상:** `⚠️ Motion clip failed: Nothing was written into output file. frame=0 fps=0.0 elapsed=0:00:00.10`
- **원인:** STEP A에서 `-hwaccel cuda -hwaccel_output_format cuda` + `zoompan` 필터 동시 사용. zoompan은 CPU 전용 필터라 GPU 메모리 프레임을 처리 불가 → 0 packets
- **해결:** STEP A motion clip에서 `-hwaccel cuda -hwaccel_output_format cuda` 제거. `-c:v h264_nvenc` 인코더는 유지 (디코딩 CPU, 인코딩만 GPU)
- **파일:** `beat_video.py` STEP A
- **주의:** STEP C(EQ overlay)도 동일한 원리로 `-hwaccel_output_format cuda` 제거 필요

---

### [E003] FFmpeg EQ Overlay - CUDA 픽셀 포맷 충돌
- **증상:** `src: cuda / Impossible to convert between the formats supported by the filter`
- **원인:** STEP B loop에서 `codec copy` → nvenc 스트림 그대로 복사 → cuda 픽셀 포맷 메타데이터 유지 → STEP C CPU 필터 처리 불가
- **해결 과정 (여러 시도):**
  1. `-hwaccel none` → CPU 렌더링 1시간 소요 (임시방편)
  2. `filtergraph`에 `[0:v]format=yuv420p[bg_fmt]` 추가 → `-hwaccel_output_format cuda`가 여전히 강제 변환
  3. **최종 해결:** STEP C final_cmd에서 `-hwaccel cuda -hwaccel_output_format cuda` 제거 + STEP B loop를 `codec copy` → `-c:v libx264 -crf 23 -pix_fmt yuv420p`로 변경
- **파일:** `beat_video.py` STEP B, STEP C
- **교훈:** `-hwaccel_output_format cuda` 쓰면 이후 CPU 필터 체인 전부 불가

---

### [E004] FFmpeg showcqt fullhd=0 + custom dimension 충돌
- **증상:** `⚠️ EQ 합치기 실패: fullhd set to 0 but with custom dimension. Error initializing filters`
- **원인:** `showcqt` 필터에서 `fullhd=0`(854x480 강제)과 `s=1280x200`(커스텀 크기) 동시 사용 → 충돌
- **해결:** `fullhd=0,` 제거 (2군데)
- **파일:** `beat_video.py` `_build_circular_eq_filter()`

---

### [E005] ChromeDriver 버전 불일치
- **증상:** `session not created: Chrome instance exited`
- **원인:** `ChromeDriverManager(chrome_type=ChromeType.GOOGLE).install()` 자동 감지 불안정. 현재 Chrome 버전(146.0.7680.178)과 매칭 실패
- **해결:** ChromeDriver 경로 하드코딩
  ```python
  service = Service(r"C:\Users\computer\.wdm\drivers\chromedriver\win64\146.0.7680.178\chromedriver-win32\chromedriver.exe")
  ```
- **파일:** `studio_ab_tester.py` line ~162, `studio_crawler.py` line ~68
- **주의:** Chrome 업데이트 시 버전 수동 변경 필요

---

### [E006] pyautogui FAILSAFE 트리거
- **증상:** `PyAutoGUI fail-safe triggered from mouse moving to a corner of the screen`
- **원인:** Suno 자동화 중 마우스가 화면 모서리 이동 → pyautogui 안전장치 작동
- **해결:** `suno_bot.py` 상단에 `pyautogui.FAILSAFE = False` 추가
- **파일:** `suno_bot.py`

---

### [E007] YACHA/3CROW 채널 token 혼선 (잘못된 채널에 업로드)
- **증상:** 3CROW 선택했는데 YACHA에 영상 업로드됨
- **원인:** 두 채널이 `youtube_token_shared.pickle` 동일 파일 공유. `get_youtube_service()`가 항상 같은 계정 로드
- **해결:** 채널별 별도 토큰 파일 분리
  - YACHA: `youtube_token_yacha.pickle`
  - 3CROW: `3Crow\youtube_token_3crow.pickle`
- **파일:** `master_pipeline.py`, 각 채널 config

---

### [E008] SINGLE_EXTRA_BLACKLIST 중복 정의 (import 덮어쓰기)
- **증상:** 단곡 제목에 "TOP 10", "TRACKS" 같은 금지어가 그대로 생성됨
- **원인:** `master_pipeline.py` line 19에서 `config.py`에서 import 후 line 185에서 로컬 재정의 → 로컬 정의가 import 덮어씀. config.py 버전(MINUTE, TRACKS 등 포함)이 무시됨
- **해결:** 로컬 중복 정의 제거, config.py 단일 소스 유지
- **파일:** `master_pipeline.py`

---

### [E009] 단곡 제목/썸네일 생성 누락
- **증상:** 단곡 업로드 후 Sheets에 title_B, title_C, thumb_B, thumb_C 전부 빈칸
- **원인:** "기존 썸네일 재사용" 로직 추가 시 썸네일 생성 skip 조건 안에 제목 생성까지 묶임
- **해결:** 썸네일 재사용 조건과 제목 생성 로직 분리. 제목 생성은 항상 실행
- **파일:** `master_pipeline.py`

---

### [E010] Google Sheets 셀 한도 초과
- **증상:** `YACHA video_id 추가 시도 → Error: 셀 한도 초과`
- **원인:** Analytics_3CROW 탭: 9997행 × 265열 = 264만 셀 (빈 셀 대부분). Sheets 셀 한도 500만 초과 근접
- **해결:** 탭별 불필요한 행/열 축소
  - Analytics_3CROW: 9997행×265열 → 231행×20열
  - Analytics_YACHA: 9941행×451열 → 217행×20열
- **파일:** 일회성 스크립트 실행 (Python gspread)

---

### [E011] AB 테스트 C 썸네일 업로드 DOM 타이밍 버그
- **증상:** A, B 썸네일은 업로드되는데 C 썸네일만 `+` 버튼 클릭 안 됨
- **원인:** YouTube Studio AB 테스트 다이얼로그에서 C 슬롯의 `input[type='file']`이 "제목 추가" 클릭 전까지 DOM에 존재하지 않음. B 업로드 시점에 file input 목록을 캐싱하면 C 슬롯 input이 없는 상태
- **해결:** C 썸네일 업로드 직전에 file input 목록 재수집 + "제목 추가" 후 C 슬롯 input 나타날 때까지 대기
- **파일:** `studio_ab_tester.py`

---

### [E012] AB 테스트 실행 실패 - cp949 UnicodeEncodeError
- **증상:** `ab_bg_yacha.log`에서 UnicodeEncodeError → AB 테스트 등록 자체가 실행 안 됨
- **원인:** 이모지 포함 제목을 Windows cp949 인코딩 환경에서 출력 시도
- **해결:**
  1. `PYTHONUTF8=1` 환경변수 추가 (자식 프로세스에도 적용)
  2. `with open()` → `open()` 변경 (with 블록 종료 시 fd 닫히는 문제 해결)
- **파일:** `master_pipeline.py` Step G (AB 테스트 백그라운드 실행 부분)

---

### [E013] PCM 파일명 충돌
- **증상:** 여러 영상 동시 렌더링 시 PCM 파일 덮어쓰기 → 오디오 오염
- **원인:** PCM 임시 파일명이 고정값 사용
- **해결:** PCM 파일명에 고유 ID(날짜폴더명 등) 포함
- **파일:** `master_pipeline.py`

---

### [E014] thumbnail_builder.py - scene["prompt"] KeyError
- **증상:** `KeyError: 'prompt'` 썸네일 생성 단계에서 크래시
- **원인:** scene dict에 "prompt" 키가 없는 케이스 미처리
- **해결:** `.get("prompt", "")` 방어 처리
- **파일:** `thumbnail_builder.py`

---

### [E015] 3CROW B variant is_b=True 버그
- **증상:** 3CROW B 썸네일이 A와 동일하게 생성됨
- **원인:** `generate_crow_thumbnail()` 호출 시 `is_b=True` 파라미터 누락
- **해결:** B 생성 호출 시 `is_b=True` 명시
- **파일:** `thumbnail_builder.py`

---

### [E016] studio_crawler.py - Sheets 컬럼 범위 초과
- **증상:** `APIError: Requested writing within range that is out of bounds` (col 19 쓰기 시도)
- **원인:** `ensure_extra_columns()` 함수에서 추천%/홈%/검색% 컬럼(19~21) 추가 시 `sheet.resize()` 미호출 → 현재 18열 시트에 19열 쓰기 시도
- **해결:** `sheet.resize(rows=sheet.row_count, cols=21)` 추가
- **파일:** `studio_crawler.py` `ensure_extra_columns()`

---

### [E017] suno_bot.py - 폴더 저장 경로 혼재
- **증상:** `⚠️ 새 폴더 감지 실패 → fallback / ❌ wav/mp3 없음`
- **원인:** `get_latest_suno_folder()`가 `channel_dir` 스캔하는데 suno_bot은 `base_dir`(`C:\ThunderDrop\2026-04-07_1`)에 저장 → 경로 불일치
- **해결:** `suno_bot.py` `base_dir`을 채널별 분리 (`YACHA\audio`, `3Crow\audio`)
- **파일:** `suno_bot.py`

---

### [E018] dev 브랜치 → master 전환 시 코드 손실
- **증상:** `thumbnail_builder.py` 채널별 프롬프트 분리(mood word context), temperature=0.7 등 작업 내용 소실
- **원인:** dev 브랜치에서 작업 후 master로 전환 시 merge 없이 전환 → dev 변경사항 소실
- **해결:** dev 브랜치 폐기. **master 단일 브랜치** 운영 확정
- **파일:** 전체 레포
- **교훈:** 브랜치 운영하려면 merge 필수. 지금 규모에서는 master 단일이 더 안전

---

### [E019] AB 테스트 Shorts 등록 불필요 (정책 확정)
- **증상:** Shorts AB 테스트 등록 시도 → 실패 or 무의미
- **원인:** Shorts는 제목 고정(`{곡명} | {브랜드} #shorts`) + 썸네일 피드 자동 크롭 구조
- **해결:** Shorts AB 테스트 등록 스킵 확정. `master_pipeline.py`에서 Shorts AB 호출 제거
- **파일:** `master_pipeline.py`

---

### [E020] analytics_collector.py - 4.1일 영상 누락
- **증상:** 특정 날짜 영상이 Sheets에 기록되지 않음
- **원인:** `get_video_analytics()`가 `None` 반환 시 row 자체를 skip. YouTube Analytics API는 조회수 0이거나 데이터 없으면 row를 아예 반환 안 함
- **해결:** `analytics`가 None이어도 views=0, avg_dur=0 등 기본값으로 기록 (video_id/제목/날짜는 항상 저장)
- **파일:** `analytics_collector.py`

---

### [E021] 자동 git push 설정 오류
- **증상:** `run_git_push.bat`이 Task Scheduler에 등록된 것처럼 보임 → 테스트 전 코드가 자동 push될 위험
- **원인:** `ThunderDrop_GitPush` 태스크가 이전에 등록됨
- **해결:** `ThunderDrop_GitPush` 태스크 `Disabled` 확인 완료. `run_git_push.bat`은 수동 실행 전까지 동작 안 함
- **파일:** Task Scheduler
- **현재 상태:** ✅ Disabled (안전)

---

### [E022] 단곡 영상에 플리 제목 템플릿 적용됨
- **증상:** 4:36짜리 단곡에 "TOP 10 AGGRESSIVE PHONK MIX" 제목 붙음
- **원인:** 단곡 제목 생성 로직이 플리용 SEO 제목 템플릿 그대로 사용. 단곡 전용 템플릿 없음 + SINGLE_EXTRA_BLACKLIST 중복 정의(E008)로 금지어 필터 미작동
- **해결:** E008 해결로 동시 해결
- **파일:** `master_pipeline.py`, `seo_scraper.py`

---

### [E023] Shorts 영상 배경에 B/C 썸네일 미적용
- **증상:** Shorts A/B/C가 전부 A 썸네일 배경으로 생성됨
- **원인:** `build_shorts()` 함수에서 variant 인덱스 매핑 누락. `short_sub_idx`와 썸네일 variant(A/B/C)가 연결 안 됨
- **해결:** `build_shorts()`에서 `short_sub_idx` 0→A, 1→B, 2→C로 매핑 (video 경로 + thumbnail 둘 다)
- **파일:** `master_pipeline.py` `build_shorts()`

---

## 🔍 현재 미해결 / 모니터링 중

| ID | 증상 | 추정 원인 | 상태 |
|---|---|---|---|
| E024 | AB 테스트 0/10 실패 케이스 | Chrome 프로필 충돌 (Chrome 열린 채 실행) | Chrome 닫고 재실행 필요 |

---

## 📌 자주 실수하는 패턴

1. **hwaccel + CPU 필터 동시 사용** → zoompan, showcqt 등은 CPU 전용
2. **Sheets 빈 셀 방치** → 시간이 지나면 셀 한도 초과 유발
3. **브랜치 switch 전 merge 누락** → 작업 내용 소실
4. **Windows cp949 환경 이모지 출력** → 항상 `PYTHONUTF8=1` 또는 `sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')`
5. **동일 pickle 파일 두 채널 공유** → 채널별 독립 토큰 필수
