## 주제 : 아파트 실거래가 예측 모델링 대회

기간 :  2018년 11 월 12 일 ~ 2019년 1 월 31 일

결과 

![](https://github.com/choco9966/Dacon/blob/master/Real%20House%20Price%20Prediction/image/image-20191028122710488.png?raw=true)

### 폴더 구조

```
.
├── code
│   ├── PREPROCESSING
│   │   └── public_juso_crawling.zip
│   ├── FEATURES
│   │   └── 3. Subway, School features.ipynb
│   │   └── 3. Gonggong data features.ipynb
│   ├── MODEL
|   │   └── 4. Modeling_Lightgbm_Quantile.ipynb
|   │   └── 4. Modeling_Lightgbm_Regression_Last_validation.ipynb
|   │   └── 4. Modeling_Lightgbm_Regression.ipynb
|   │   └── 4. Modeling_Xgboost_3fold.ipynb
|   │   └── 4. Modeling_Xgboost_5fold_oversampling.ipynb
|   │   └── 4. Modeling_Xgboost_5fold_not_oversampling.ipynb
│   ├── ENSEMBLE
|   │   └── 5. Ensemble.ipynb
└── Experience
```

### 실행 환경

- python3.6 

### 필요 라이브러리

- LightGBM 2.2.3
- Xgboost 0.83
- pandas 0.25.1
- numpy 1.16.4

### 전처리 설명

- 방의 수에 대한 결측치를 동일한 아파트 크기의 방의 갯수로 대체 
- 주차장, 난방, 현관구조 결측치는 없음으로 대체 
- 공공데이터의 도로명을 위경도로 변환 
- 아파트 가격이 높은 아파트들에 대해 오버샘플링 

### 피쳐 설명

- 편의 시설 및 공공 시설 : 근처의 지하철, 아파트의 갯수. 구청의 존재 등등. 
- 아파트의 가치 : 방의 갯수, 크기, 세대 별 주차장의 수 등등.  
- 과거 아파트의 가격 : 최근 아파트의 가격 
- 재개발 여부 : 재개발 가능성 여부를 평가 

### 모델 설명

- 부산과 서울을 구분해서 모델을 돌림. 
- LightGBM (Quantile - 5fold, Regression - 5fold, Regression - hold out) 
- XGBoost (Regression - 3fold, Regression - 5fold,  Regression - 5fold with oversampling)

