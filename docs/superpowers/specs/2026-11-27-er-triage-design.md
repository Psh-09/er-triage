# 응급실 중증도 분류 프로젝트 설계 (2026-11-27)

## 의도

119구급상황관리센터가 이송 전 현장 정보만으로 중증(KTAS 1~2) 환자를 가려내,
권역응급센터로 보낼 환자를 정하도록 돕는다. (응급의료법 개정안: 2026-10 국회 통과, 2027-04 시행 예정 보도)

## 주제 카드 (학생 작성)

- 질문: 환자의 응급 수준은 어느정도인가?
- 결정자: 119구급상황관리센터
- 결정 내용: KTAS 등급 판단 (중증이면 권역응급센터로 이송)
- 정답: 전문가 KTAS(`KTAS_expert`) 1~2 → 1(중증), 3~5 → 0
- 기준 이유: 3급은 중증 의심 응급환자이고 2급부터 긴급 환자이므로, 확실히 긴급한 환자 이상만 분류
- 비용: FN 10, FP 2.5 (놓치는 쪽이 4배 비쌈)
- 1순위 지표: Recall@K ≥ 0.95, 정밀도 ≥ 0.8 유지
- 예산 K: 55 (권역센터가 55명까지 받을 수 있어서). Valid 중증 약 49명 중 47명 이상이 상위 55명에 들어가야 목표 달성

## 데이터

- 출처: Moon SH et al. (2019) PLoS ONE 14(9): e0216972, 부록 S1(데이터)·S2(열 설명)
- 1,267명 × 24열, KTAS 1~2 비율 19.4%(246명)
- 결측·이상값: Saturation 688 결측, NRS_pain `#NULL!` 556, 활력징후 '측불' 18~25건

## 특성 (현장에서 알 수 있는 것만)

사용 12개: Sex, Age, Injury, Mental, Pain, NRS_pain, SBP, DBP, HR, RR, BT, Saturation

제외:

- 정답 누수: KTAS_RN, Diagnosis in ED, Disposition, Error_group, mistriage, Length of stay_min
- 응급실 안에서만 아는 정보: Patients number per hour, KTAS duration_min
- 결정 대상·배포 시 무의미: Group, Arrival mode
- 자유 텍스트: Chief_complain

## 분할·검증

- 층화 60/20/20 (Train 760 / Valid 253 / Test 254), random_state=42
- 순위표: Valid 점수 + Train+Valid 5-fold 교차 검증(평균 ± 표준편차)
- Test는 `09_final_test.py` 한 곳에서 한 번만 읽음

## 접근 방식

- 채택: 번호 붙인 단계별 스크립트(수업 저장소 `02-data-preprocessing`과 같은 방식). 구술 확인 때 파일 하나씩 설명하기 쉬움.
- 버림: 노트북 하나에 전부 → Test를 여러 번 읽기 쉽고 재현 순서가 흐려짐.
- 버림: 패키지/모듈 구조 → 6시간 과제에 과함.

## 저장소 구조

```
er-triage/
├─ CLAUDE.md, README.md, ai_log.md, requirements.txt, scoreboard.csv
├─ data/raw/ (원본, 수정 금지), data/ktas.csv, data/train|valid|test.csv
├─ src/01_convert.py   엑셀 → CSV (파이썬 기본 기능, openpyxl 추가 안 함)
├─ src/02_split.py     층화 분할
├─ src/prep.py         특성 목록 + Pipeline (결측 채우기·스케일링은 Train 기준)
├─ src/03_explore.py   탐색 그래프 2개 (Train만)
├─ src/04_baseline.py  아무것도 안 함 + 사람 규칙
├─ src/05_models.py    로지스틱 포함 4개 이상 → scoreboard.csv
├─ src/06_overfit.py   하이퍼파라미터 1개, Train·Valid 그래프
├─ src/07_threshold.py 임계값별 정밀도·재현율, Recall@K (비용 기준 임계값 0.2 참고)
├─ src/08_explain.py   특성 중요도 / SHAP
├─ src/09_final_test.py Test 1회
└─ figures/            보고서 그림 5개
```

## 하지 않는 것

- 요청하지 않은 모델·라이브러리 추가
- Test를 두 번 이상 읽기, 전체 데이터로 기준값 계산
- 주호소 텍스트 처리, 딥러닝

## 학생에게 남은 결정

- 사람 규칙 기준선 (3차시)
- 실험 전 예상, 최종 모델 선택, 제안, 회고
