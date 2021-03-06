# p2-tab-김경민 

### 온라인 상점 고객 구매 예측 

## 개요 

많은 데이터가 정형으로 이루어져 있고, 정형데이터는 데이터 분석의 기초라고 할 수 있습니다.

그렇기 때문에 정형 데이터 분석을 통해 EDA, 전처리 실습을 진행하고 머신러닝 모델링을 통해서 데이터 분석에 대한 전반적인 역량을 익힐 수 있습니다.

최근 온라인 거래를 이용하는 고객들이 많이 늘어나고 있어 고객들의 log 데이터가 많이 늘어나고 있습니다. 온라인 거래 고객 log 데이터를 이용하여 고객들의 미래 소비를 예측 분석프로젝트를 진행하려 합니다.

고객들의 월별 총 구매 금액을 확인했을 때 연말에 소비가 많이 이루어지고 있는 것으로 확인이 되었습니다. 그리하여 12월을 대상으로 고객들에게 프로모션을 통해 성공적인 마케팅을 하기 위해 모델을 만들려고 합니다.

온라인 거래 log 데이터는 2009년 12월부터 2011년 11월까지의 온라인 상점의 거래 데이터가 주어집니다. 2011년 11월 까지 데이터를 이용하여 2011년 12월의 고객 구매액 300초과 여부를 예측해야 합니다.

## Components

<code>Main.ipynb</code> 

* feature engineering, random seed고정, 결과 도출등 예측에 필요한 모든 함수들을 구현한 파일입니다.

<code>ensemble.ipynb</code>

* 여러개의 모델을 앙상블하여 결과를 도출하는 파일입니다. 앙상블에는 단순 평균 방법을 사용했습니다.

<code>test.txt</code>  

* 대회 진행동안 제출한 모델의 test AUC, train AUC, 최종 AUC, 사용한 피쳐, 모델, 하이퍼 파라미터등을 정리한 파일입니다.

## 데이터 

* order_id : 주문 번호, 데이터에서 같은 주문번호는 동일 주문을 나타냄
* product_id : 상품 번호
* description : 상품 설명
* quantity : 상품 주문 수량
* order_date : 주문 일자
* price : 상품 가격
* costomer_id : 고객 번호
* country : 고객 거주 국가
* total : 총 구매액 
* 총 780502의 row , 9개의 feature

![tab_csv](https://github.com/bcaitech1/p2-tab-kyungminkim-dev/blob/main/image/tab_csv.png)

## 평가 방법 

metric : AUC(Area Under Curve)

![AUC](https://github.com/bcaitech1/p2-tab-kyungminkim-dev/blob/main/image/AUC.png)

## 등수

최종 LB AUC : 0.8572 , 36등/ 99


## 검증 전략 

stratified K fold(fold = 10)을 사용 하여 검증 했습니다. 

## 사용한 모델의 아키텍쳐 및 하이퍼 파라미터

가장 성능이 좋았던 두 모델을 앙상블 해서 최종 점수를 얻었습니다.
	
첫번째 모델은 피쳐로 group cumsum Agg와 Time-Series diff를 가지고 있습니다. nunique 피쳐를 추가해 보았지만 오히려 성능이 떨어져서 사용을 하지 않았습니다.
하이퍼 파라미터는 optuna를 사용해서 최적화를 했습니다. n_trial값을 10을 사용했습니다. n_trial값을 30을 주니 train, test AUC 점수는 10일때 보다 더 높게 나왔지만 public LB 점수는 오히려 낮았습니다. 오버피팅이 발생한것으로 보입니다.
	이번 대회에서는 catboost와 xgboost는 사용하지 않았습니다. 두 모델 다  전부
	lightGBM보다 성능이 낮게 나왔기 때문입니다. 이 모델의 하이퍼 파라미터는 다음과             같습니다.

* tuned_lgb_params = {
    'objective': 'binary', 
    'boosting_type': 'gbdt',
    'metric': 'auc', 
    'feature_fraction': 0.4037902763824288, 
    'bagging_fraction': 0.44651205628348084, 
    'bagging_freq': 4,
    'n_estimators': 10000, 
    'early_stopping_rounds': 100,
    'seed': SEED,
    'verbose': -1,
    'n_jobs': -1, 
    'num_leaves': 43,
    'max_bin': 140,
    'min_data_in_leaf': 36,
    'lambda_l1': 1.6722093110936513,
    'lambda_l2': 2.1262853662727266,
}

첫번째 모델의 Public AUC 점수는 0.8567입니다.

두번째 모델은 첫 번째 모델에 nunique 변수를 추가 한 뒤, 수치형 변수에 quantile_transform을 적용한 모델입니다. nunique변수를 추가해서 q_t를 적용하니 점수가 더 잘나왔습니다. 하이퍼 파라미터 튜닝을 진행 했고 파라미터 값을 다음과 같습니다. 

* {'num_leaves': 159,
 'max_bin': 135,
 'min_data_in_leaf': 28,
 'feature_fraction': 0.9068781382922049,
 'bagging_fraction': 0.5427572227580095,
 'bagging_freq': 4,
 'lambda_l1': 2.531979160239369,
 'lambda_l2': 1.645404534997536e-05}

두번째 모델의 Public AUC점수는 0.8558 입니다.

마지막으로 제출한 모델은 위 두 모델을 단순평균하여 앙상블을 한 모델입니다.

## 아쉬운점 

* 피쳐 엔지니어링 부분에서 창의적인 변수를 만들어 내지 못한것이 아쉽습니다. 데이터를 보는 시각이 부족해 보입니다. 토론 게시판을 보면 계절성이나 주기성을 추가하여 성능향상을 보인 분들이 보였습니다.

* pandas에 대한 숙련도가 부족해서 다양한 시도를 하지 못했습니다. 베이스라인 코드를 읽고 응용하는 부분에서 pandas숙련도의 부족으로 인해 에러가 많이 발생하고 시간을 많이 잡아먹었습니다.

* TabNet을 써보고 싶었지만 시간이 부족해서 시도해보지 못한것이 아쉽습니다. 

* 여러가지 앙상블 방법을 사용해보지 못한것이 아쉽습니다.  


