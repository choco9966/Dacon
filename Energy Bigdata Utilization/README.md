## 주제 : 각 미터기별 전기소요량 예측 

팀명 : TEAM-EDA(김현우, 이해중)

기간 : 2019.10.8 ~ 2019.10.27 

### 폴더 구조

```
.
├── Input
├── Code
│   ├── EDA
│   │   └── 1. Data Exploratory Analysis.ipynb
│   ├── PREPROCESSING
│   │   └── 2. Preprocessing.ipynb
│   ├── MODEL
|   │   └── 3. Modeling LightGBM.ipynb
|   │   └── 3. Modeling ARIMA.ipynb
|   │   └── 3. Modeling ProPhet.ipynb
|   │   └── 3. Modeling LSTM.ipynb
│   ├── ENSEMBLE
|   │   └── 4. Ensemble.ipynb
└── Experience.md
```

### 실행 환경

- python3.6 

### 필요 라이브러리

- Tensorflow 1.14
- LightGBM 2.2.3
- pandas 0.25.1
- numpy 1.16.4
- fbprophet 0.5 
- statsmodels 0.10.0
- plotnine 0.6.0

### 전처리 설명

- 직전의 사용량을 결측치의 갯수로 평균내어서 대체 
- 요일 + 시간별 평균값을 대입하여 대체 

### 모델 설명

- LightGBM 
- LSTM
- Prophet
- Arima