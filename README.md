# GIST ict-2023 EMS-Track: Team MYsolar

GIST 창의융합경진대회 2023 EMS-Track, MYsolar 팀 GitHub Repo

## Set-ups

아나콘다(Anaconda) 설치 링크: [Anaconda 공식문서](https://conda.io/projects/conda/en/latest/user-guide/install/download.html)

> conda version: `23.1.0`

`git clone` 으로 파일 다운로드후, 아나콘다 설치 및 가상환경 생성, 패키치 다운로드

```bash
git clone https://github.com/willystumblr/ict-2023-ems.git
conda create -n ENVIRONMENT_NAME python==3.9
pip install -r requirements.txt
```

## Files & Program execution

### Directories and Files

```plaintext
.
├── README.md
├── dataloader.py
├── model.py
├── predict.py     ### FOR EXECUTION ###
├── requirements.txt
├── schedule.py    ### FOR EXECUTION ###
├── load
│   ├── log_optuna.txt
│   ├── main.ipynb
│   ├── pickleviewer.ipynb
│   ├── predict.ipynb
│   ├── best_results
│   │   ├── 20230807_165319.pkl
│   │   └── model_20230807_165319.pt
│   └── data
│       ├── merged_data_KW.xlsx
│       ├── merged_data_MWH.xlsx
│       ├── preprocessing.ipynb
│       ├── test_for_0901.xlsx
│       ├── test_for_0901_withseq.xlsx
│       └── dataset_example
│           └── 2022-08-31.xlsx
└── pv
    ├── \010ict_2023_model_predict.ipynb
    ├── eda.ipynb
    ├── fix.py
    ├── preprocess.py
    ├── checkpoints
    │   ├── LG도서관-checkpoint-2023-08-10 22_10.pt
    │   ├── 대학C동-checkpoint-2023-08-10 22_35.pt
    │   ├── ...
    │   ├── 중앙도서관-checkpoint-2023-08-10 23_26.pt
    │   └── 신재생에너지동-checkpoint-2023-08-10 23_18.pt
    ├── eval_sequence
    │   ├── LG도서관-eval.csv
    │   ├── 대학C동-eval.csv
    │   ├── ...
    │   ├── 삼성환경동-eval.csv
    │   └── 신재생에너지동-eval.csv
    └── results
        ├── LG도서관-2023-08-11-hp.pkl
        ├── 대학C동-2023-08-11-hp.pkl
        ├── ...
        ├── 중앙도서관-2023-08-11-hp.pkl
        └── 신재생에너지동-2023-08-11-hp.pkl
```

### 예측 알고리즘

LSTM 모델을 사용하여, 다음날 24시간의 시간당 발전량/전력사용량 예측을 위해 지난 일주일간의 시간대별 데이터로 학습한다. 손실을 줄이기 위해, 비교적 건물 수가 14개로 적었던 태양광 발전량 데이터는 건물별로 모델을 생성하여 학습하였으며, 건물 수가 많았던(56) 부하 데이터의 경우 컴퓨팅 자원의 한계와 실행시간의 비효율성을 고려하여 건물을 하나의 feature로 설정한다.

- sequence length: 24*7 (지난 일주일의 시간당 데이터)
- label length: 24 (다음날 시간당 예측값)
- hyper-parameters:
  - 태양광 발전량 예측모델:
    ```python
    config ={
    	"num_layers":9,
    	"hidden_size":32,
    	"batch_size":256,
    	"num_epochs":100
    }
    ```
  - 부하 예측 모델
    ```python
    config ={
    	"num_layers": 5,
    	"hidden_size": 128,
    	"batch_size": 128,
    	"num_epochs": 700
    }
    ```

학습 및 검증 후, `.pt` 확장자로 모델의 state를 체크포인트에 저장하였다.

저장된 체크포인트와 미리 저장한 검증 데이터셋을 활용, `predict.py`를 실행하여 **2022.08.31**의 시간당 전력사용량 및 태양광 발전량을 예측한다.

```bash
python predict.py --mode pv     ## prediction for PV generation
python predict.py --mode load   ## prediction for load
```

### 스케줄링 알고리즘

파이썬 오픈소스 라이브러리 중,  `pulp`를 활용하여 선형계획법을 구성한다.

이를 정의하기 위하여 사용한 변수 및 함수는 아래와 같다.

1. 의사결정 변수(Decision Variables)

- `energy_bought`: ESS grid로부터 시간당 공급받는(import) 에너지
- `energy_sold`: ESS grid에 시간당 공급하는 에너지
- `energy_charged`: ESS grid에 시간당 충전되는 에너지
- `energy_discharged`: ESS grid에서 시간당 방전되는 에너지
- `battery_level`: 배터리의 상태, 시간대별 시작과 끝을 고려하여 총 25개의 인덱스를 가지도록 변수를 생성함 (0시 0분, 0시 59분 등)

2. 목적 함수(Objective Function)

- 전기요금을 최소화함 (`energy_bought`로 인한 요금 최소화)
- ESS에 저장하는 에너지로부터 이익을 최대화함

3. 제약조건(Constraints)

- 배터리 충전/방전 및 배터리 효율을 고려한 상태(level) 제약
- 에너지 균형

매개변수(Parameter)는 배터리 충·방전 효율, 전기요금, 배터리 최대용량, 그리고 예측모델이 산출한 부하 및 태양광 발전량 예측값 등이 있다.

아래 명령어로 `schedule.py` 파일을 실행한다.

```python
python schedule.py --mode eval
```

## Results

정확한 결과를 위해 다음 두 환경에서 실험하였다.

- conda 가상환경 (Apple M1 core, CPU), python 3.9.17 버전
- Google Colaboratory 환경, V100 GPU 1장, python 3.10.12 버전

CPU 환경에서 `predict.py` 로 태양광 발전량/부하를 예측한 결과는 다음과 같다. 여기서 `Total Prediction Error`의 단위는 kWh이다.

```plaintext
(ict-2023) minsk@Maximus 2023ict % python predict.py --mode pv  
08/13/2023 01:03:18 - INFO - __main__ -   Predicting PV for 0831...
08/13/2023 01:03:18 - INFO - __main__ -   MAE: 8.0234, MSE: 143.0949, RMSE: 11.9622 (no normalization)
08/13/2023 01:03:18 - INFO - __main__ -   Total Prediction Error: 279.2447814941406
08/13/2023 01:03:18 - INFO - __main__ -   Prediction completed. filepath: ./pv_predict_for_0831.csv
(ict-2023) minsk@Maximus 2023ict % python predict.py --mode load
08/13/2023 01:03:23 - INFO - __main__ -   Predicting LOAD for 0831...
torch.Size([1, 1344]) torch.Size([1, 1344])
08/13/2023 01:03:24 - INFO - __main__ -   MAE: 16.4693, MSE: 1237.4397, RMSE: 35.1773 (no normalization)
08/13/2023 01:03:24 - INFO - __main__ -   Total Prediction Error: 428.0094988500473
08/13/2023 01:03:24 - INFO - __main__ -   Prediction completed. filepath: ./load_predict_for_0831.xlsx
```

CPU 환경에서 `schedule.py`로 2022년 8월 31일 전기요금을 최적화한 결과는 아래와 같으며, **₩43,234,599**로 계산된다.

```plaintext
Welcome to the CBC MILP Solver 
Version: 2.10.3 
Build Date: Dec 15 2019 

command line - /Users/minsk/opt/anaconda3/envs/ict-2023/lib/python3.9/site-packages/pulp/solverdir/cbc/osx/64/cbc /var/folders/5m/_dn3q76s5zb9tr9xkr_8vtnh0000gn/T/c72eac47906b4b0ab130b4233eeb77a6-pulp.mps timeMode elapsed branch printingOptions all solution /var/folders/5m/_dn3q76s5zb9tr9xkr_8vtnh0000gn/T/c72eac47906b4b0ab130b4233eeb77a6-pulp.sol (default strategy 1)
At line 2 NAME          MODEL
At line 3 ROWS
At line 102 COLUMNS
At line 437 RHS
At line 535 BOUNDS
At line 561 ENDATA
Problem MODEL has 97 rows, 121 columns and 286 elements
Coin0008I MODEL read with 0 errors
Option for timeMode changed from cpu to elapsed
Presolve 91 (-6) rows, 115 (-6) columns and 273 (-13) elements
Perturbing problem by 0.001% of 151.07957 - largest nonzero change 1.6838469e-05 ( 1.2727507e-05%) - largest zero change 1.6589949e-05
0  Obj 42416730 Primal inf 8094.6687 (24) Dual inf 2511.075 (23)
0  Obj 42416726 Primal inf 8094.6687 (24) Dual inf 3.5307098e+11 (47)
60  Obj 42830738 Primal inf 3025.0365 (9) Dual inf 1.0902913e+11 (37)
80  Obj 43234599
Optimal - objective value 43234599
After Postsolve, objective 43234599, infeasibilities - dual 0 (0), primal 0 (0)
Optimal objective 43234598.6 - 80 iterations time 0.002, Presolve 0.00
Option for printingOptions changed from normal to all
Total time (CPU seconds):       0.00   (Wallclock seconds):       0.00

Status: Optimal
Optimized Cost: ₩43,234,599
```
