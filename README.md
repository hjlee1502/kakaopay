# Kakaopay - Data Scientist 사전과제

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



