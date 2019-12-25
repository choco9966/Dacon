## 주제 : 제주 퇴근시간 버스승차인원 예측 

기간 :  2019년 11 월 11 일 ~ 2019년 12 월 6 일

결과 

![image-20191225222623996](C:\Users\choco\AppData\Roaming\Typora\typora-user-images\image-20191225222623996.png)

### 폴더 구조

```
.
├── code
│   ├── EXTERNAL
│   │   └── crawling-Copy0.ipynb
│   │   └── crawling-Copy1.ipynb
│   │   └── crawling-Copy2.ipynb
│   │   └── crawling-Copy3.ipynb
│   │   └── crawling-Copy4.ipynb
│   │   └── pickle 종합.ipynb
│   │   └── 제주공항 도착 고객.ipynb
│   ├── FEATURE
│   │   └── FE.ipynb
│   │   └── FE_port.ipynb
│   │   └── Make a feature-set.ipynb
│   ├── MODEL
|   │   └── catboost.ipynb
|   │   └── lgbm.ipynb
|   │   └── lgbm-port.ipynb
│   ├── ENSEMBLE
|   │   └── stacking.ipynb
```

### 실행 환경

- python3.6 

### 필요 라이브러리

- LightGBM 2.3.0
- Xgboost 0.83
- Catboost 0.16.5
- pandas 0.25.1
- numpy 1.16.4

### 피쳐 설명

- kmeans를 이용한 비슷한 station_code 확인 
- 날짜에 대한 정보 : 공휴일, 주말, 주중 
- 오전과 정각에 대한 승하차 인원 정보 
- 버스간의 배차간격 

### 외부 데이터

- 공항의 위경도 
- 날씨 
- 행정구역별 인구정보 

### 모델 설명

- lgbm, catboost 

