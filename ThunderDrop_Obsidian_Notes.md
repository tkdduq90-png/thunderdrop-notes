# ThunderDrop Master Knowledge Base
*최종 업데이트: 2026-04-11*

## 🎯 프로젝트 정체성
**One-liner:** 데이터 → 분석 → 자동 반영 → 업로드 → 반복 (무한 루프)
**목표:** 월 $20,000 / 12개월 로드맵 / 채널 풀 20개
**루트:** C:\ThunderDrop\
**GitHub:** https://github.com/tkdduq90-png/music.git (master)

## 🔄 피드백 루프 단계별 현황
| 단계 | 내용 | 상태 |
|------|------|------|
| STEP 1 | SEO · 컨셉 세팅 · AB테스트 | ⚠️ 부분 완료 |
| STEP 2 | 업로드 (플리/단곡/Shorts) | ✅ 완료 |
| STEP 3 | 데이터 수집 → Sheets | ✅ 완료 |
| STEP 4 | 원인 분석 (dashboard) | ⚠️ 수동 단계 |
| STEP 5 | 자동 반영 | ❌ 미착수 |

**현재 병목:** STEP 5 미착수

## 📺 채널 현황
### YACHA
- ID: UCh5vBYhXQKW44U64S_q_teA / 니치: Gym × Phonk × 야차 세계관
- BPM: 150~175 / 색상: 보라+레드
- 고정태그: cowbell hits, chopped vocal chops, mega bass boosted, subwoofer shaking, extended mix, 3 minutes, do not fade early
- 토큰: youtube_token_yacha.pickle
- 오디오 경로: YACHA/audio

### 3CROW
- ID: UC9bWppwPvgtC7sjsgIOMcHQ / 니치: Drive × Techno × 삼족오 세계관
- BPM: 130~150 / 색상: 청록
- 고정태그: mega bass boosted, occasional vocals, vocal stabs on drops, subwoofer shaking, extended mix, 3 minutes, do not fade early
- 토큰: 3Crow\youtube_token_3crow.pickle
- 오디오 경로: 3Crow/audio

### 공용
- Sheets: music-220@thunderdrop.iam.gserviceaccount.com
- 시트 ID: 1e5zAPL05m3Tl0dEhB8tx3DdwpXgge3CpVFzrZttKpjQ
- Suno 모델: chirp-fenix

## ⚙️ 파이프라인 현황
### 운영 중 ✅
- Suno 음원 생성 (pyautogui) / 플리+단곡+Shorts 영상 생성
- Leonardo AI 썸네일 A/B/C / YouTube API 업로드 (매일 21시)
- Analytics 수집 → Sheets / Studio 크롤링 / AB테스트 / dashboard.html
- Task Scheduler (analytics 10시, git push 23시)
- seo_scraper.py: generate_hook_title 추가 (Title B=감성훅/C=숫자자극, 경쟁채널 패턴 분석 기반)
- master_pipeline.py: --dry-run 플래그 추가, 플리/단곡/숏츠 Title A/B/C 생성 연결 완료

### 최근 완료 ✅
- suno_bot.py base_dir 채널별 분리 (YACHA/audio, 3Crow/audio)
- YACHA_DIR 경로 야차→YACHA 통일
- dev 브랜치 복구: neon/shorts/B-C variants + mood word 컨텍스트 YACHA/3CROW 분리
- claude_api() temperature 파라미터 복구
- studio_ab_tester.py 프로필 경로 채널 내부로 통일
- 폴더 구조화: scripts/(독립 유틸), logs/(로그/스크린샷), temp/, seo_cache/
- 루트 잡동사니 정리 (야차 병합, chrome_profile 정리)
- CLAUDE.md 업데이트 (브랜치/커밋/폴더구조/채널 정보)
- **2026-04-09:** dev 브랜치 폐기, master 단일 운영 확정
- **2026-04-09:** thumbnail_builder.py scene prompt KeyError 수정, 3CROW B variant is_b=True 버그 수정
- **2026-04-09:** beat_video.py EQ 16:9=200px/Shorts=300px, gblur sigma 축소, 피드백 루프 제거, 렌더링 진행률 출력 추가
- **2026-04-09:** master_pipeline.py Shorts RMS 상위3개/ABC variant 재작업, PCM 파일명 충돌 수정, stdout flush 추가
- **2026-04-09:** 로그파일 저장(logs/날짜_채널.log) + upload_history.csv 추가. 체크포인트 재시작 기능 추가(mode 1 전용). 클로드 코드 자율 실행 프롬프트 설계 완료 — mode 1 전체 자율실행(에러 자동 수정 + fix_log.txt 기록). 화면 잠금 해제 필수.
- **2026-04-09:** analytics_collector/studio_ab_tester 경로 버그 수정. studio_ab_tester cp949 인코딩 수정. AB테스트 thumb_C 누락 + title_B/C 미기록 버그 수정. Sheets Analytics_Manual 정리 - 144행→75행, 중복 69건 삭제, ab_registered 실제 등록 2건만 유지. 자율실행 프롬프트 완성 (claude --dangerously-skip-permissions + mode1, settings.json 불필요).
- **2026-04-09:** 자율주행 첫 성공 - run_test_auto.py 생성, suno_bot.py cp949 수정, 체크포인트 활용, 3시간 만에 YACHA 5개 비공개 업로드 완료. AB테스트 자율주행 성공 - thumb_c 없을때 thumb_b 폴백 + 버튼 대기 30초 수정, YACHA 2/2 등록. Sheets video_id 누락 영상 추가 (YACHA 7건, 3CROW 6건). 자율주행 프롬프트 확정 (mode1 전체 + AB테스트 포함 버전).
- **2026-04-09:** AB테스트 4개 영상 전부 등록 완료 - DuT9l-v8fFQ(YACHA 플리), HRcpM2vxRk0(YACHA 단곡), j98jZbijN1A(3CROW 플리), Xe91nBNqjXs(3CROW 단곡) 모두 Sheets ab_registered=TRUE 처리
- **2026-04-09:** Shorts 영상효과 beat_video.py 통합(is_shorts=True), color grading 제거, Shorts raw 이미지 사용. 3CROW B 썸네일 6종 확정(좌->우 주행+수평네온+6색). C 썸네일 6종 확정(정적 스포츠카+6색). drawbox 제거. dashboard thumb_B/C 컬럼 인덱스 수정(9->10, 10->11). 썸네일 경로 서브폴더 분기(playlist/single/shorts). AB테스트 0/10 실패 미해결.
- **2026-04-09:** SEO 영상설명/해시태그 전체 적용. 썸네일 keyword_hints 주입. History 21컬럼 확장. analytics_collector 승자제목 자동 업데이트. TEST MODE Sheets skip. Shorts temp 폴더 충돌 수정. Analytics_Manual 정리(87→65행). 미완료: Shorts 실테스트, YouTube 설명 반영, History 기록 확인.
- **2026-04-10:** thumbnails().set() sleep 5초 + 3회 retry 추가 (_upload_one). E023 관련 작업 A/B 시도 후 롤백 (thumbnails().set() 원래 작동 확인).
- **2026-04-10:** studio_crawler.py --no-interactive 플래그 추가 (Task Scheduler 블로킹 방지).
- **2026-04-10:** run_analytics.bat PYTHONUTF8=1 + 로그 리다이렉션 추가. PowerShell 날짜 + append-only 로그 구현.
- **2026-04-10:** Claude API 최적화 완료 - prompt_builder claude_api() model 파라미터 추가 및 모델 ID claude-sonnet-4-6 최신화. crow_generate_intro/generate_hashtags/thumbnail 훅단어/scene JSON Haiku 전환. seo_scraper 클라이언트 singleton 전환. CLAUDE_API_KEY None 방어 및 해시태그 검증 로직 추가.
- **2026-04-11:** AB 테스터 인터랙티브 UI 개발 완료 - 플래그 없이 실행 시 채널/영상/액션 선택 UI 제공. video_type 컬럼 추가 (Shorts AB skip). Chrome 실행 감지로 로그인 유지 문제 해결. sheets_logger/master_pipeline video_type 전달 연동 완료.
- **2026-04-11:** 디버깅 세션 완료 - seo_scraper 설명수집 0→10개 확대, 해시태그 채널제약 추가. master_pipeline Shorts thumb B/C fallback 구현. prompt_builder crow_get_lyrics track_name+fallback 로직 추가. AB테스터 실패원인 규명: Chrome 기존실행 문제 → _check_chrome_running() 모니터링으로 해결.
- **2026-04-11:** AB 테스터 썸네일 모드 선택 UI 개발 완료 - sheets/fallback_a/regenerate 3가지 모드 제공. thumb_a 빈값 경고 추가. Shorts skip 한글 방어 주석 보강.

### 개발 필요 ❌
- STEP 5: 성과 → 프롬프트/제목/썸네일 자동 반영 로직
- 포스트 프로세싱 (ambient 레이어링)
- 댓글 자동화
- Shorts 실테스트, YouTube 설명 반영, History 기록 확인
- **예정:** #3 다음사이클 AB테스터 정상작동 확인 필요

### 기술 결정
- Docker suno-api: hCaptcha 서버사이드 감지 → 완전 포기, pyautogui 유일
- FFmpeg: -hwaccel cuda + h264_nvenc (zoompan은 CPU 전용)
- ChromeDriver: 146.0.7680.178 하드코딩
- SONGS_PER_MIX: 20 / 날짜폴더: 2026-04-07_1, _2, _3
- **브랜치 전략:** master 단일 운영 (dev 브랜치 폐기)
- **YouTube API 안정성:** thumbnails().set() sleep 5초 + 3회 retry (E023 대응)
- **Task Scheduler 호환성:** studio_crawler.py --no-interactive 플래그 (블로킹 방지)
- **배치 인코딩:** run_analytics.bat PYTHONUTF8=1 + PowerShell 날짜 + append-only 로그
- **Claude API:** model=claude-sonnet-4-6 (최신), Haiku 전환 (훅단어/scene JSON), singleton 클라이언트, CLAUDE_API_KEY None 방어
- **AB 테스터 UI:** 인터랙티브 선택 모드 (플래그 불필요), video_type 컬럼 기반 Shorts skip, Chrome 감지 로그인 유지, _check_chrome_running() 모니터링, 썸네일 모드 선택 (sheets/fallback_a/regenerate), thumb_a 빈값 경고
- **SEO 수집:** 설명 10개 수집, 채널별 해시태그 제약 (YACHA/3CROW 분리)
- **Shorts 썸네일:** B/C fallback 구현 (C 없을 시 B 사용)
- **가사 수집:** track_name 기반 + fallback 로직 (crow_get_lyrics)

## 🧠 핵심 전략
### 니치 공식
목적 > 장르 > 기타요소 (기타요소는 팬 고착 도구, 신규 유입 아님)

### 제목 ABC
- A: seo_generate_title (경쟁채널 키워드 분석 기반 SEO)
- B: generate_hook_title type=B (감성/상황 훅 + fixed_keyword)
- C: generate_hook_title type=C (숫자/행동 자극 훅 + fixed_keyword)
- YACHA fixed_keyword: "Gym Phonk Mix 2026"
- 3CROW fixed_keyword: "Night Drive Techno Mix 2026"
- 플리/단곡: 영상 1개 업로드 + YouTube AB테스트 A/B/C 등록
- 숏츠: 숏츠1=A, 숏츠2=B, 숏츠3=C 각각 업로드
- 세계관은 썸네일+설명란 전용, 제목 금지

### 썸