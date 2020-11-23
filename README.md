# Kakaopay - Data Scientist 사전과제
## 과제 1번
<pre><code>
# =============================================================================
# 문제1 :‘더치페이 요청에 대한 응답률이 높을수록 더치페이 서비스를 더 많이 사용한다.’라는 가설을 통계적으로 검정해주세요.
# 더치페이 요청에 대한 응답률, 유저의 서비스 사용수 상관분석
# =============================================================================

# =============================================================================
# 패키지 로딩 및 옵션 설정
# =============================================================================
import pandas as pd
import numpy as np
import pythena
from pandasql import sqldf
from sklearn.preprocessing import MinMaxScaler
from scipy import stats import pearsonr

# =============================================================================

# =============================================================================
# 데이터 업로드 
# =============================================================================
a_payment_trx = pd.read_csv("./DS_사전과제_v2/a_payment_trx.csv")
dutchpay_claim_detail = pd.read_csv("./DS_사전과제_v2/dutchpay_claim_detail.csv")
dutchpay_claim = pd.read_csv("./DS_사전과제_v2/dutchpay_claim.csv")
users = pd.read_csv("./DS_사전과제_v2/users.csv")
# =============================================================================

# =============================================================================
# 데이터 전처리
# =============================================================================
# 더치페이 상세 + 요청자, 요청시간
dutchpay_fnl = pd.merge(dutchpay_claim_detail,dutchpay_claim, how='inner', on='claim_id')

# status별 받은 요청수 (status='CHECK' 제외)
user_status_cnt=dutchpay_fnl[(dutchpay_fnl['status']!='CHECK')].pivot_table(index=['claim_user_id','claim_id'],columns='status',values='claim_detail_id',aggfunc='nunique').reset_index().fillna(0)
# 요청률 계산 (send/send+claim)
user_status_cnt['rspnd_ratio'] = user_status_cnt['SEND']/(user_status_cnt['SEND']+user_status_cnt['CLAIM'])

# 유저별 평균 응답률
user_avg_rspnd = user_status_cnt.groupby('claim_user_id').agg({'rspnd_ratio': 'mean', 'claim_id': 'count'}).reset_index()
user_avg_rspnd = user_avg_rspnd.rename({'claim_user_id':'user_id','rspnd_ratio':'avg_rspnd_ratio','claim_id':'claim_cnt'}, axis='columns')

# 유저별 더치페이 요청을 받았을 때 돈을 보낸 횟수(status='CHECK' 제외)
user_rspnd_cnt=dutchpay_fnl[(dutchpay_fnl['status']!='CHECK')].pivot_table(index=['recv_user_id'],values='claim_detail_id',aggfunc='nunique').reset_index().fillna(0)
user_rspnd_cnt=user_rspnd_cnt.rename({'recv_user_id':'user_id', 'claim_detail_id':'rspnd_cnt'}, axis='columns')

# 유저별 평균 응답률, 돈을 보낸 횟수
df = pd.merge(user_avg_rspnd, user_rspnd_cnt, how='outer', on='user_id').fillna(0)

# 사용횟수 = 요청횟수+돈을 보낸 횟수
df['use_cnt'] = df['claim_cnt']+df['rspnd_cnt']

# 최종 분석 마트 생성
df_anal = df.loc[:,['use_cnt','avg_rspnd_ratio']]

# 변수 표준화
scaler = MinMaxScaler()
scale_columns=['use_cnt','avg_rspnd_ratio']
df_anal[scale_columns] = scaler.fit_transform(df_anal[scale_columns])

# =============================================================================
# 상관분석
# =============================================================================
pearsonr(df_anal.avg_rspnd_ratio, df_anal.use_cnt)

# 결과
#(0.1873641684586831, 0.0)

분석 결과 : 상관분석 결과 p-value는 0이므로 '응답률과 사용횟수 간의 상관관계가 없다' 라는 귀무가설을 기각한다. 따라서 응답률과 사용횟수간의 선형적인 상관관계가 있다.
상관계수는 약 0.187로 약한 선형성이 있음을 알 수 있다.
</code></pre>
## 과제 2번
<pre><code>
select user_id
from (
    select user_id, sum(amount) as tot_amt
    from (
        select a.transaction_id, a.user_id, a.amount
              ,max(case when b.payment_action_type = 'CANCEL' and substr(b.transacted_at,1,7) <= '2020-02' then 1 else 0 end) as cancel_yn
        from a_payment_trx a
        left join a_payment_trx b on a.transaction_id = b.transaction_id
        where substr(a.transacted_at,1,7) = '2019-12'
        and a.payment_action_type = 'PAYMENT'
        group by a.transaction_id, a.user_id, a.amount
        ) aa
    where cancel_yn = 0
    group by user_id
    ) aaa
where tot_amt >=10000
;
</code></pre>

## 과제 3번
<pre><code>
# =============================================================================
# 패키지 로딩 및 옵션 설정
# =============================================================================
from datetime import datetime
from dateutil.relativedelta import relativedelta
import pandas as pd
import numpy as np


# =============================================================================
# 데이터 업로드 
# =============================================================================
df = pd.read_csv('./OneDrive/housing.csv'
                 ,names=['CRIM','ZN','INDUS','CHAS','NOX','RM','AGE','DIS','RAD','TAX','PTRATIO','B','LSTAT','MEDV'])

df.describe()
# =============================================================================
# 모델 생성 및 검증
# =============================================================================

# test/ train 데이터 분할 
from sklearn.model_selection import train_test_split
x_train, x_test, y_train, y_test = train_test_split(df.drop('MEDV', 1), df['MEDV'])

# 모델별 성능 확인을 위한 함수
# 모델 평가 지표: MSE(예측값과 실제값의 차이에 대한 제곱에 대하여 평균을 낸 값)
from sklearn.metrics import mean_absolute_error, mean_squared_error
import matplotlib.pyplot as plt
import seaborn as sns

my_predictions = {}

colors = ['r', 'c', 'm', 'y', 'k', 'khaki', 'teal', 'orchid', 'sandybrown',
          'greenyellow', 'dodgerblue', 'deepskyblue', 'rosybrown', 'firebrick',
          'deeppink', 'crimson', 'salmon', 'darkred', 'olivedrab', 'olive', 
          'forestgreen', 'royalblue', 'indigo', 'navy', 'mediumpurple', 'chocolate',
          'gold', 'darkorange', 'seagreen', 'turquoise', 'steelblue', 'slategray', 
          'peru', 'midnightblue', 'slateblue', 'dimgray', 'cadetblue', 'tomato'
         ]

def plot_predictions(name_, pred, actual):
    df = pd.DataFrame({'prediction': pred, 'actual': y_test})
    df = df.sort_values(by='actual').reset_index(drop=True)

    plt.figure(figsize=(12, 9))
    plt.scatter(df.index, df['prediction'], marker='x', color='r')
    plt.scatter(df.index, df['actual'], alpha=0.7, marker='o', color='black')
    plt.title(name_, fontsize=15)
    plt.legend(['prediction', 'actual'], fontsize=12)
    plt.show()
    
def mse_eval(name_, pred, actual):
    global predictions
    global colors

    plot_predictions(name_, pred, actual)

    mse = mean_squared_error(pred, actual)
    my_predictions[name_] = mse

    y_value = sorted(my_predictions.items(), key=lambda x: x[1], reverse=True)
    
    df = pd.DataFrame(y_value, columns=['model', 'mse'])
    print(df)
    min_ = df['mse'].min() - 10
    max_ = df['mse'].max() + 10
    
    length = len(df)
    
    plt.figure(figsize=(10, length))
    ax = plt.subplot()
    ax.set_yticks(np.arange(len(df)))
    ax.set_yticklabels(df['model'], fontsize=15)
    bars = ax.barh(np.arange(len(df)), df['mse'])
    
    for i, v in enumerate(df['mse']):
        idx = np.random.choice(len(colors))
        bars[i].set_color(colors[idx])
        ax.text(v + 2, i, str(round(v, 3)), color='k', fontsize=15, fontweight='bold')
        
    plt.title('MSE Error', fontsize=18)
    plt.xlim(min_, max_)
    plt.show()

def remove_model(name_):
    global my_predictions
    try:
        del my_predictions[name_]
    except KeyError:
        return False
    return True   

# 모델1) Linear Regression
from sklearn.linear_model import LinearRegression
model = LinearRegression(n_jobs=-1)
model.fit(x_train, y_train)
pred = model.predict(x_test)
mse_eval('LinearRegression', pred, y_test)

# 모델2) Ridge
from sklearn.linear_model import Ridge

# 값이 커질 수록 큰 규제임
alphas = [100, 10, 1, 0.1, 0.01, 0.001, 0.0001]

for alpha in alphas:
    ridge = Ridge(alpha=alpha, random_state=42)
    ridge.fit(x_train, y_train)
    pred = ridge.predict(x_test)
    mse_eval('Ridge(alpha={})'.format(alpha), pred, y_test)

# coef에 대한 plot
def plot_coef(columns, coef):
    coef_df = pd.DataFrame(list(zip(columns, coef)))
    coef_df.columns=['feature', 'coef']
    coef_df = coef_df.sort_values('coef', ascending=False).reset_index(drop=True)
    
    fig, ax = plt.subplots(figsize=(9, 7))
    ax.barh(np.arange(len(coef_df)), coef_df['coef'])
    idx = np.arange(len(coef_df))
    ax.set_yticks(idx)
    ax.set_yticklabels(coef_df['feature'])
    fig.tight_layout()
    plt.show()
    
plot_coef(x_train.columns, ridge.coef_)

# alpha값에 따른 coef 차이 확인
ridge_100 = Ridge(alpha=100)
ridge_100.fit(x_train, y_train)
ridge_pred_100 = ridge_100.predict(x_test)

ridge_001 = Ridge(alpha=0.001)
ridge_001.fit(x_train, y_train)
ridge_pred_001 = ridge_001.predict(x_test)

plot_coef(x_train.columns, ridge_100.coef_)
plot_coef(x_train.columns, ridge_001.coef_)


# 모델3) ElasticNet

# l1_ratio = 0 (L2 규제만 사용).
# l1_ratio = 1 (L1 규제만 사용).
# 0 < l1_ratio < 1 (L1 and L2 규제의 혼합사용)

from sklearn.linear_model import ElasticNet
ratios = [0.2, 0.5, 0.8]

for ratio in ratios:
    elasticnet = ElasticNet(alpha=0.5, l1_ratio=ratio, random_state=42)
    elasticnet.fit(x_train, y_train)
    pred = elasticnet.predict(x_test)
    mse_eval('ElasticNet(l1_ratio={})'.format(ratio), pred, y_test)


# 모델4) Polynomial Features + ElasticNet / Ridge / LinearRegression
# 다항식의 계수간 상호작용을 통해 새로운 feature를 생성
    
from sklearn.preprocessing import PolynomialFeatures
poly = PolynomialFeatures(degree=2, include_bias=False)
poly_features = poly.fit_transform(x_train)[0]
poly_features
x_train.iloc[0]

from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler
from sklearn.pipeline import make_pipeline

# poly+ElasticNet
poly_pipeline = make_pipeline(
    PolynomialFeatures(degree=2, include_bias=False),
    StandardScaler(),
    ElasticNet(alpha=0.1, l1_ratio=0.2, random_state=42)
)
poly_pred = poly_pipeline.fit(x_train, y_train).predict(x_test)
mse_eval('Poly ElasticNet', poly_pred, y_test)

# poly+Ridge
poly_pipeline = make_pipeline(
    PolynomialFeatures(degree=2, include_bias=False),
    StandardScaler(),
    Ridge(alpha=0.1, random_state=42)
)
poly_pred = poly_pipeline.fit(x_train, y_train).predict(x_test)
mse_eval('Poly Ridge', poly_pred, y_test)

# poly+LinearRegression
poly_pipeline = make_pipeline(
    PolynomialFeatures(degree=2, include_bias=False),
    StandardScaler(),
    LinearRegression(n_jobs=-1)
)
poly_pred = poly_pipeline.fit(x_train, y_train).predict(x_test)
mse_eval('Poly linearRegression', poly_pred, y_test)
</code></pre>

### 예측결과
<img width="400" src="https://user-images.githubusercontent.com/68849635/99972209-983da380-2de1-11eb-8b93-1a2b044e369f.png">

