# ThunderDrop Master Knowledge Base
*최종 업데이트: 2026-04-09*

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
- **2026-04-09:** 로그파일 저장(logs/날짜_채널.log) + upload_history.csv 추가. 체크포인트 재시작 기능 추가(mode 1 전용, logs/checkpoint_채널.json). 클로드 코드 자율 실행 프롬프트 설계 완료 (mode 2 기준, 에러 자동 수정 + fix_log.txt 기록)

### 개발 필요 ❌
- STEP 5: 성과 → 프롬프트/제목/썸네일 자동 반영 로직
- 포스트 프로세싱 (ambient 레이어링)
- 댓글 자동화

### 기술 결정
- Docker suno-api: hCaptcha 서버사이드 감지 → 완전 포기, pyautogui 유일
- FFmpeg: -hwaccel cuda + h264_nvenc (zoompan은 CPU 전용)
- ChromeDriver: 146.0.7680.178 하드코딩
- SONGS_PER_MIX: 20 / 날짜폴더: 2026-04-07_1, _2, _3
- **브랜치 전략:** master 단일 운영 (dev 브랜치 폐기)
- **로그 시스템:** logs/날짜_채널.log + upload_history.csv + checkpoint_채널.json (mode 1 전용)
- **에러 처리:** 클로드 자율 실행 (mode 2) + fix_log.txt 기록

## 🧠 핵심 전략
### 니치 공식
목적 > 장르 > 기타요소 (기타요소는 팬 고착 도구, 신규 유입 아님)

### 제목 ABC
- A: {keyword} 2026 🔥 | MEGA BASS | {brand} (SEO)
- B: 감성 캐치문구 + Gym Phonk Mix 2026
- C: 숫자/행동 자극 + Gym Phonk Mix 2026
- 세계관은 썸네일+설명란 전용, 제목 금지

### 썸네일 ABC
- A: 세계관 Control (init_image 고정)
- B: 용도 직접 연상 (YACHA=근육남+헬스장, 3CROW=야간드라이브)
- C: 세계관+용도 합성
- 비중: 33/33/33 → 승자 50% → 70/30

### 변수 분리
- CTR↓ = 패키징 문제 / 시청지속↓ = 음악 문제 / 구독전환↓ = 정체성 문제

## 📊 데이터 인사이트
- YACHA 초기: 노출 8500~11000, 추천 46~69% → 최근: 노출 29~72, 추천 0%
- 원인: 시청지속 30초 이후 급락
- 3CROW j6_2Rj0kipE: 검색 93.3%, 노출 2400 → Night Drive SEO 유일 성공
- jdvYsTiTydQ 근육남 썸네일: 추천 60.6% → 용도 직접 연상이 세계관보다 효과적

## 🔧 옵시디언 노트 시스템
- 옵시디언 vault: C:\ThunderDrop\
- 파일: docs/ThunderDrop_Obsidian_Notes.md
- 스크립트: scripts/update_obsidian.py
- 컨텍스트 유지: 새 대화 시작시 GitHub raw URL 읽기 → 대화 중 메모 → 마무리시 push
- 역할 분리: 마스터 스트래티지(HTML)=전략/큰 그림, 옵시디언 노트=실무 현황+기술 결정+컨텍스트
- 상태: 세팅 진행 중

## 🚀 SaaS
- USP: 변수→성과 연결 데이터 기반 자동화 루프
- 순서: Analytics수집✅ → 프롬프트×성과매핑 → 가중치반영(70/30) → 검증 → 베타 $49/월

## 📋 TODO
### P0
- [ ] 3CROW 썸네일 3CROW 텍스트 제거
### P1
- [ ] 포스트 프로세싱 / AI Disclosure / SEO 스크래퍼 실전적용
### P2
- [ ] 프롬프트×성과 매핑 / 가중치 반영(70/30) / Fast Exit
### P3
- [ ] SaaS 랜딩페이지 / Udio 연동 / 댓글 감성분석

## 🔧 커맨드
cd C:\ThunderDrop && python master_pipeline.py
python scripts/analytics_collector.py --channel yacha --days 7
python scripts/studio_ab_tester.py --channel yacha --register
python scripts/generate_dashboard.py
python scripts/update_obsidian.py --delta "변경사항"
git add -A && git commit -m "feat: ..." && git push origin master