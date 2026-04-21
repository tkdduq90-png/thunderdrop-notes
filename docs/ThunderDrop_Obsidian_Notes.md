# ThunderDrop Master Knowledge Base
*최종 업데이트: 2026-04-20*

## 📍 최근 세션
*최신 순서, 최대 10개까지 유지. 오래된 엔트리는 자동 삭제.*

### 2026-04-20 — POMADE 플리 #1 9곡 역공학 완료 (Tom Hardy 10곡 SaaS 준비)

#### 완료
- POMADE 조회수 1위 플레이리스트 9곡 Suno 1-2-3차 검증 완료
- 2-of-3 다수결로 9개 공식 확정 (BPM/Key/장르)
- Notion 1장 통합 구조: https://www.notion.so/3489b7a1c23f81b694f5d9a49b86d225
- 9곡 전체 Suno Full Style + 풀 가사 저장 완료

#### 9개 확정 공식 (Tom Hardy 10곡 매핑 템플릿)
| # | 곡 | BPM | Key | 장르 |
|---|---|---:|-----|-----|
| 1 | NO MERCY IN MY WAKE | 150 | D minor | Hard rock + industrial + electronic |
| 2 | UNCHAIN THE THUNDER | 150 | D minor | Hard rock + heavy blues |
| 3 | BORN FOR THE IMPACT | 85 | B minor | Cinematic hard rock + industrial |
| 4 | RISE FROM THE FIRELINE | 160 | B minor | Modern hard rock + cinematic orchestral |
| 5 | ASHES DON'T HOLD ME | 160 | D minor | Nu-metal + rap rock fusion |
| 6 | STAND IN THE FLAMES | 150 | E minor | Modern hard rock + cinematic |
| 7 | ENTER THE STORM | 85 | D minor | Modern hard rock + alt metal |
| 8 | WHEN THE GROUND SHAKES | 150 | B minor | Nu-metal + rap rock |
| 9 | BREAK THE HORIZON | 150 | B minor | Cinematic arena rock |

#### 플리 전체 분포
- BPM: 150 BPM = 5곡 / 160 BPM = 2곡 / 85 BPM = 2곡
- Key: D minor = 4곡 / B minor = 4곡 / E minor = 1곡 (모두 minor)
- 공통 DNA: Distorted electric guitar + Gritty 남성 보컬 + Compressed 헤비 드럼 + 4/4 time

#### 다음 작업 우선순위
1. Tom Hardy Track 1 "Bronson's Cell" 실제 Suno 생성 테스트
2. Tom Hardy 10곡 트랙리스트를 9개 확정 공식에 매핑 재설계
3. 나머지 9곡 가사 작성 + Suno 생성
4. 신규 3번째 채널명 + 시각 IP + Gmail 생성
5. thumbnail_service 신규 채널 적용

#### 핵심 교훈
- N=2 샘플로 전략 결론 금지: 2차만 보고 "도박 곡" 판정한 것이 오류, 3차로 2차가 이상치 판명
- Suno 분석은 대부분 안정적, 최소 2회 일치 확인이 신뢰 기준
- 마스터 오 지시 위반 3회 발생: 섹션 분리 난립, 서브페이지 무분별 생성, transcript 파싱 오류 (E040)

### 2026-04-18
- **Gym Music 본채널 설계 리서치 — 신규 3번째 채널 방향 확정**
  - YACHA D방향(Drift/Aggressive Phonk) 폐기, 옵션4(Gym Music 일반) 확정
  - YACHA/3CROW = 테스트 채널 격하, 신규 채널 = Gym Music 본채널
  - 한국 시장 부적합 확인(전문 채널 부재) → 글로벌 단일 진입 확정, 10만급 큐레이션 목표
- **3단계 시장 리서치 스크립트 실행**
  - v1(gym_music_research.py): 14채널/67영상, 트랙리스트 추출 97% 실패
  - v2(gym_music_analysis_v2.py): 7월2025 시장 변곡점 발견 (Phonk/Bass House/Trap 부상)
  - v3(gym_music_research_v3.py): 글로벌+한국+일본 3시장 분리, regionCode 강제, 91채널 통과, 글로벌 10만+ 채널 18개 식별
- **글로벌 7채널 심층 분석 (gym_channels_deep_analysis.py)**
  - CURSEDEVIL(55.6만): 시리즈IP(ADRENALINE/AURA), 영상당 100만~1270만, 월1회 이하 퀄리티형
  - TRAP WORKOUT MUSIC(53.4만): 죽은 채널, 카탈로그 효과만 (2018년 이후 미활성)
  - Thug Radio(55.9만): 2Pac Hip Hop 집중 616K평균, 12분 단위 영상
  - DJ Naydee(40.5만): Latin/Reggaeton 특화, Workout 키워드 사용이나 실체는 파티 믹스
  - Hard EDM Workout/TheGymBeatsOfficial: 표준화 공식 + 자체음원(Suno유사) 검증
- **핵심 원칙 도출**: 목적분류≠시청자분류 (마스터오 통찰), 자체음원(Suno) 가능 검증됨
- **잠정 설계 모델**: CURSEDEVIL 시리즈IP + Hard EDM Workout 표준화 하이브리드 → "BEAST MODE Vol.1 - Push Day Music (130 BPM, 60min)"
- **미해결 (다음 세션)**
  - 글로벌 10만+ 채널 14개 추가 분석 필요
  - 100만+ 큐레이션 공식 (Revive 122만, DJ Noize 125만, Epidemic Pop 106만)
  - 신규 채널 설계 1차안 (제목/썸네일/Suno 프롬프트)

### 2026-04-17
- **파이프라인 run 후 이슈 6건 진단 → 수정 15건 커밋 완료**
  - Thumnail C2→C 파일명 수정, villain arc(YACHA+3CROW) 제거, Popen 로그 리다이렉트
  - raw_bg variant별 선택(E066), Shorts min(3) 강제, VILLAIN ARC MOOD 제거
  - rear 포즈 제거 + day_idx 날짜 기반 로테이션(E067)
  - AB 테스터 --no-interactive 플래그(E069), PLAYLIST_LINK 치환(E070)
  - single 썸네일 track_title 표시(T-4), Hook 캐시 title_type 분리(E068)
  - Claude Code 알림 hooks, VARIANTS["rear"] 데드코드 제거(00c06e3), 해시 구분자 "|"(e2ce3e9)
- **진단 문서 6건 생성**
  - diagnosis_20260417.md / _part2.md / _part3.md / followup_20260417.md / code_review_20260417.md / _part2.md
- **코드리뷰 3회 실시**
  - 1차: 7건(92828b5까지) — OK
  - 2차: 5건(T-1~T-5, ae14afb까지) — NOT OK (Issue 1/2)
  - 3차: 2건(Issue 수정 후, e2ce3e9까지) — OK
- **신규 에러 E066~E070 기록, 실수 패턴 8~11번 추가**
- **📍 다음 세션 우선순위**
  - 1순위: 파이프라인 run 실측 검증 (수정 15건 효과 확인)
    * Shorts A/B/C 각자 다른 init 생성 확인
    * 영상 내부 텍스트 없음 (raw_bg 사용 확인)
    * A variant 포즈가 closeup/front 로테이션 (rear 미출현)
    * AB 테스트 자동 실행 (10분 후) → ab_bg_yacha.log 기록 확인
    * 단곡 썸네일 line1이 곡명
    * 단곡 description에 플리 URL 삽입 확인
    * Hook 제목 B/C 각자 생성 (0개 반복 없음)
    * villain 키워드 노출 0건
  - 2순위: C 썸네일 원본 교체 (보류 중) — Thumnail C.png 테두리 포함, 마스터 오 이미지 작업 필요
  - 3순위: base_rear.jpg 에셋 삭제 여부 결정 (dead asset)
- **피드백 루프 업데이트**
  - AB 테스터 자동 실행 수정(E069) → STEP 1 상태 ⚠️ → 조건부 ✅ (다음 run 검증 후 확정)

### 2026-04-16
- **Phase E 로깅 인프라 + schedule 백업 시스템 완성**
  - Phase A: mode 2/3 schedule 재사용 confirm (f07aa2f) — 마지막 수정 시간/첫 항목 표시, 기본값 N
  - Phase C: Leonardo B/C fallback 확장 (f75a7d2) — B/C 둘 다 실패 시 raw_path A로 fallback, build_video/build_single_video 양쪽 적용
  - Phase D: single 썸네일 fallback (a2e23d9) — single 실패 시 playlist A 사용, 둘 다 실패 시 업로드 스킵
  - Phase E-1~E-7: 핵심 7개 모듈 logging 전환 (thumbnail_service, seo_scraper, prompt_builder, sheets_logger, beat_video, suno_bot, master_pipeline) — import logging + logger = logging.getLogger(__name__), print → logger.info/warning/error 의도별 분류, CLI 출력은 print 유지, master_pipeline 93→127 logger 확대
  - Phase E-8: schedule 자동 백업 시스템 (ea8fe01) — step3 json.dump 직전 schedule_backups/ 폴더로 자동 복사, 파일명 upload_schedule_{channel_key}_{YYYYMMDD_HHMMSS}.bak.json, 30일 이상 자동 삭제, 백업 실패 시 파이프라인 계속
  - Phase E-9: 코드리뷰 반영 exc_info=True 9곳 추가 (6c09c3f) — master_pipeline 8곳 + seo_scraper 1곳, AST 전수 검증 except 블록 내 exc_info 100% 커버리지
  - 12커밋 분리 (f07aa2f, f75a7d2, a2e23d9, f403249, 61351dd, 5d66de0, e886a58, 08dfbe3, ca33bae, 7793e93, ea8fe01, 6c09c3f)
- **해소된 이슈**
  - AB 테스터 5쌍 동일 제목 영상: Sheets 행 삭제 + YouTube 비공개 삭제, 정상 등록
  - 2xD9gTGN4GU video_type 오분류: 실제 2곡 합본 playlist (버그 아님 확정)
  - Phase A/B schedule 덮어쓰기: Phase E-8 백업 시스템으로 해결
- **E062 studio_ab_tester regenerate YACHA 경로 수정 (d836456, 푸시 완료)**
  - 레거시 generate_thumbnail_variants() → build_and_adapt_yacha() 전환
  - 3CROW/focusarchitect 기존 경로 유지
- **E063 영상 bg_image raw 전환 5개 경로 (4c27d8b, 푸시 완료)**
  - thumbnail_adapter._variants_dict() A_raw/B_raw/C_raw 키 추가
  - generate_crow_thumbnail() 반환 3→4 tuple 확장 (raw_path)
  - master_pipeline playlist/single/shorts 3블록 raw→final→banner fallback
- **E064 YACHA Shorts A 썸네일 16:9 → 9:16 비율 수정 (b64ec60)**
  - yacha_thumbnail_variants.py RESOLUTIONS dict 추가, composite_thumbnail video_type 분기
  - B/C는 thumbnail_service.py RESOLUTIONS 사용으로 정상이었으나 A만 레거시 하드코딩
- **E065 오디오 저장 경로 audio/ 하위 통합 (0f9d769)**
  - pick_best_versions sel_dir/unsel_dir에 "audio" 세그먼트 추가 (2줄)
  - 기존 16폴더 419파일 14.7GB audio/ 하위로 이동 완료
  - 새 구조:
    ```
    YACHA/audio/
    ├── {날짜}_1/        (Suno raw + failed_tracks)
    ├── {날짜}/           (선별 wav)
    └── {날짜}_미선택/    (미선별 wav 아카이브)
    ```
- **학습/원칙 추가**
  - 하드코딩된 해상도 상수는 video_type 분기가 필요한 모듈의 사일런트 버그 원인이 될 수 있음
  - ch_dir/base_dir처럼 의미 구분된 경로 상수가 실제 디렉토리 구조와 일치하지 않으면 운영 혼동 유발
- **백로그 (다음 세션 작업 후보)**
  - pyflakes 미사용 import 정리 5파일 (master_pipeline/prompt_builder/sheets_logger/seo_scraper/beat_video)
  - scripts/test_shorts_916_live.py logging.xxx 28건 → named logger 통일 (LOW)
  - 로그 파일명 통일: master_pipeline vs suno_bot 패턴 불일치 (LOW)
  - _dry_run_titles 단독 호출 시 basicConfig 미설정 → logger 출력 부재 (MEDIUM)
  - W1 upload_schedule.json dead write → 제거 또는 _debug_dump.json 리네임 (LOW)
  - prompt_builder fallback 가시성, Leonardo 잔액 모니터링, verify_upload.py 독립 검증 스크립트

### 2026-04-15
- **History 탭 트랙 단위 로깅 구조 전면 개편 (Phase 4-history)**
  - 이전: 플리 1회 = 트랙 20 row만, 영상 자체 row 없음. 트랙 row가 영상 메타 중복 보유. 음원_프롬프트_전체 첫 트랙만 저장, 가사/BPM 빈값
  - 새 구조: video_type 4종 분리 (track/playlist/single/shorts). 트랙 row는 음원 메타만, 영상 row는 영상 메타만. 책임 분리 매트릭스 적용
  - prompt_builder: schedule[].tracks 배열에 트랙별 (title/suno_prompt/lyrics/bpm) 누적
  - master_pipeline _log_to_sheets: playlist는 log_batch(track row N개) + log_song(playlist row 1개) 분리 호출, 독립 try-except로 부분 실패 격리
  - sheets_logger HEADERS 22컬럼 확정. lyrics/full_prompt truncation 1000/2000자 증설 (기존 500자 가사 짤림 해결)
- **커스텀 곡 수 옵션 신설 (모드 1)**
  - 구성: 1=10곡 / 2=20곡 / 3=커스텀(2~30) 메뉴 추가
  - songs_per_mix 파라미터 master_pipeline → run_channel → _step1 → step1_generate_prompts 전파
  - Shorts 출력 항상 2개 통일 (n=min(2, len(wav_files)))
  - 빠른 검증 사이클 가능 (2곡 모드 약 17분)
- **YACHA thumb_prompt A/B/C 별도 컬럼 저장 (옵션 3)**
  - 기존: 의도적 빈 문자열 ("Phase 4/5에서 별도 작업" 주석 명시)
  - 진단 결과: A=정적 dict, B=Claude Haiku 동적 생성, C=YACHA_C_VARIANTS 순환. 셋 다 의미 있는 prompt
  - 구현: variants_dict에 A_prompt/B_prompt/C_prompt 키 추가 (4-tuple 반환 구조 유지)
  - thumbnail_service _build_variant_a_raw에서 use_prompt를 ThumbnailResult.prompt에 저장 (성공/실패 경로 모두)
  - sheets_logger HEADERS thumb_prompt → thumb_prompt_A/B/C 분리
- **회귀 방지 작업**
  - generate_dashboard.py row 인덱스 22컬럼 스키마 보정 (row[12] thumb_text → row[14], Critical fix)
  - test_sheets_logger.py 신설 — log_song/log_batch 22컬럼 정합성 자동 검증
  - test_thumbnail_adapter.py b_prompt/c_prompt 헬퍼 + 신규 케이스 추가, 19/19 PASS 유지
- **마무리 검증**
  - 2곡 모드 실 테스트 통과: 트랙 분리/통합 row 신설/Single Shorts 트랙 매칭/thumb_prompt A/B/C 분리 모두 OK
  - 5커밋 분리 (c360677, e717951, c992257, d117e69, acf05dd)
  - Leonardo API 잔액 부족 시 모든 썸네일 실패 → Shorts 0개 생성 1회 발생 (충전 후 정상화). 디펜시브 처리는 별건
- **YACHA A 썸네일 텍스트 시스템 본 파이프라인 통합 (Phase 3~4)**
  - Phase 3: A variant COMBO 9조합 generate_variant 연결
  - Phase 4: Claude Sonnet 동적 텍스트 생성 (_generate_a_text)
  - 날짜 서브폴더 {YYYYMMDD} 추가, C variant 텍스트 오버레이 skip
  - BEAST 고정 버그: Haiku→Sonnet 모델 업그레이드, 블랙리스트 {DARK,GRIND,BEAST,IRON}, 프롬프트 5번 반복 조정
  - 핵심 학습: LLM 다양성 한계 — 트랙명만으론 수렴 (5번 시도 모두 BEAST 고정). 진정한 원인 = 입력 데이터 부족
  - 해결: Suno 태그 + 가사 첫 줄 입력 확장 → 5/5 고유 캐치프레이즈 달성 (FORGE PAIN/RISE FURY/BREAK CHAINS/CRUSH BONES/RAGE FUEL)
- **Leonardo API list 응답 방어 (디펜시브 코딩)**
  - _leonardo_i2i_v2() 3곳에 isinstance(body, list) 가드 추가 (e0e8e5a)
  - E051 동일 근본원인 일괄 해결
- **세션 커밋 합계**: 8커밋 (7a3be3f, f78698e, c89e3b8, 3a6dc4d, acf05dd, b975990, e0e8e5a + history 5건)

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
| STEP 3 | 데이터 수집 → Sheets | ✅ 완료 (트랙 단위 + 영상 단위 분리 기록) |
| STEP 4 | 원인 분석 (dashboard) | ⚠️ 수동 단계 |
| STEP 5 | 자동 반영 | ⚠️ 부분 착수 (A 텍스트 동적 생성 완료) |

**현재 병목:** STEP 5 나머지 (B/C 자동 반영, 승자 피드백 루프)

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
- Suno