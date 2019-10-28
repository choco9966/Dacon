## 느낀점 

- Meter_id별로 시계열을 적용해야 할 때, 각 Meter_id별 시계열을 튜닝해줘야하는 문제점이 있었음. 이를 단순하게 해결하려고 Grid Search방식을 사용했지만 이는 비효율적인 방법임. 이를 보다 효율적으로 하려면 시계열에 대한 공부가 더 필요하다고 느꼈음. 
- 위의 방법을 대체하기 위해서 정형데이터 형태로 만들어서 LightGBM을 돌리려고 했지만, 미래를 예측하기 위해서는 과거 및 현재가 어떻게 될지를 반영해줄 feature들이 필요했음. 
- 이를 반영하고자, 1시간전의 전력소요량, 24시간전의 전력소요량, 일주일전의 전력소요량등의 변수를 도입했음. 
- 하지만, 향후 24시간을 예측하는 경우 1시간전의 전력소요량변수는 처음 1시간만 결측치가 없고 남은 23시간은 결측치인 문제가 있었음. 이를 해결하기 위해서 1시간 예측하고 이를 다시 변수로 넣는 방법이 있었으나 시간상의 문제로 시도해보지는 못함. 