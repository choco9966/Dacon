## Don't overfit 솔루션 

출처 :  https://dacon.io/competitions/official/196878/codeshare/416

전처리 

- target 변수의 outlier를 제거하기 위해 평균을 기준으로 3sigma 외의 값들 clipping

변수 

- Time으로부터 생성하는 변수 : month, week, weekday, day, hour
- weekend와 holiday를 이용하여, 휴일을 정의
- 직전주차에 사용한 전력량의 std와 mean을 변수로 사용(only hour predict) -> 모델에 추세에 대한 정보를 부여하는 목적

변수 세트 

- include temp and hour predict
- exclude temp and hour predict
- exclude temp and day, month predict

모델 

- 각각의 모델을 만듬으로서 총 200개의 모델 생성
  - 이 부분에 대한 내용이 없어서 개인적으로 물어보기