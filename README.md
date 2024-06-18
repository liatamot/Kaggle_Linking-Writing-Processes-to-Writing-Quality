# Linking-Writing-Processes-to-Writing-Quality

![image](https://github.com/liatamot/Linking-Writing-Processes-to-Writing-Quality/assets/138054658/1d3d7594-263a-4d48-82dd-ad2cbe02f192)

## Team

### Role
- 박성우: EDA, Feature Engineering, Feature Selection
- 김윤겸: 팀장, EDA, Feature Selection, Ensemble
- 김영천: EDA, Feature Engineering, Modle Efficiency 개선
- 이명진: EDA, Ensemble, Feature Engineering


## 1. Competiton Info

### Overview
- 약 5000개의 로그 데이터를 사용하여 글쓰기 프로세스를 분석하고, 작성자의 에세이 품질을 예측하는 것을 목표로 하는 대회
- 학습자의 글쓰기 행동과 성능 간의 관계를 탐구하며, 최종 결과물에 중점을 두기 보다 글쓰기 프로세스에 주목하여 글쓰기의 자율성과 메타인지를 촉진을 도모함
- 참가자들은 SAT 주제에 대해 30분 이내에 논증 에세이를 작성
- 에세이는 최소 200단어, 3개의 문단으로 구성해야 함
- 온라인이나 오프라인 참고 자료 사용 금지
- 글쓰기 도중 2분 이상 활동하지 않거나 새 창으로 이동 시 경고 발생

  
### Evaluation Metrix
- 모델을 통해 예측한 작성자의 에세이 점수와 실제 작성자의 에세이 점수간의 차이를 RMSE(Root Mean Squared Error) 지표를 통해 측정함


### Timeline
- 2023년 12월 19일 ~ 2024년 1월 10일


## 2. Data descrption

### Dataset overview
#### train_logs.csv
- 구두점 및 기타 특수 문자를 제외한 문자 입력은 문자 q로 대체
- id: 에세이의 고유 ID
- event_id: 이벤트의 인덱스, 시간 순으로 정렬됨
- down_time: 다운 이벤트의 시간(밀리초)
- up_time: 업 이벤트의 시간(밀리초)
- action_time: 이벤트의 지속 시간(down_time과 up_time의 차이)
- activity: 이벤트가 속한 활동의 범주
  - Nonproduction: 이벤트가 텍스트를 변경하지 않음
  - Input: 이벤트가 에세이에 텍스트를 추가
  - Remove/Cut: 이벤트가 에세이에서 텍스트를 제거
  - Paste: 이벤트가 붙여넣기로 텍스트를 변경
  - Replace: 이벤트가 텍스트의 일부를 다른 문자열로 교체
  - Move From [x1, y1] To [x2, y2]: 텍스트 일부를 이동
- down_event: 키/마우스가 눌릴 때 이벤트의 이름
- up_event: 키/마우스가 해제될 때 이벤트의 이름
- text_change: 이벤트로 인해 변경된 텍스트 (있는 경우)
- cursor_position: 이벤트 후 텍스트 커서의 문자 인덱스
- word_count: 이벤트 후 에세이의 단어 수

#### test_logs.csv
- 테스트 데이터로 사용되는 입력 로그 데이터로, train_logs.csv와 동일한 필드를 포함하는 예시 로그 데이터

#### train_scores.csv
- 에세이의 고유 ID와 에세이가 받은 6점 만점의 점수를 포함하는 학습 데이터의 점수 파일

#### sample_submission.csv
- test_logs.csv 데이터를 포함하는 실제 평가 시에 사용되는 테스트 세트 데이터(약 2500개의 에세이 로그)

### EDA

- action 지속시간, 키 입력 시작-종료 시간 등의 피처에서 이상치 탐지
- 시간을 역행하는 이상치 탐지
- 주요 결측치 작성자 Score 사분위 점수 기준으로 하위 25%, 중위 50%, 상위 25% 그룹화하여 그룹별 평균값 대체
- 공개된 베이스라인 참고하여 입력값을 기준으로 essay를 복원 (discussion)
- 주어진 train & test 데이터와 sample_submission 파일의 shape의 차이 확인
  - Id별로 그룹핑 및 유의미한 피처를 도출하며 차원 축소 진행
    
### Data Processing

- 기타 자료로 부터 추출한 Feature 활용
  - [논문1] Uncovering the Writing Process through Keystroke Analyses : 주요 지표 참고
  - [논문2] A Case Study with Deceptive Reviews and Essays
- 복원된 essay를 기반으로 생성된 word_len_sum(단어의 길이 합), word_len_mean(단어 길이 평균), 정지 관련 피쳐,
문단 및 단어 간 간격, 구두문자의 개수, burst 관련 피쳐, etc
- 6가지 Feature Selection 수행
  - Pearson Correlation 두 변수 간의 선형 관계를 측정
  - Variance Inflation Factor (VIF) 다중 공선성을 평가
  - Step Forward Selection 특성을 하나씩 추가하면서 모델의 성능을 측정하고 최상의 특성 조합을 찾는 방법
  - Backward elimination 가장 덜 중요한 특성을 하나씩 제거하면서 모델의 성능을 측정, 최적의 특성 집합을 찾을 때까지 반복
  - Recursive Feature elimination 모델을 훈련하고 중요도가 낮은 특성을 반복적으로 제거
  - Embedded Method 모델 훈련 과정에서 특성 선택을 수행하는 방법

## 3. Modeling

### Model Process

- 선형 회귀, 결정 트리, 랜덤 포레스트, 딥러닝 등 다양한 모델을 활용하여 regression 알고리즘을 개발
- Random Forest Regressor, XGBRegressor, LGBMRegressor, timeseries split을 적용한 LGBMRegressor 총 4개의 모델을 상관 관계, feature importance & imputation 지표를 통해 성능을 비교함
- Optuna Library와 Stratified K-Fold 교차 검증을 사용하여 모델을 평가한 후, 평균 성능 점수를 반환하여 하이퍼 파라미터 튜닝 진행

## 4. Issue

### 라벨 불균형 처리
- Stratified kfold를 쓰지만 0.5~6.0 양극으로 치닫을 수록 데이터가 매우 적어짐
- 전통적인 오버샘플링 방식인 Smote를 사용하기엔 보수적인 관점
- 0.5, 1점 등 점수가 상대적으로 낮은 구간은 작성자가 에세이를 매우 대충 쓴 것으로 가정하여 특징 탐색 필요
- 
### 오버피팅 핸들링
- 대다수 방법론에서 오버피팅 발생 (특성 선택, early stopping, 파라미터 튜닝 등)
- 특히 cv 검증에서 early stopping을 보수적으로 가야한다는 것을 알게 됨
- cv에서 early stopping은 지나치게 낙관적인 cv 점수 유발 가능 (LB에서 성능 안좋은 이유)
  - 고정된 estimators 사용을 고민해 볼 수 있음
  - nested CV 사용을 고민해 볼 수 있음

## 5. Result

### Leader Board

- Private Score 28091.5216  →  Public Score 20935.909
- [발표 자료](https://drive.google.com/file/d/1wIzui2uzwiAZLEbkkduyJdITcGKE71qa/view?usp=drive_link)
