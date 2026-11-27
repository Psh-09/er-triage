# 프로젝트 규칙

## 목표

- 질문: 환자의 응급 수준은 어느정도인가?
- 결정자: 119구급상황관리센터. 이송 전 현장 정보만으로 중증 환자를 골라 권역응급센터로 보낸다.
- 정답: `KTAS_expert` 1~2 → 1(중증), 3~5 → 0
- 1순위 지표: Recall@K ≥ 0.95, 정밀도 ≥ 0.8 유지. 예산 K = 55 (권역센터가 55명까지 받을 수 있어서)
- 틀림의 비용: FN(중증을 놓침) 10, FP(경증을 중증으로 봄) 2.5

## 데이터 규칙

- data/test.csv 는 내가 '최종 판정'이라고 말하기 전까지 절대 읽지 않는다. 읽는 코드는 `src/09_final_test.py` 한 곳뿐이다.
- `data/raw/` 의 원본 파일은 수정하지 않는다.
- 특성은 아래 12개만 쓴다. 119가 현장에서 알 수 있는 값이다.
  Sex, Age, Injury, Mental, Pain, NRS_pain, SBP, DBP, HR, RR, BT, Saturation
- 아래 열은 특성으로 쓰지 않는다.
  - 정답 누수: KTAS_RN, Diagnosis in ED, Disposition, Error_group, mistriage, Length of stay_min
  - 응급실 안에서만 아는 값: Patients number per hour, KTAS duration_min
  - 결정 대상이거나 실제 사용 때 의미 없는 값: Group, Arrival mode
  - 자유 텍스트: Chief_complain
- 분할은 정답 비율을 유지하는 층화 60/20/20 (Train / Valid / Test).

## 코드 규칙

- 전처리는 Pipeline 으로 묶고 기준값(결측 채우기, 스케일링)은 Train 에서만 구한다.
- random_state=42 로 고정한다.
- 모델 결과는 scoreboard.csv 에 한 줄씩 남긴다. Valid 점수와 Train+Valid 5-fold 교차 검증(평균 ± 표준편차)을 함께 적는다.
- 파이썬 환경은 `~/projects/ai-modeling/.venv` 를 쓴다. 수업 환경과 같은 버전을 유지한다.
- 코드를 실행하기 전에 무엇을 할지 3줄로 먼저 설명한다.
- 실험을 돌리기 전에 내가 ai_log.md 에 예상을 적었는지 확인하고, 안 적었으면 먼저 적게 한다.
- 주석과 설명은 한국어로, 고등학생이 이해할 수 있게 쓴다.
- 내가 요청하지 않은 모델이나 라이브러리를 추가하지 않는다.

## 참고

- 설계 문서: docs/superpowers/specs/2026-11-27-er-triage-design.md
- 데이터 출처: Moon SH et al. (2019) PLoS ONE 14(9): e0216972 부록 S1, S2
