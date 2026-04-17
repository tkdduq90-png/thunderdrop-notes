# ThunderDrop 에러 로그
*이전 대화기록 전수 조사 기반 정리*

---

## 사용법
- 에러 발생 시 이 파일 먼저 검색
- 새 에러 해결 후 아래 형식으로 추가
- 형식: `에러명 | 원인 | 해결 | 파일`

---

## ✅ 해결된 에러 목록

---

### [E024] AB 테스트 0/10 실패 — 해결 (2026-04-14)
- **증상:** studio_ab_tester.py 실행 시 AB 테스트 등록 0/10 실패
- **원인:** Chrome 기존 실행 프로필과 Selenium 프로필 충돌
- **해결:** dedicated profile (`chrome_profile_yacha_ab`) + `_check_chrome_running()` 감지로 해결. 2026-04-14 AB 테스터 2/2 성공으로 작동 증명.
- **파일:** `scripts/studio_ab_tester.py`
- **재발 여부:** ❌ (근본 해결)

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

## 2026-04-09 Hook 제목 생성 관련

- [FIXED] YACHA 숏츠 Title B 생성 실패 → dry-run이 used_titles 소모하는 문제. dry_run=True 파라미터 추가로 해결
- [FIXED] Title C에 감성훅 혼입 ("when 3am hits different") → B/C 프롬프트 분기 처리로 해결
- [FIXED] 허위 통계 문구 생성 ("streamed 500k times") → DEFAULT_HOOK_BLACKLIST에 streamed/played/times 계열 추가
- [FIXED] 경쟁채널 곡명 훅 혼입 ("murder in my mind vibes") → DEFAULT_HOOK_BLACKLIST 강화
- [NOTE] used_titles.json 무한 누적 → 장기 이슈, 추후 30일 이상 항목 자동 정리 로직 필요

---

## 🔍 현재 미해결 / 모니터링 중

| ID | 증상 | 추정 원인 | 상태 |
|---|---|---|---|
| E_TODO | verify_upload.py 미제작 | 파이프라인이 자체 검증하는 구조 → 신뢰도 낮음 | 독립 스크립트 제작 필요 |

---

### [E025] Shorts temp 폴더 충돌
- 증상: short_clip_0_1.wav 등 2번째 이후 Shorts FFmpeg 오디오 merge 실패
- 원인: 6개 Shorts가 동일 temp_shorts/ 폴더 공유 → 중간 파일 충돌
- 해결: temp_dir을 temp_shorts/{day_idx}_{short_sub_idx}/로 변경
- 파일: master_pipeline.py build_shorts()

---

### [E026] 가사 20곡 동일 (mode 1 Suno 음원 단조화)
- 증상: tasks.txt의 20곡 전부 완전히 동일한 가사 블록을 공유, Suno 보컬 파트 반복
- 원인: step1_generate_prompts() 내부에서 `lyrics = yacha_generate_lyrics(...)` / `crow_get_lyrics(...)` 호출이 `if j == 0:` 블록 안으로 잘못 들여쓰기됨 → j=0에서만 가사 생성, j=1..19는 j=0의 lyrics 변수를 그대로 재사용
- 해결: lyrics 생성 호출을 `else:` 블록 내부(`suno_prompt` 생성 바로 아래)로 이동. `if j == 0:` 블록에는 `schedule[-1]["suno_style_prompt"] = suno_prompt` 한 줄만 남김. 매 루프 새로 뽑힌 `mood`가 `yacha_generate_lyrics(genre, purpose, mood, ...)` 호출에 반영되도록 유지
- 파일: prompt_builder.py step1_generate_prompts()
- 부수효과: Claude API 호출 횟수 1회 → 20×num_days회. 크레딧 소진 시 fallback 하드코딩 가사로 재차 동일화될 위험 있음 → 크레딧 모니터링 필요

---

### [E027] YACHA/3CROW 영상 설명 purpose 혼재 (Drive/Gym)
- 증상: YACHA(Gym Phonk) 플리 설명에 "car music", "night drive", "late-night drive" 등 3CROW용 키워드가 등장. 반대로 3CROW 설명에 "workout"/"gym" 키워드가 섞여 들어오는 역방향 사례도 발생 가능
- 원인: seo_scraper.generate_description()이 Claude에게 `Channel: {channel_key.upper()}` 문자열만 전달 → LLM이 채널 컨텍스트를 문자로만 인식. competitor analysis에서 잡힌 키워드(`keyword_clusters`)에 타 채널 장르 단어가 섞여 있으면 그대로 재사용함
- 해결: generate_description() 프롬프트에 `channel_constraint` 블록 주입. YACHA="GYM WORKOUT ONLY + drive/car/road/midnight/late-night drive 금지", 3CROW="NIGHT DRIVE ONLY + gym/workout/lifting/reps/training 금지". CTA도 채널별 고정 문자열("🔔 New gym phonk every day" / "🔔 New night drive music every day")로 하드코딩하여 LLM이 rephrase 못 하게 함
- 파일: seo_scraper.py generate_description()
- 관련: analyze_title_patterns() JSON 파싱 실패 시 `{"raw": text}` 반환 → 호출자 `"error" in patterns` 가드 우회 → 잘못된 패턴으로 제목 생성되던 부수 버그도 `{"error": "json parse failed", "raw": text}` 반환으로 함께 수정

---

### [E028] AB 테스트 "테스트 설정" 버튼 예약 비공개 영상에서 비활성 판단 실패
- 증상: 2026-04-10 mode 1 run에서 studio_ab_tester.py가 8건 0성공. 모든 영상에서 "30초 후에도 비활성 상태"로 포기
- 원인 3가지:
  1. 셀렉터가 `document.querySelectorAll('*')` 전체 DOM 순회 + `children.length === 0` leaf 조건 + `textContent === 'target'` 완전 일치 → 느리고 취약. 반환된 요소가 `<span>` (버튼 내부 라벨)이라 disabled 상태는 부모 `ytcp-button`에서 별도 조회 필요
  2. 대기 시간 30초 고정 (60회 × 0.5초) — 예약 비공개 영상의 YouTube 내부 처리가 완료되기 전에 타임아웃
  3. `except Exception: pass` 후 `is_ready = True`로 빠져서 **예외 발생 시 오히려 "활성"으로 오탐**. stale element 발생 시 치명적
- 해결:
  1. 셀렉터를 `document.querySelectorAll('ytcp-button')`로 변경, `textContent.includes(target)` 부분일치. 반환 객체가 바로 `ytcp-button`이라 disabled 체크가 부모 탐색 없이 바로 가능 → `parent_dis` 체크 로직 제거
  2. 대기 시간 30초 → 10분 (600회 × 1초). 60초마다 진행상황 로그 출력 ("여전히 비활성 Xs 경과")
  3. `except Exception as e: print(...); time.sleep(1); continue` — 예외 시 활성으로 간주하지 않고 계속 대기
- 파일: scripts/studio_ab_tester.py
- 비고: Shadow DOM에 버튼이 있는 경우는 여전히 미해결 (`querySelectorAll`이 shadow root 관통 안 함). 추후 필요 시 `driver.find_element` + shadow root traversal 추가 고려

---

### [E029] Shorts thumbnails().set() 즉시 업로드 타이밍 실패
- 증상: 테스트 모드(즉시 업로드) 시 Shorts 썸네일 미적용
- 원인: videos().insert() 완료 직후 바로 thumbnails().set() 호출 → YouTube 서버 처리 전
- 해결: thumbnails().set() 전 sleep(5) + 최대 3회 retry (간격 5초)
- 파일: master_pipeline.py _upload_one()

---

### [E030] studio_crawler.py Task Scheduler 실행 시 input() 블로킹
- 증상: 오전 10시 자동 실행 시 채널 선택창 뜨며 멈춤
- 원인: 세션 만료 시 wait_for_studio_login()에서 input() 대기 발생
- 해결: --no-interactive 플래그 추가. run_analytics.bat에 적용
- 파일: scripts/studio_crawler.py, run_analytics.bat

---

## 📌 자주 실수하는 패턴

1. **hwaccel + CPU 필터 동시 사용** → zoompan, showcqt 등은 CPU 전용
2. **Sheets 빈 셀 방치** → 시간이 지나면 셀 한도 초과 유발
3. **브랜치 switch 전 merge 누락** → 작업 내용 소실
4. **Windows cp949 환경 이모지 출력** → 항상 `PYTHONUTF8=1` 또는 `sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')`
5. **동일 pickle 파일 두 채널 공유** → 채널별 독립 토큰 필수
6. **FFmpeg 필터 체인 재사용 시 입력 크기 변경 주의** → crop_prefix 같은 전처리 필터가 16:9 전제로 설계됐는데 9:16 입력을 받으면 의도치 않은 대규모 crop 발생. 입력 해상도 변경 시 모든 필터 체인 재검토 필수.
7. **YouTube API 200 OK ≠ 실제 반영** → thumbnails.set() Shorts, videos.insert 중복 콘텐츠 모두 200 OK 반환하면서 서버에서 무음 거부. 반드시 후속 videos.list() 또는 snippet.thumbnails 조회로 실제 반영 검증 필수.
8. **day_idx 같은 로테이션 인덱스에 folder_idx/루프 변수 할당** → 고정 선택 증상 유발. 로테이션용 인덱스는 날짜 기반 또는 누적 카운터로 독립 계산. (E067)
9. **Popen 백그라운드 자식 프로세스에 input() 호출** → 무한 대기. 백그라운드 실행 경로는 stdin 없는 조건에서 동작하도록 --no-interactive 플래그 분리. (E069)
10. **캐시 키 생성 시 구분 파라미터 누락** → type=B/C 같은 변종이 동일 캐시 공유해서 한쪽이 소진되면 다른쪽도 빈 결과. 키 생성 함수 시그니처에 모든 구분자 포함 + 해시 구분자는 입력에 나올 수 없는 문자. (E068)
11. **템플릿 마커 삽입 후 치환 로직 누락** → 업로드된 결과물에 {{XXX}} 날것 노출. 마커는 반드시 치환 시점 명시 + 실패 fallback 지정. (E070)

---

## 2026-04-10 API 최적화 세션 예방적 수정

- [PREVENTION] seo_scraper.py CLAUDE_API_KEY None 시 RuntimeError 조기 raise → 모호한 AttributeError 방지
- [PREVENTION] generate_hashtags 해시태그 10개 미만 시 fallback 트리거 → Haiku 전환 후 품질 drift 방어
- [FIXED] seo_scraper.py import 순서 PEP 8 준수 (pickle stdlib → local import 순)
- [NOTE] claude-sonnet-4-20250514 → claude-sonnet-4-6 전체 최신화 (prompt_builder, seo_scraper)
- [NOTE] seo_scraper Anthropic 클라이언트 singleton 전환 (6곳 재생성 → 모듈 레벨 1개)

---

## 2026-04-11 AB 테스터 인터랙티브 UI 개발

- [FEAT] studio_ab_tester.py 플래그 없이 실행 시 인터랙티브 모드 (채널/영상/액션 선택 UI)
- [FEAT] video_type 컬럼 추가 → Shorts AB 테스트 자동 skip
- [FIXED] Chrome 실행 중 감지 → 로그인 유지 문제 근본 해결 (_check_chrome_running)
- [FEAT] sheets_logger/master_pipeline video_type 전달 연동
- [NOTE] _check_chrome_running subprocess timeout=5 + returncode 체크 추가 (코드리뷰 반영)

---

### [E031] search_competitor_descriptions 설명 수집 0개
- **증상:** 설명 수집 완료: 0개 — 설명/해시태그가 빈 컨텍스트로 생성됨
- **원인:** videoCategoryId="10" 고정 (night drive/gym 영상은 Music 외 카테고리), 구독자 필터로 전부 제외, 빈 결과 캐시 저장 → 이후 히트로 계속 0개
- **해결:** videoCategoryId 제거, 구독자 필터 전체 제거, if results: 가드로 빈 결과 캐시 차단
- **파일:** seo_scraper.py search_competitor_descriptions()

---

### [E032] generate_hashtags YACHA/3CROW 채널 혼재
- **증상:** YACHA 해시태그에 drive/night, 3CROW에 gym/workout 태그 혼입
- **원인:** channel_key를 프롬프트 문자열에만 삽입, 채널별 금지어 제약 없음
- **해결:** channel_constraint 분기 추가 — YACHA: gym/phonk only, 3CROW: night drive/techno only
- **파일:** seo_scraper.py generate_hashtags()

---

### [E033] Shorts thumbnail_B/C Leonardo 실패 시 미기록 (E023 재발)
- **증상:** Shorts B/C 썸네일 sched_entry 누락
- **원인:** if thumb_variants.get("B"): 조건으로 Leonardo 실패 시 필드 자체가 안 들어감
- **해결:** 조건 제거, raw fallback 항상 기록
- **파일:** master_pipeline.py build_shorts()
- **재발 여부:** E023 동일 원인, 근본 수정 완료

---

### [E034] crow_get_lyrics 폴백 20곡 동일 의심
- **증상:** API 실패 시 20곡 가사 동일 가능성
- **원인:** try/except 없이 에러 전파, 고정 2단어 폴백 반환
- **해결:** try/except 추가 + track_name 파라미터로 곡별 동적 폴백
- **파일:** prompt_builder.py crow_get_lyrics()

---

### [E035] FFmpeg drawtext 특수문자 파싱 오류
- **증상:** Claude API 반환값에 콜론/콤마 포함 시 FFmpeg 필터 파싱 깨짐
- **원인:** `replace("'", "\\'").replace("&", "and")` 만 적용, 콜론/콤마 미이스케이프
- **해결:** `.replace(":", "\\:").replace(",", "\\,")` 추가
- **파일:** thumbnail_builder.py yacha_add_text_overlay() L281

---

### [E035-2] suno_done.flag / tasks.txt / suno_bot.py 경로 불일치
- **증상:** step2_wait_for_suno()에서 suno_bot.py 못 찾음 + suno_done.flag 감지 실패
- **원인:** CHANNEL_CONFIG base_dir이 루트(`C:\ThunderDrop`)로 설정되어 suno_bot.py 경로가 `YACHA\audio\suno_bot.py`로 조합됨. suno_done.flag도 루트에서 감시하지만 suno_bot은 `YACHA\audio`에 생성
- **해결:** base_dir → 채널별 audio 폴더(`YACHA\audio`, `3Crow\audio`, `FocusArchitect\audio`), tasks.txt/upload_schedule.json → channel_dir 기준 통일, suno_bot.py 경로 루트 하드코딩
- **파일:** `master_pipeline.py`, `prompt_builder.py`, `suno_bot.py`

### [E036] 썸네일 시스템 구조적 버그 9건 (2026-04-13)

**증상**: 
- Shorts 썸네일 Studio 미반영
- 플리 썸네일 업로드 videoNotFound 404
- B/C 썸네일 텍스트 오버레이가 B-roll에 섞여 들어감

**원인**: 
전수조사 결과 `thumbnail_builder.py` / `yacha_thumbnail_variants.py` 
구조적 문제:
1. B 해상도 요청(1280×720) vs 실제(1024×1024) 불일치 - Leonardo v2 기본값
2. generate_thumbnail_variants() B 섹션 is_shorts 미전달
3. C 섹션 variant_num=0 하드코딩 (E023 재발)
4. C 섹션 video_type 미전달 → playlist 경로 저장
5. generate_yacha_thumbnail() thumb_prompt="" 반환
6. yacha_add_text_overlay() "PLAYLIST" 하드코딩
7. fixed_colors dead parameter
8. YACHA_B_NEGATIVE_PROMPT 정의만, 미사용
9. Shorts _log_title_variants 미호출

**해결**: 
패치 포기. `thumbnail_service.py` 신규 단일 모듈로 전면 재설계.
Raw/Final 분리, Leonardo v2 단일화, A/B/C 통합 진입점.
Phase 1~5 단계별 마이그레이션.

**파일**: thumbnail_service.py (신규), master_pipeline.py (호출부), 
beat_video.py (B-roll 경로)

---

### [E043] shutil.copy2 src==dst PermissionError (2026-04-13)

**증상**: `PermissionError: [WinError 32] 다른 프로세스가 파일을 사용 중`  
`scripts/update_obsidian.py`, `save_and_push()` 내 `shutil.copy2()` 라인

**원인**: `LOCAL_PATH`를 `notes-repo/docs/`에 직접 지정하도록 리팩터한 후,  
`notes_file` 변수도 동일 경로를 가리키게 됨.  
파일을 `open(LOCAL_PATH, "w")` 로 쓴 직후 `shutil.copy2(LOCAL_PATH, notes_file)` 호출 → src == dst → WinError 32

**해결**: `shutil.copy2` 블록 전체 제거. `LOCAL_PATH`가 이미 `notes-repo/docs/`를 직접 가리키므로 복사 불필요

**파일**: `scripts/update_obsidian.py`

---

### [E044] subprocess UnicodeDecodeError (cp949) (2026-04-13)

**증상**: `UnicodeDecodeError: 'cp949' codec can't decode byte 0xed`  
`update_obsidian.py` 실행 시 git 출력 처리 중 발생

**원인**: Windows 환경에서 `subprocess.run(text=True)` 시 시스템 기본 인코딩(cp949)으로 stdout/stderr 디코딩.  
git 출력에 이모지/한글 포함 시 cp949로 디코딩 불가

**해결**: `text=True` 있는 `subprocess.run` 전부에 `encoding="utf-8", errors="replace"` 추가  
(`git add`, `git commit`, `git push`, `git diff --cached --quiet` 4곳)

**파일**: `scripts/update_obsidian.py`

---

### [E045] git commit 오탐 "nothing to commit" (2026-04-13)

**증상**: `⚠️ git commit 실패:` 출력(빈 메시지). 실제 에러가 아닌데 실패 메시지 출력

**원인**: Claude API가 노트 내용을 변경 없이 그대로 반환 → `git add` 후 staged diff 없음 →  
`git commit` exit 1 → 스크립트가 실패로 판정

**해결**: `git commit` 전에 `git diff --cached --quiet` 체크.  
exit 0 (변경 없음) → `"변경사항 없음 - commit/push 생략"` 출력 후 graceful return

**파일**: `scripts/update_obsidian.py`

---

# 신규 포맷 (E037~)

독자: 다음 세션의 Claude. 작업 시작 전 작업 영역 태그로 grep.
포맷: 근본원인 1줄 + 체크리스트 + 재발 목록.

---

## [E037] 이식 시 상수·페이로드·해상도 필드 누락
태그: #이식 #leonardo #payload #해상도
재발: 3회 / 2주
커밋: 78d801b, ccac432

근본원인: "이식"을 함수 로직 복사로만 해석. 상수/페이로드 필드/매직넘버 검증 누락. Mock 단위 테스트가 이 차이를 못 잡음.

체크리스트 (이식 작업 전):
- 원본/신규 파일 상수 전수 diff (`grep -E '^[A-Z_]+ = '`)
- API 페이로드 딕셔너리 필드 단위 diff (누락/추가/변경 전부)
- 매직넘버(해상도/BPM/색상/엔드포인트 URL/모델명) grep 대조
- 라이브 테스트 먼저 설계 (E038 참조)
- 공식 API 문서 확인, 추측 금지

재발:
- v1 → v2 endpoint 조용히 변경 (yacha_thumbnail_variants → thumbnail_service)
- 해상도 1344×768 → 1280×720 (Leonardo v2 VALIDATION_ERROR)
- 폰트 경로 콜론 미이스케이프 `C:/...` (FFmpeg drawtext PARSE ERROR, 6/9 silent)

---

## [E038] Mock 단위 테스트 과신, 라이브 검증 누락
태그: #테스트 #mock #라이브검증
재발: 1회
커밋: 78d801b

근본원인: Mock은 "함수가 호출됐는가"만 검증. "그 호출이 외부 시스템에서 유효한가"는 검증 불가. 고정 dict 반환, 에러 시뮬레이션 없음, side effect 없음.

체크리스트 (신규 모듈 구현 시):
- 라이브 테스트 스크립트를 mock 테스트와 동시에 설계
- "완료" 기준 = mock PASS 아님. 라이브 PASS여야 완료
- 라이브는 최소 1회 실제 API 호출 포함 (과금 불가피)
- 라이브 성공 판정: HTTP 200 + 반환 구조 검증 + 파일 존재/크기>0 + WARN 0건
- 과금 절약: 최소 케이스로 설계 (video_type 1개 × variant 3개 등)

재발:
- Phase 1 mock 13/13 PASS → 라이브 호출 시 AttributeError + 6/9 silent 실패

---

## [E039] 부분 성공을 성공으로 판정
태그: #테스트판정 #WARN #거짓성공
재발: 1회
커밋: ccac432

근본원인: 성공 기준이 느슨. "파일 존재" = 성공으로 판정하고 내용 검증 생략. WARN을 "성공이지만 경고"로 해석. Fallback이 silent하게 동작해서 사용자가 실패 인지 못 함.

체크리스트 (라이브 테스트 판정 시):
- WARN 로그 0건 엄격 강제
- 파일 검증 = 존재 + 크기 > 0 + 내용 (텍스트 오버레이면 final 크기 > raw 크기)
- Fallback 경로는 stderr/traceback 로깅 필수 (silent 금지)
- 보고 시 "N/N PASS"와 별개로 `grep -c WARN` 결과 명시
- WARN 1건이라도 = 실패. 부분 성공 금지.

재발:
- Phase 1 라이브: "9/9 PASS" 보고, 실제 [WARN] text overlay 6/9 silent

---

## [E040] 리팩토링 시 Sheets 스키마 변경 → 하위 호환 위반
태그: #하위호환 #sheets #스키마 #어댑터
재발: 0회 (잠재, 사전 차단)
커밋: be2979d

근본원인: 신규 모듈이 "더 풍부한 정보"를 반환한다고 어댑터가 그대로 전달하면, Sheets 컬럼에 기존과 다른 형태의 값이 쌓임. 기존 값들로 이미 암묵적 계약 형성돼 있음. 분석 쿼리 + E010(셀 한도) 리스크.

체크리스트 (어댑터/리팩토링 작업 시):
- 구 API 반환값 각 필드의 실제 값 샘플 확인 (Sheets 또는 로그)
- 어댑터 원칙: "Wrap, don't enhance" - 변환만, 개선 금지
- 정보 확장 필요 시: 기존 필드 보존 + 새 필드 추가. 기존 필드 오염 금지
- Sheets 컬럼 형태 변경 전 E010 영향 평가
- "풍부함"이 목표면 별도 Phase로 분리

재발:
- Phase 2.0 어댑터: thumb_prompt 필드에 Leonardo 전문 전달 vs 빈 문자열 유지 논의 → 빈 문자열 확정. 프롬프트 로깅은 Phase 5로 분리.

---

## [E041] else 블록 재구성 시 빈 줄 누락
태그: #코드스타일 #refactor #gate3
재발: 1회 (초발)
커밋: 70a0465

근본원인: master_pipeline.py 에서 if/else 플래그 분기를 새로 도입하면서 기존 else 블록 안으로 코드를 들여쓰기할 때, 블록 마지막 라인과 다음 블록 사이의 빈 줄이 사라짐. 기능 영향 없으나 code-review Important 이슈.

증상:
- Phase 2.1 build_shorts() else 블록 selected_thumb = ... 다음 bg_image = ( 직전 빈 줄 누락
- 원본 HEAD 는 해당 위치에 빈 줄 존재
- 라이브 테스트 전부 통과, pytest 전부 통과
- code-reviewer 스킬만 발견

체크리스트 (if/else 블록 재구성 시):
- diff 검토 시 라인 수 변화만 보지 말고 빈 줄 대조
- if/else 분기 재구성 후 기존 스타일 (빈 줄 포함) 유지
- code-review 스킬을 커밋 전 필수 게이트로 유지

재발:
- 2026-04-13 Phase 2.1 build_shorts() else 블록 (초발)

---

## [E042] 레거시 삭제 시 scripts/ 하위 테스트 파일 스캔 누락
태그: #refactor #cleanup #dead-reference #grep범위
재발: 1회 (초발)
커밋: e2139bd

근본원인: 레거시 함수 삭제 작업 시 루트 디렉터리 파일만 grep 검사, scripts/ 하위 폴더 누락. 삭제된 함수를 호출하는 파일이 남아 실행 시 AttributeError 발생.

증상:
- Phase 4 레거시 제거 후 Gate 1 diff 검증 통과
- code-review 에서 scripts/test_thumb_gen.py 발견: thumbnail_builder.yacha_add_text_overlay(...) 호출 잔존
- 실행 시 AttributeError: module has no attribute

체크리스트 (레거시 제거 작업 시):
- grep -rn 으로 프로젝트 전체 스캔 (scripts/, tests/, utils/ 등 하위 폴더 포함)
- 함수명 + import 문 둘 다 검색 (from X import Y / X.Y())
- --include="*.py" 사용 시 폴더 깊이 제한 없음 확인
- code-review 를 2차 안전망으로 유지

재발:
- 2026-04-13 Phase 4 Gate 3 Critical (초발): scripts/test_thumb_gen.py 잔존

---

## [E046] YouTube 중복 콘텐츠 무음 삭제 (2026-04-13, 04-14 확대)
태그: #upload #youtube #duplicate #ghost #playlist
재발: 1회 (초발)
커밋: Phase5 진단 (수정 미구현)

근본원인: 동일 schedule.json을 여러 번 재실행하면 동일 오디오 파일을 여러 차례 업로드. YouTube 중복 콘텐츠 감지 시스템이 videos.insert() 응답에 200 OK + video_id를 반환한 뒤 내부적으로 영상을 무음 삭제. 반환된 video_id는 실제로 존재하지 않음 → thumbnails.set() 시 404 videoNotFound.

증상:
- videos.insert() 성공 → 로그에 video_id 기록 → thumbnails.set() 3회 retry 전부 404
- YouTube Studio에서 해당 video_id 조회 불가
- YACHA 4/13 20건 업로드 전부 ghost. 4/10~ 모든 업로드 ghost
- uploads playlist 36건 중 4/10 이후 영상 0건
- 4/13 동일 schedule.json 4~5회 재실행이 트리거 추정

확인된 ghost video_id (2026-04-13 YACHA 플리):
- STVACAQE2Sg (4번째 업로드, 12:23:45)
- Fs0Xih-YIQg (5번째 업로드, 21:11:39)
- 실제 성공 video_id: sSo9TXRffLw (3번째, 11:21:00)

간접 증거:
- 3CROW 4/14 5건 (새 오디오 + 새 schedule) → 5/5 ghost 없음, 정상
- 대조군 정상 작동 확인

임시 조치:
- YACHA 업로드 전 새 Suno 음원 확인 후에만 진행
- 동일 schedule 반복 실행 금지

Phase 6 수정 예정:
- 중복 업로드 방지 (upload_history.csv 기반 skip)
- 업로드 후 videos().list() 즉시 존재 확인
- ghost 감지 시 경고 + Sheets 미기록

체크리스트 (업로드 전):
- upload_history.csv에서 동일 제목/파일의 최근 성공 업로드 여부 확인
- 중복 업로드 방지 로직: 당일 동일 sched_type+title 업로드 기록 있으면 skip
- thumbnails.set() 전 videos().list(part="id", id=video_id) 존재 여부 확인
- pipeline 재실행 시 --dry-run 먼저 확인

재발:
- 2026-04-13 YACHA 플리 5회 중복 (초발): schedule.json 재실행 5회

관련: phase5_diagnosis.md §9.5

---

## [E047] Shorts thumbnails.set() 즉시 실행 시 403 forbidden (2026-04-13)
태그: #upload #youtube #shorts #thumbnail #403
재발: (E029 재확인, 원인 규명)
커밋: Phase5 진단 (수정 미구현)

근본원인: YouTube Shorts 업로드 직후 thumbnails.set() 호출 시 403 forbidden. 업로드 성공 직후 YouTube 내부 처리가 완료되기 전 임시 제한. 시간 경과(수십분~수시간) 후 자동 해제.

증상:
- videos.insert() 성공 → thumbnails.set() 즉시 호출 → 403 forbidden
- 동일 video_id에 수 시간 후 thumbnails.set() 재시도 → SUCCESS
- master_pipeline.py L1187: Shorts는 sleep=0초 (플리/단곡은 5초) — 부족

확인 (2026-04-13):
- CS00SUzgJtE: 업로드 시 403 → 수 시간 후 수동 테스트 SUCCESS
- 현재 retry: 5초 간격 3회 → 부족, 최소 30초+ 필요 (또는 별도 재시도 메커니즘)

체크리스트 (Shorts 업로드 후):
- thumbnails.set() 전 sleep 최소 30초 (현재 0초 → 부족)
- 실패 시 재시도 간격: 30초 이상으로 확대
- 또는 별도 "delayed thumbnail setter" 큐 구현 고려

재발:
- E029 (2026-04-10): Shorts 즉시 업로드 타이밍 실패 — 동일 근본원인 재확인

---

## [E050] 가설 과신 + 라이브 검증 누락 (2026-04-13)
태그: #디버깅방법론 #가설과신 #라이브검증 #E038재발
재발: 1회 (초발, 메타 에러)
커밋: Phase5 세션

근본원인: 디버깅 세션에서 증상 하나에 대해 가설을 추정으로 쌓고, 각 가설을 기각할 때마다 새 가설로 교체. 라이브 검증 전에 결론 내리고 다음 가설로 넘어감. 결과: 하루에 가설 6~7개 생성·기각·재생성, 원인 미확정 상태로 세션 종료.

증상:
- 2026-04-13 Phase 5 세션: 8시간 동안 가설 6번 뒤집힘
- 각 가설마다 "이번엔 확실" 판단
- 원본 증거(ghost 의심 영상) 사용자가 중간에 삭제 → 검증 불가 상태로 진행
- Phase 6 코드 수정 직전까지 갔다가 최종 원인 미확정으로 중단

체크리스트 (미스터리 디버깅 세션 시):
- 세션 시작 시 "시간 제한" 명시 (예: 30분 → 연장 시 재평가)
- 가설 교체 시 이전 가설 "왜 기각했는지" 명문화
- 원본 증거 보존 우선 (사용자가 지우지 못하게 "증거 보존 요청" 먼저)
- 라이브 검증 없이 "수정 착수" 금지 — E038 체크리스트 강제
- 하루 동안 가설 3개 이상 뒤집히면 "세션 중단, 기록, 내일 재개" 강제

재발:
- 2026-04-13 Phase 5 미스터리 디버깅 (초발)

관련: E038 (Mock 단위 테스트 과신), E039 (부분 성공 판정)

---

## [E048] Shorts 썸네일 API 미반영 — 해결 (2026-04-14)
태그: #upload #youtube #shorts #thumbnail #api
재발: ❌ (API 포기, 전략 변경으로 근본 해결)
커밋: 4abf55f, 4718a1f, f49bf1d

근본원인: thumbnails().set() Shorts 호출 시 200 OK 반환하지만 Studio 피드/목록/상세에서 첫 프레임만 표시됨. 커스텀 썸네일 반영 안 됨. Google Issue Tracker #381127084 케이스.

증상:
- 16:9 (1280×720) 썸네일 → 반영 안 됨
- 9:16 (768×1344) 썸네일 API 업로드 → 여전히 반영 안 됨
- Studio 상세 페이지에 "YouTube 모바일 앱에서 썸네일을 변경할 수 있습니다" 안내
- API 경로 자체가 Shorts에 대해 사실상 무력화

해결 전략: API 포기, 영상 첫 프레임을 썸네일 역할로 활용
1. Leonardo 768×1344 (9:16) 생성 — 썸네일 = 영상 배경 동일 이미지
2. beat_video.py crop_prefix 제거 — 원본 전체가 영상 첫 프레임에 표시 (24.6% → 96.2%)
3. master_pipeline.py _upload_one() Shorts 썸네일 API 호출 스킵
4. 결과: 영상 첫 프레임 = Leonardo 이미지 = 피드/목록 썸네일

검증: 라이브 테스트 YMiKD1p0kl0 Studio 육안 확인 통과.

파일:
- thumbnail_builder.py _leonardo_generate_one, generate_crow_thumbnail, generate_thumbnail_variants
- beat_video.py _build_dynamic_zoompan, _static_zoompan crop_prefix
- master_pipeline.py _upload_one

참고 링크:
- https://issuetracker.google.com/issues/381127084
- https://issuetracker.google.com/issues/391129953

---

### [E051] Leonardo API 잔액 부족 시 모든 썸네일 실패 → Shorts 0개 생성 (2026-04-15)
- **증상**: `⚠️ Leonardo v2 실패: 'list' object has no attribute 'get'`
  - A/B/C 모든 variant 동일 에러
  - ThumbnailSet success=False
  - Shorts 영상: `expected str, bytes or os.PathLike object, not NoneType` (None 폴백 부재)
- **원인**: Leonardo API 잔액 0일 때 에러 응답이 list 형태로 반환되는데 코드는 dict 가정하고 `.get()` 호출
- **일시 해결**: API 충전
- **항구 해결 (별건)**: Leonardo 응답 list/dict 양쪽 처리, Shorts None 폴백 추가
- **파일**: `thumbnail_service.py`, `master_pipeline.py` build_shorts

---

### [E052] _log_to_sheets 키 불일치 (tracks vs tracks_meta) (2026-04-15)
- **증상**: build_video result dict에 `tracks_meta` 키만 존재하는데 build_single_video, build_shorts는 `playlist_sched.get("tracks", [])` 로 읽음 → 빈 리스트 → 매칭 실패 → 폴백 "gym phonk, workout" 출력
- **해결**: build_video result dict에 `"tracks": _tracks_meta` 추가 (양방향 호환)
- **커밋**: c992257
- **파일**: `master_pipeline.py` build_video, build_single_video, build_shorts

---

### [E053] generate_dashboard row 인덱스 22컬럼 미반영 (2026-04-15)
- **증상**: dashboard.py L96에서 `row[12]`를 thumb_text로 읽음. 옵션 3 작업으로 `row[12]` = thumb_prompt_B로 변경됨. dashboard가 thumb_text 자리에 prompt_B 값 표시
- **해결**: `row[12]` → `row[14]` (thumb_text 위치 조정)
- **커밋**: acf05dd 포함
- **파일**: `scripts/generate_dashboard.py`

---

### [E054] Leonardo v2 API list 응답 → 'list' has no attribute 'get' (2026-04-15)
- **발견**: 2026-04-14 실테스트, Shorts B variant (file_idx=202). 같은 실행에서 file_idx=200, 201은 성공.
- **증상**: `⚠️ Leonardo v2 실패: 'list' object has no attribute 'get'`
- **원인**: Leonardo v2 API가 quota/rate limit 시 HTTP 200 + list body 반환 (`[{"error": "..."}]`). 코드는 dict 가정하고 `.get()` 호출 → AttributeError.
- **영향**: 제한적 (A/C variant fallback 작동, 영상 업로드 무영향)
- **해결**: e0e8e5a — `_upload_init_image` L186, `_leonardo_i2i_v2` 생성 L253, 폴링 L269에 `isinstance(body, list)` 가드 3곳 추가. 생성/업로드는 RuntimeError, 폴링은 continue (range(30) 보호).
- **파일**: `thumbnail_service.py`
- **패턴**: E051과 동일 근본원인 (API 200 OK + 비정상 body 형태)

### [E055] AB 테스터 5쌍 동일 제목 영상 등록 실패 (2026-04-16)
- **발견**: 2026-04-16 세션 시작 시, studio_ab_tester.py에서 1eXOsJUB6Qo 미등록 발견
- **증상**: Sheets Analytics_Manual에 동일 제목 영상 5쌍(10건) 존재. 1eXOsJUB6Qo가 ab_registered=TRUE로 오마킹되어 목록 제외
- **원인**: 4/13 mode 2/3 반복 실행으로 동일 schedule이 재처리됨. 5쌍 중복: HyS5/CB3S/sSo9/STVA/Fs0X + iXpz/JkcG/6ckA/VSLC/XNpI
- **해결**: Sheets 10건 행 삭제 + YouTube 비공개 영상 삭제 + 1eXOsJUB6Qo ab_registered 비우기 + AB 테스터 재실행 → 정상 등록
- **파일**: `scripts/studio_ab_tester.py`, Sheets Analytics_Manual
- **재발 방지**: Phase A (mode 2/3 confirm) 도입으로 의도하지 않은 schedule 재실행 차단 (f07aa2f)

### [E056] 2xD9gTGN4GU video_type 오분류 의심 — 정상 확정 (2026-04-16)
- **발견**: 2026-04-16 디버깅 중 2xD9gTGN4GU가 단곡인데 video_type=playlist로 표기
- **증상**: video_type 필드값 불일치 의심
- **원인**: 조사 결과 실제로 2곡 합본 playlist. video_type=playlist는 곡 수 무관, "합본"을 의미
- **해결**: 코드 변경 없음 (의도된 동작 확정). 진단 단계 마무리
- **파일**: `prompt_builder.py` (build_video logic)
- **패턴**: 단곡=1곡 영상, playlist=합본(2곡 이상)

### [E057] mode 1 schedule 덮어쓰기로 복구 불가 (2026-04-16)
- **발견**: Phase A/B 디버깅 시 1차 실행 schedule이 2차 실행으로 덮어써져 사후 분석 불가
- **증상**: Leonardo 실패 시점, 제목 생성 결과 등 이전 schedule 데이터 추적 불가
- **원인**: master_pipeline.py step3 json.dump가 기존 schedule_file을 무조건 덮어씀. 백업 메커니즘 부재
- **해결**: Phase E-8 — schedule 자동 백업 시스템 도입 (ea8fe01). step3 json.dump 직전 schedule_backups/ 폴더로 shutil.copy2. 파일명 upload_schedule_{channel_key}_{YYYYMMDD_HHMMSS}.bak.json. 30일 이상 mtime 기준 자동 삭제. 백업 실패 시 logger.warning + 파이프라인 계속
- **파일**: `master_pipeline.py` L701~L722 (step3_create_videos_with_groups)

### [E058] print 분산으로 파일 로그 부재 (2026-04-16)
- **발견**: 2026-04-16 Phase E 계획 시 전수 조사
- **증상**: 8개 모듈에서 print() 사용으로 운영 정보가 콘솔에만 출력. 파이프라인 실행 후 stdout 캡처 안 했으면 정보 손실
- **원인**: 각 모듈에서 print 사용 + master_pipeline에 logging.xxx 혼재. file logging 핸들러 부재. 모듈명 식별 불가
- **해결**: Phase E-1~E-7 — 8개 핵심 모듈 named logger 전환. `logger = logging.getLogger(__name__)` + print → logger.info/warning/error 의도별 분류. CLI 출력(메뉴/진행률/dry-run)은 print 유지. 로그 파일 logs/{channel}_{YYYY-MM-DD}.log
- **파일**: thumbnail_service, seo_scraper, prompt_builder, sheets_logger, beat_video, suno_bot, master_pipeline
- **커밋**: f403249, 61351dd, 5d66de0, e886a58, 08dfbe3, ca33bae, 7793e93

### [E059] suno_bot subprocess logger 합류 불가 (2026-04-16)
- **발견**: Phase E-6 작업 중 suno_bot.py 로그가 master_pipeline 파일에 미기록 발견
- **증상**: suno_bot.py 로그 출력이 stdout으로만 가고 파일에 기록 안 됨
- **원인**: master_pipeline이 subprocess.Popen으로 suno_bot.py를 별도 프로세스 실행. 별도 프로세스는 부모 logging 핸들러 상속 불가
- **해결**: Phase E-6 — suno_bot.py __main__ 블록 내 자체 basicConfig(force=True) 추가. FileHandler(logs/{날짜}_suno.log) + StreamHandler
- **파일**: `suno_bot.py` L276~L291
- **커밋**: ca33bae
- **패턴**: subprocess로 실행되는 모듈은 자체 basicConfig 필요 (예: analytics_collector 동일 패턴 적용 검토)

### [E060] except 블록 traceback 부재 (2026-04-16)
- **발견**: Phase E 통합 코드리뷰 (AST 분석) 시 9곳 누락 검출
- **증상**: except 블록 내 logger.error/warning에 exc_info=True 누락. 실패 시 메시지만 기록되고 stack trace 부재 → 근본 원인 분석 불가
- **원인**: Phase E 변환 시 일부 except에 exc_info=True 미적용. HIGH 2건 (L1238 채널 검증, L1535 체크포인트) + MEDIUM 7건 (L504/L633/L722/L777/L1145/L1302 master + seo_scraper L271)
- **해결**: Phase E-9 (6c09c3f) — 9곳 전부 exc_info=True 추가. AST 전수 검증으로 except 내 100% 커버리지 확인. exc_info 누적: master 11→20, seo 8→9
- **파일**: `master_pipeline.py`, `seo_scraper.py`

### [E061] dry-run 단곡C/숏츠B,C "(생성 실패)" — 외부 요인 확정 (2026-04-16)
- **발견**: Phase E 완료 후 dry-run 3회 검증 실행 시 비결정론적 실패
- **증상**: _dry_run_titles 실행 시 단곡 C, 숏츠 B/C에서 "(생성 실패)" 메시지. 3차에서 단곡 B도 실패
- **원인**: Claude API rate limit (Anthropic side throttling). 코드 회귀 아님 (Phase E 변경 무관). 운영 mode 1에서는 정상 동작 (요청 간 시간 여유 있음)
- **해결**: 정상 동작 확정 (외부 요인). 코드 수정 불필요
- **파일**: `prompt_builder.py` (fallback 로직), `master_pipeline.py` (_dry_run_titles)
- **백로그**: dry-run 단독 호출 시 basicConfig 미설정으로 logger 출력 미흐름 → dry-run 진입 시 가벼운 basicConfig 추가 검토

### [E062] studio_ab_tester regenerate YACHA silent 실패 (2026-04-16)
태그: #ab-tester #regenerate #yacha #레거시경로
- **발견**: 2026-04-16 세션, YACHA 2건(rYUKCfjkWkk, XqccuydKjoA) regenerate 모드
- **증상**: "❌ Leonardo B 실패 / ❌ Leonardo C 실패" 만 출력. Leonardo 크레딧 충분. 원인 불명
- **원인**: studio_ab_tester.py regenerate 모드가 레거시 `thumbnail_builder.generate_thumbnail_variants()` 만 호출. 이 함수의 VARIANT_B_SCENES / VARIANT_C_PROMPTS 에 YACHA 키 미등록 → B/C 항상 None 반환 → Leonardo 호출 자체 발생 안 함. Phase 1~4 마이그레이션에서 studio_ab_tester 경로 누락
- **해결**: d836456 — channel_key=="yacha" 분기 추가, `build_and_adapt_yacha()` 호출. 3CROW/focusarchitect 기존 경로 유지
- **파일**: `scripts/studio_ab_tester.py`
- **체크리스트**: Phase 마이그레이션 시 master_pipeline 외 호출 진입점 grep 전수 (studio_ab_tester, dashboard, 일회성 스크립트 포함). YACHA 전용 경로가 build_and_adapt_yacha() 통하는지 검증

### [E063] 영상 bg_image 텍스트 박힌 final 사용 — 5개 경로 (2026-04-16)
태그: #영상배경 #썸네일혼용 #raw누출 #silent
- **발견**: 2026-04-16 영상 배경 이미지 재감사 (docs/video_text_audit_2026-04-16.md)
- **증상**: YACHA/3CROW 플리/단곡/Shorts 5개 경로 영상 배경에 PLAYLIST/HOOK/Phonk 타이포 등 텍스트 박혀있음. "영상 내부에 drawtext/drawbox 없음" 1차 감사로는 못 잡힘 — 영상 체인이 아니라 배경 이미지 소스가 원인
- **원인**: master_pipeline.py가 영상 bg_image로 final(텍스트 박힌 버전) 사용. ThumbnailResult.raw_path 필드 이미 존재했으나 _variants_dict()에서 미노출. Shorts에서는 직접 raw 경로 추측했으나 3중 불일치(날짜 형식/서브폴더/파일명 패턴)로 raw 미존재 → fallback으로 final 사용
- **해결**: 4c27d8b — (1) thumbnail_adapter._variants_dict()에 A_raw/B_raw/C_raw 키 추가 (2) generate_crow_thumbnail() 반환 4-tuple 확장 (3) master_pipeline playlist/single/shorts 3블록 bg_image raw→final→banner fallback 체인 (4) raw 미존재 시 logger.warning (5) Shorts raw_bg 경로 추측 로직 폐기
- **파일**: `thumbnail_adapter.py`, `thumbnail_builder.py`, `master_pipeline.py`
- **체크리스트**: "영상 배경 텍스트 없음" 검증은 drawtext grep으로 부족, bg_image 소스 역추적 필요. 파일 경로를 호출부에서 추측하는 방식 금지 (단일 소스 원칙 위반)

### [E064] YACHA Shorts A 썸네일 16:9 저장 버그 (2026-04-16)
태그: #썸네일 #shorts #해상도 #레거시경로
- **발견**: 2026-04-16 Shorts A 최종본(A_XXX.jpg) 1344×768(16:9). raw(A_XXX_raw.jpg)는 정상이었으나 확인 결과 raw도 16:9 (Leonardo I2I 기본값)
- **증상**: Shorts A 썸네일이 16:9로 저장되어 YouTube 피드에서 좁게 표시. B/C는 thumbnail_service.py RESOLUTIONS[video_type] 사용으로 정상 9:16
- **원인**: yacha_thumbnail_variants.py가 A variant 전용 레거시 경로. WIDTH=1344, HEIGHT=768 상수 하드코딩. _leonardo_i2i() 기본값 width=1344, height=768. generate_variant()가 video_type 파라미터를 받지만 내부에서 해상도 결정에 미사용
- **해결**: b64ec60 — RESOLUTIONS dict 추가 (playlist/single=1344×768, shorts=768×1344). composite_thumbnail에 video_type 파라미터 추가. _leonardo_i2i 호출 시 leo_w/leo_h 명시 전달. yacha_c resize도 video_type 반영
- **파일**: `yacha_thumbnail_variants.py`
- **체크리스트**: 하드코딩된 해상도 상수는 video_type 분기가 필요한 모듈의 사일런트 버그 원인. B/C와 A가 다른 모듈을 타는 경우 해상도 처리 방식 대조 필수

### [E065] 오디오 저장 경로 이원화 문제 (2026-04-16)
태그: #디렉토리구조 #오디오경로 #pick_best_versions
- **발견**: 2026-04-16 폴더 구조 조사 — Suno 다운로드는 YACHA/audio/{날짜}_1/에, 선별 후 wav는 YACHA/{날짜}/에, 미선별은 YACHA/{날짜}_미선택/에 분산 저장. failed_tracks.txt만 audio/ 하위에 남음
- **증상**: audio/ 폴더 안에 failed_tracks.txt만 있고 wav가 없어 보이는 착시. 실제 wav는 channel_dir 최상위 날짜 폴더에 산재
- **원인**: master_pipeline.py pick_best_versions()가 sel_dir/unsel_dir을 channel_dir(=YACHA) 기준으로 생성. base_dir(=YACHA/audio)와 불일치
- **해결**: 0f9d769 — sel_dir/unsel_dir 경로에 "audio" 세그먼트 추가 (L244, L261). 기존 16폴더 419파일 14.7GB shutil.move로 audio/ 하위 통합 이동 (파일 무결성 100%)
- **파일**: `master_pipeline.py` (2줄 변경)
- **체크리스트**: ch_dir/base_dir처럼 의미 구분된 경로 상수가 실제 디렉토리 구조와 일치하는지 정기 확인. 경로 분기점은 pick_best_versions 같은 중간 함수에 숨어있을 수 있음

### [E066] Shorts raw_bg 하드코딩 A_raw (2026-04-17)
태그: #shorts #raw_bg #variant
재발: 0회
커밋: 9b64f00

근본원인: thumb_variants.get("A_raw", "") 하드코딩으로 variant_key(A/B/C) 무시.
Short B/C 비디오 배경이 A variant의 raw 이미지를 사용. 동일 init 2중 사용 증상.

체크리스트:
- variant_key 파라미터가 해당 스코프에서 정의됐는지
- thumb_variants dict에 {variant}_raw 키가 A/B/C 모두 존재하는지
- raw_bg 선택 시 동일 패턴이 플리/단곡에도 있는지 (구조 차이 확인)

해결: `thumb_variants.get(f"{variant_key}_raw", "")` 로 variant별 선택

### [E067] day_idx 로테이션 고장 (고정 선택 증상) (2026-04-17)
태그: #thumbnail #rotation #day_idx
재발: 0회
커밋: 11af853

근본원인: master_pipeline.py day_idx = folder_idx 할당.
1폴더(days=1) 실행 시 folder_idx=0 고정 → COMBO[0] 항상 선택 → rear 포즈만 생성.
로테이션 로직(COMBO[day_idx % 9])은 정상이었으나 입력 day_idx가 날짜 비연동.

체크리스트:
- day_idx 값이 run마다 달라지는가 (dry-run 출력)
- % 로테이션 결과 값 분포 확인
- day_idx가 날짜 식별자 vs 순수 로테이션 인덱스 중 어느 용도인지 grep으로 확인
- 다른 호출부에서 day_idx를 file naming 등에 쓰고 있으면 날짜 기반으로 바꿀 때 충돌 가능

해결: `_day_idx = (datetime.today() - datetime(2026, 1, 1)).days`
날짜 기반 누적 인덱스로 매일 다른 포즈 자동 선택.

### [E068] Hook 캐시 title_type 무시 (2026-04-17)
태그: #seo #cache #title_type
재발: 0회
커밋: ae14afb, e2ce3e9

근본원인: _cache_path(query)가 title_type 무시. Type=B와 Type=C가 동일 캐시 공유.
Type=B 호출이 used_titles 소진 → Type=C는 0개 반환.

체크리스트:
- 캐시 키 생성 시 모든 구분 파라미터가 입력에 포함됐는지
- 해시 구분자는 입력 문자열에 절대 나올 수 없는 문자로
- 기존 캐시 자동 무효화 리스크 검토

해결: _cache_path(query, title_type) 시그니처 확장 + "|" 구분자 사용

### [E069] AB 테스터 백그라운드 stdin 블로킹 (2026-04-17)
태그: #ab-tester #stdin #popen
재발: 0회
커밋: 7e22e3d

근본원인: _check_chrome_running()의 input()이 Step G Popen 자식 프로세스에서 무한 대기.
백그라운드 프로세스는 stdin 없음 → input() 영구 블로킹.

체크리스트:
- 백그라운드 실행 파일에 input() 잔존 여부
- --no-interactive / --headless 같은 플래그로 우회 경로 제공
- Chrome 프로필 충돌 시 실패 경로 (강제 종료 vs 에러 후 exit)

해결: --no-interactive 플래그 추가, Step G Popen에서 전달

### [E070] description 템플릿 {{PLAYLIST_LINK}} 미치환 (2026-04-17)
태그: #seo #template #playlist_link
재발: 0회
커밋: c282fd7

근본원인: Single 영상 description에 {{PLAYLIST_LINK}} 템플릿 마커 삽입만 하고 치환 로직 없음.
플리 업로드 완료 시점에서야 video_id 확정됨 → 단곡 업로드 직전 치환 필요.

체크리스트:
- 모든 템플릿 마커 ({{XXX}}) 치환 시점 명시
- 치환 실패 시 fallback (마커 제거 vs 원본 유지)
- 치환 후 grep으로 마커 잔존 확인

해결: 단곡 업로드 직전 replace("{{PLAYLIST_LINK}}", f"https://youtu.be/{playlist_video_id}")

