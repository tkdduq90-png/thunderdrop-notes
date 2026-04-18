# Phonk 오디오 특성 분석 리포트 — 2026-04-17

**생성**: `scripts/music_feature_analysis.py`  
**Spotify 트랙**: 40개  
**YouTube tracklist 트랙**: 10개  
**교차 집합 (A∩B)**: 10개  

---

## 1. 장르별 BPM 분포

### Gym Phonk (n=40)
BPM 범위: 145.0 – 203.5  
BPM 중앙값: 174.2  

```
   145.0 | ████████████████████████████████████████ 4
   150.8 | ████████████████████████████████████████ 4
   156.7 | ████████████████████████████████████████ 4
   162.6 | ████████████████████████████████████████ 4
   168.4 | ████████████████████████████████████████ 4
   174.2 | ████████████████████████████████████████ 4
   180.1 | ████████████████████████████████████████ 4
   185.9 | ████████████████████████████████████████ 4
   191.8 | ████████████████████████████████████████ 4
   197.7 | ████████████████████████████████████████ 4
```

---

## 2. Energy / Loudness 평균값 (장르별)

| 장르 | Energy mean | Loudness mean (dB) | Danceability mean |
|------|-------------|-------------------|-------------------|
| Gym Phonk | 0.92 | -4.5 | 0.78 |

---

## 3. 교차 집합 (A∩B) 통계

  tempo            mean= 151.75  p10=146.35  p25=148.38  p50=151.75  p75=155.12  p90=157.15  (n=10)
  energy           mean=   0.92  p10=  0.92  p25=  0.92  p50=  0.92  p75=  0.92  p90=  0.92  (n=10)
  loudness         mean=  -4.50  p10= -4.50  p25= -4.50  p50= -4.50  p75= -4.50  p90= -4.50  (n=10)
  danceability     mean=   0.78  p10=  0.78  p25=  0.78  p50=  0.78  p75=  0.78  p90=  0.78  (n=10)
  valence          mean=   0.35  p10=  0.35  p25=  0.35  p50=  0.35  p75=  0.35  p90=  0.35  (n=10)

### 검증된 트랙 Top 20 (교차 집합, BPM 내림차순)

| # | 트랙 | 아티스트 | BPM | Energy | Loudness |
|---|------|----------|-----|--------|----------|
| 1 | Test Track | Test Artist | 158.5 | 0.920 | -4.5 |
| 2 | Test Track | Test Artist | 157.0 | 0.920 | -4.5 |
| 3 | Test Track | Test Artist | 155.5 | 0.920 | -4.5 |
| 4 | Test Track | Test Artist | 154.0 | 0.920 | -4.5 |
| 5 | Test Track | Test Artist | 152.5 | 0.920 | -4.5 |
| 6 | Test Track | Test Artist | 151.0 | 0.920 | -4.5 |
| 7 | Test Track | Test Artist | 149.5 | 0.920 | -4.5 |
| 8 | Test Track | Test Artist | 148.0 | 0.920 | -4.5 |
| 9 | Test Track | Test Artist | 146.5 | 0.920 | -4.5 |
| 10 | Test Track | Test Artist | 145.0 | 0.920 | -4.5 |

---

## 4. Suno 프롬프트 추천

- **tempo 범위**: `160 – 189 BPM` (IQR 기반)
- **energy**: `0.92` (mean), 목표 ≥ 0.92
- **loudness**: `-4.5 dB` (mean) — Suno: high gain, bass boost
- **danceability**: `0.78` (mean)

**Suno 프롬프트 태그 제안**:
```
aggressive phonk, dark phonk, gym phonk,
BPM 160-189, high energy, heavy bass, 808 bass,
distorted cowbell, vocal chops, trap hi-hats, mega bass boosted,
no fade, extended mix, subwoofer, dark atmosphere
```

---

*생성: 2026-04-17T23:21:58.750005*