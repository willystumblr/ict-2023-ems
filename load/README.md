## 각 파일에 대한 설명

```
├── load
│   ├── log_optuna.txt : 하이퍼파라미터 최적화 로그
│   ├── main.ipynb : 모델 구현 및 학습, 평가 코드
│   ├── pickleviewer.ipynb : pickle 파일 내용 확인을 위한 코드
│   ├── predict.ipynb : 모델을 이용한 예측 Application 코드
│   ├── best_results : 모델 저장
│   │   ├── 20230807_165319.pkl
│   │   └── model_20230807_165319.pt
│   └── data : 데이터 전처리 코드 및 전처리한 결과
│       ├── merged_data_KW.xlsx
│       ├── merged_data_MWH.xlsx
│       ├── preprocessing.ipynb : 데이터셋 전처리 코드
│       ├── test_for_0901.xlsx
│       ├── test_for_0901_withseq.xlsx
│       └── dataset_example
│           └── 2022-08-31.xlsx
```

**모델 학습은 jupyter notebook을 사용하였다. 아래는 전반적인 코드 실행 과정에 대한 설명으로, 보다 자세한 사항은 각 코드의 주석을 참고하기를 바란다.**

## 코드 실행 과정

### 데이터 전처리

1. 대회에서 제공한 데이터 기간(2022.07.01~2022.08.31)에 맞는 날씨 정보(날짜, 온도, 습도)를 기상청 홈페이지로부터 다운로드한다. 엑셀파일로 다운로드하고, 해당 데이터는 [https://data.kma.go.kr/data/grnd/selectAsosRltmList.do?pgmNo=36](https://data.kma.go.kr/data/grnd/selectAsosRltmList.do?pgmNo=36)로부터 얻을 수 있다.
2. `data/preprocessing.ipynb` 파일을 연다.
3. 2번째 셀까지 실행하되 `folder` 변수와 `save` 변수에 원하는 경로를 설정한다.
4. 해당 경로의 파일을 열어 한 날짜에 대한 유효전력 데이터가 잘 추출되었는지 확인한다.
5. 다음 셀을 실행하여 22년 7월 1일부터 8월 31일까지 각각의 날짜에 대한 유효전력 데이터만을 추출한다. 이 결과 62개의 엑셀 파일이 생성된다.
6. 마지막 셀까지 실행하되 `power_data_folder`, `weather_data_file`은 필요한 데이터들이 존재하는 경로로 알맞게 설정해주어야 하며, `save_file`에는 원하는 경로를 설정한다.

### 모델 학습 과정

1. `main.ipynb` 파일을 연다.
2. 3번째 셀의 Hyperparameter 값을 원하는대로 설정한다. 각 파라미터에 대한 설명은 주석을 참고한다. thread를 사용할 경우 `num_workers=8`로 설정되어 있다. 학습시킨 모델을 더 학습시키고 싶다면, `pretrained_model_path`에 해당 모델의 경로를 지정하면 된다.
3. 모델 학습 결과는 `results_folder = "/home/kimyirum/EMS/ict-2023-ems/load/results/"`로 지정되어 있기 때문에 해당 경로는 원하는 위치로 설정해준다.
4. 전체 셀을 실행하여 모델을 학습시킨다. 평가 결과는 MAE, MSE, RMSE로 측정하였다.
5. 마지막 셀은 하이퍼파라미터 최적화하는 코드로, optuna 라이브러리를 활용하였다. 해당 레포지토리에 업로드한 모델은 이미 최적화 과정을 거쳐 얻은 하이퍼 파라미터이지만, 실험을 더 진행하고 싶다면 `do_optuna = True`로 설정하고 `lr, hidden_dim, n_layers, batch_size` 변수에 원하는 boundary를 설정해주면 된다.

### 모델의 예측 Application

1. `predict.ipynb` 파일을 연다.
2. 2번째 셀에서 학습을 진행한 모델의 경로를 설정해준다. `path` 변수에는 모델이 저장된 폴더를, `name` 변수에는 원하는 pre-trained 모델의 이름을 설정한다.
