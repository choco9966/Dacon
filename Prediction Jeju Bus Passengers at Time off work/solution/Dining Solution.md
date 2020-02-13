## Dining Solution 

출처 :  https://dacon.io/competitions/official/229255/codeshare/594 

데이터 전처리 

- bus_route_id는 마지막 4개자리 제거 
- station_name는 앞의 2자리만 사용 
- lat_long은 소숫점2번째, 3번째 자리까지 결합하여 각각 사용
- train에만 있거나 test에만 있는 값들은 0으로 처리 

변수 생성 

- 탑승승객의 합 

  ```python
  df['6~8_ride'] = df[['6~7_ride','7~8_ride']].sum(1)
  df['6~9_ride'] = df[['6~7_ride','7~8_ride','8~9_ride']].sum(1)
  df['6~10_ride'] = df[['6~7_ride','7~8_ride','8~9_ride', '9~10_ride']].sum(1)
  df['6~8_takeoff'] = df[['6~7_takeoff','7~8_takeoff']].sum(1)
  df['6~9_takeoff'] = df[['6~7_takeoff','7~8_takeoff','8~9_takeoff']].sum(1)
  df['6~10_takeoff'] = df[['6~7_takeoff','7~8_takeoff','8~9_takeoff', '9~10_takeoff']].sum(1)
  ```

- day를 기준으로 bus, station, station_lat_long들을 groupby해서 위의 탑승승객의 합을 mean, quantile aggregation 

모델 

- LGBM 