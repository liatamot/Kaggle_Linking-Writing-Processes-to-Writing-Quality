## train_logs.csv
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

## test_logs.csv
테스트 데이터로 사용되는 입력 로그 데이터로, train_logs.csv와
동일한 필드를 포함하는 예시 로그 데이터

## train_scores.csv
에세이의 고유 ID와 에세이가 받은 6점 만점의 점수를 포함하는 학습
데이터의 점수 파일

## sample_submission.csv
test_logs.csv 데이터를 포함하는 실제 평가 시에 사용되는 테스트 세트
데이터(약 2500개의 에세이 로그)
