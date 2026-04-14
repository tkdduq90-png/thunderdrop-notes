# ThunderDrop Master Knowledge Base
*최종 업데이트: 2026-04-14*

## 📍 최근 세션
*최신 순서, 최대 10개까지 유지. 오래된 엔트리는 자동 삭제.*

### 2026-04-14
- AB 테스터 2/2 성공 — E024 "0/10 실패" 해결 (dedicated profile 작동 증명)
- Phase 5 §9.5: YACHA 4/10~ 전면 ghost 확정 — E046 중복 콘텐츠 무음 삭제
  - uploads playlist 36건 중 4/10 이후 영상 0건
  - videos().insert() 200 OK 반환하되 YouTube가 무음 삭제
  - 4/13 동일 schedule.json 4~5회 재실행이 트리거 추정
- Phase 5 §10~13: Shorts 썸네일 9:16 전면 재설계
  - 문제: 16:9 썸네일이 Shorts에 반영 안 됨 (Google Issue #381127084 케이스)
  - 해결: Leonardo 768×1344 생성, beat_video crop_prefix 제거, 썸네일 API 스킵
  - 영상 첫 프레임 = Leonardo 이미지 = 피드 썸네일 역할
  - 라이브 검증 완료 (YMiKD1p0kl0 Studio 육안 확인)
- SEO/카니발라이제이션 정리 7건
  - YACHA: night drive/car music 키워드 제거, villain arc → beast mode, 세계관 단어 제거
  - 3CROW: gym/workout 키워드 제거 (config tags + hashtags 폴백)
- 테스트 스크립트 3개 신규 (scripts/test_shorts_*.py)
- 설계 문서 신규 (docs/shorts_thumbnail_redesign.md)
- 11 commits pushed to origin/master

### 2026-04-13
- Phase 5 미스터리 디버깅: 가설 6번 뒤집힘 (타이밍/shorts정책/video_id파싱/중복silent deletion/채널제재/스코프부족/response파싱), 원본 증거 소실로 검증 불가
- 유력 가설 2개 남음: H(OAuth 스코프 youtube.upload만으로 비공개 영상 API 조회 불가) + I(_upload_one response 파싱 오류)
- E050 신규 기록 (가설 과신 메타 에러, E038 재발)
- 내일 재개 시작점: uploads playlist 열거 (docs/phase5_diagnosis.md 섹션 9.4)
- Phase 6 착수 조건: 원인 확정 후, 다음 세션 권장 1순위: Step 2 uploads playlist 열거로 가설 H/I 확정
- update_obsidian.py 리팩터: shutil.copy2 PermissionError 제거, LOCAL_PATH notes-repo/docs/ 직접 지정, subprocess utf-8 encoding, git diff 오탐 방지
- CLAUDE.md 문서 경로 절대경로 명시, ErrorLog E043~E045 기록
- thumbnail_service.py dead code 정리: YACHA_B_NEGATIVE_PROMPT/C_PARAMS 12줄 제거
- Phase 5 진단 완료: playlist 404 = YouTube 중복 콘텐츠 무음 삭제, schedule.json 5회 재실행 → 같은 플리 5번 업로드, 4·5번째 ghost video_id 확인, 실제 성공 = sSo9TXRffLw
- CS00SUzgJtE Shorts 403 = 임시 처리 제한 (시간 경과 후 자동 해제), _upload_one() video_id 파싱 정상

---

## 🎯 프로젝트 정체성
**One-liner:** 데이터 → 분석 → 자동 반영 → 업로드 → 반복 (무한 루프)
**목표:** 월 $20,000 / 12개월 로드맵 / 채널 풀 20개
**루트:** C:\ThunderDrop\
**GitHub:** https://github.com/tkdduq90-png/music.git (master)

## 🔄 피드백 루프 단계별 현황
| 단계 | 내용 | 상태 |
|------|------|------|
| STEP 1 | SEO · 컨셉 세팅 · AB테스트 | ✅ 완료 |
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
- 오디오 경로: YACHA\audio

### 3CROW
- ID: UC9bWppwPvgtC7sjsgIOMcHQ / 니치: Drive × Techno × 삼족오 세계관
- BPM: 130~150 / 색상: 청록
- 고정태그: mega bass boosted, occasional vocals, vocal stabs on drops, subwoofer shaking, extended mix, 3 minutes, do not fade early
- 토큰: 3Crow\youtube_token_3crow.pickle
- 오디오 경로: 3Crow\audio
- Shorts 썸네일: 9:16 Leonardo 768×1344 전용 생성, thumbnails.set() API 스킵, 영상 첫 프레임 = 피드 썸네일

### FocusArchitect
- 오디오 경로: FocusArchitect\audio
- 상태: 신규 채널 추가

### 공용
- Sheets: music-220@thunderdrop.iam.gserviceaccount.com
- 시트 ID: 1e5zAPL05m3Tl0dEhB8tx3DdwpXgge3CpVFzrZttKpjQ
- Suno 모델: chirp-fenix

## ⚙️ 파이프라인 현황
### 운영 중 ✅
- Suno 음원 생성 (pyautogui) / 플리+단곡+Shorts 영상 생성
- Leonardo AI 썬네일 A/B/C / YouTube API 업로드 (매일 21시)
- Analytics 수집 → Sheets / Studio 크롤링 / AB테스트 / dashboard.html
- Task Scheduler (analytics 10시, git push 23시)
- seo_scraper.py: generate_hook_title 추가 (Title B=감성훅/C=숫자자극, 경쟁채널 패턴 분석 기반)
- master_pipeline.py: --dry-run 플래그 추가, 플리/단곡/숏츠 Title A/B/C 생성 연결 완료

### 최근 완료 ✅
- **2026-04-14:** Shorts 9:16 재설계 완료. Leonardo 768×1344 생성, beat_video crop_prefix 제거, thumbnails.set() Shorts 스킵. 첫 프레임 = Leonardo 이미지 = 피드 썸네일. 라이브 검증 통과 (YMiKD1p0kl0).
- **2026-04-14:** Phase 5 §9.5 — YACHA 4/10~ 전면 ghost 확정. E046 확대. uploads playlist 36건 중 4/10 이후 0건. videos.insert 200 OK 후 무음 삭제.
- **2026-04-14:** AB 테스터 2/2 성공. E024 해결. dedicated profile 작동 증명.
- **2026-04-14:** SEO/카니발라이제이션 정리 7건. YACHA drive 키워드/세계관 단어 제거, 3CROW gym 키워드 제거.
- suno_bot.py base_dir 채널별 분리 (YACHA\audio, 3Crow\audio, FocusArchitect\audio)
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
- **2026-04-09:** Shorts 영상효과 beat_video.py 통합(is_shorts=True), color grading 제거, Shorts raw 이미지 사용. 3CROW B 썬네일 6종 확정(좌->우 주행+수평네온+6색). C 썬네일 6종 확정(정적 스포츠카+6색). drawbox 제거. dashboard thumb_B/C 컬럼 인덱스 수정(9->10, 10->11). 썬네일 경로 서브폴더 분기(playlist/single/shorts). AB테스트 0/10 실패 미해결.
- **2026-04-09:** SEO 영상설명/해시태그 전체 적용. 썬네일 keyword_hints 주입. History 21컬럼 확장. analytics_collector 승자제목 자동 업데이트. TEST MODE Sheets skip. Shorts temp 폴더 충돌 수정. Analytics_Manual 정리(87→65행). 미완료: Shorts 실테스트, YouTube 설명 반영, History 기록 확인.
- **2026-04-10:** thumbnails().set() sleep 5초 + 3회 retry 추가 (_upload_one). E023 관련 작업 A/B 시도 후 롤백 (thumbnails().set() 원래 작동 확인).
- **2026-04-10:** studio_crawler.py --no-interactive 플래그 추가 (Task Scheduler 블로킹 방지).
- **2026-04-10:** run_analytics.bat PYTHONUTF8=1 + 로그 리다이렉션 추가. PowerShell 날짜 + append-only 로그 구현.
- **2026-04-10:** Claude API 최적화 완료 - prompt_builder claude_api() model 파라미터 추가 및 모델 ID claude-sonnet-4-6 최신화. crow_generate_intro/generate_hashtags/thumbnail 훅단어/scene JSON Haiku 전환. seo_scraper 클라이언트 singleton 전환. CLAUDE_API_KEY None 방어 및 해시태그 검증 로직 추가.
- **2026-04-11:** AB 테스터 인터랙티브 UI 개발 완료 - 플래그 없이 실행 시 채널/영상/액션 선택 UI 제공. video_type 컬럼 추가 (Shorts AB skip). Chrome 실행 감지로 로그인 유지 문제 해결. sheets_logger/master_pipeline video_type 전달 연동 완료.
- **2026-04-11:** 디버깅 세션 완료 - seo_scraper 설명수집 0→10개 확대, 해시태그 채널제약 추가. master_pipeline Shorts thumb B/C fallback 구현. prompt_builder crow_get_lyrics track_name+fallback 로직 추가. AB테스터 실패원인 규명: Chrome 기존실행 문제 → _check_chrome_running() 모니터링으로 해결.
- **2026-04-11:** AB 테스터 썬네일 모드 선택 UI 개발 완료 - sheets/fallback_a/regenerate 3가지 모드 제공. thumb_a 빈값 경고 추가. Shorts skip 한글 방어 주석 보강.
- **2026-04-11:** studio_ab_tester.py regenerate 모드 구현 완료 - thumbnail_builder.generate_thumbnail_variants() 연동. sys.path 루트 추가로 import 해결. 실패 시 fallback 제거→스킵. shutil 상단 이동. Leonardo 크레딧 경고 추가.
- **2026-04-11:** _channel_post_fx YACHA/3CROW 효과 전체 제거 (chromashift, hue oscillation, tblend, rgbashift). _fallback_video hwaccel_output_format 잔존(저위험). _channel_feedback no-op 미구현. 3CROW 썬네일 텍스트 원인 규명: init_image+A프롬프트 no text 누락.
- **2026-04-12:** Leonardo v2 API 발견사항 정리 - WIDTH 1472 → v2 validation 에러 (허용값: 672/768/832/864/896/1024/1152/1184/1248/1344). imagePrompts 배열에 객체 불가, 문자열만 허용. v2 폴링 엔드포인트

## 📍 현재 상태 — 다음 세션 권장 우선순위

1. **Phase 6 E046 방어 로직 구현** (1~2 세션)
   - 중복 업로드 방지 (upload_history 기반 skip)
   - 업로드 후 videos().list 즉시 존재 확인
   - YACHA 신규 음원으로 ghost 재검증
2. **Analytics zeros 문제 재확인** (30분)
   - 어제 3CROW 5건 recording zeros 현상
   - 24h 경과 후 데이터 차는지 확인
   - 아니면 audienceRetentionReports API 실제 동작 조사
3. **Phase 3 3CROW/FocusArchitect thumbnail_service 이전** (2~3 세션)
   - Phase 1/2/4 패턴 재적용
   - keyword_hints 어댑터 확장 필수
4. (장기) verify_upload.py 독립 검증 스크립트
5. (장기) B-keyword 신기능 도입 논의