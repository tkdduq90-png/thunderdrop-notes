# ThunderDrop Master Knowledge Base
*최종 업데이트: 2026-04-16*

## 📍 최근 세션
*최신 순서, 최대 10개까지 유지. 오래된 엔트리는 자동 삭제.*

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