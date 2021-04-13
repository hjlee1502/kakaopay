# 데이터 분석가(Data Analyst) 사전과제
## 목차
* [분석환경](#분석환경)
* [데이터 업로드](#데이터업로드)
* [1번문제](#문제_1)
* [2번문제](#문제_2)
* [3번문제](#문제_3)
* [4번문제](#문제_4)
---

## 분석환경
* Tool
  + Python3.5.1
  + PostgreSQL
---

## 데이터업로드
<pre><code>
-- TRANSACTION 데이터 업로드
CREATE TABLE IF NOT EXISTS TB_TRANSACTION
(
	USER_ID		VARCHAR(10),
	GENDER 		VARCHAR(1),   
	AGE 		INT,
	SVC_TYPE 	VARCHAR(1),  
	DATE 		TIMESTAMP,
	TID 		INT,
	AMOUNT 		INT
)
 sortkey (SVC_TYPE)
;
-- USER_USAGE 데이터 업로드
CREATE TABLE IF NOT EXISTS TB_USER_USAGE
(
	USER_ID        VARCHAR(10),
	USER_TYPE      VARCHAR(2),  
	AMOUNT_TYPE_1  INT,  
	CNT_TYPE_1     INT, 
	AMOUNT_TYPE_2  INT,
	CNT_TYPE_2     INT
)
 sortkey (USER_TYPE)
;
</code></pre>
---

## 문제_1
#### 주어진 두 데이터를 활용하여 탐색 후 1개 혹은 2개 가설을 수립하고 검정해주세요. 또한, 해당 가설을 수립한 이유와 탐색 과정을 기술해주세요.
가설: 두 서비스를 이용하는 

## 문제_2
#### 두 데이터에서의 user_type 그룹으로 간단한 리포트를 작성해주세요


## 문제_3
#### user_usage 데이터에서 금액 혹은 건수의 구간대별 통계를 구하는 쿼리를 2개 작성해 주세요. 각 구간대를 나눈 기준과, 어떤 통계값을 왜 사용했는지 결정한 과정에 대해 설명해 주세요.
<pre><code>
/* 유형1 금액,건수의 백분위수 */
SELECT percentile_disc(0.0) within group (order by AMOUNT_TYPE_1) over() as amt_p0
     , percentile_disc(0.1) within group (order by AMOUNT_TYPE_1) over() as amt_p10 
     , percentile_disc(0.2) within group (order by AMOUNT_TYPE_1) over() as amt_p20 
     , percentile_disc(0.3) within group (order by AMOUNT_TYPE_1) over() as amt_p30 
     , percentile_disc(0.4) within group (order by AMOUNT_TYPE_1) over() as amt_p40 
     , percentile_disc(0.5) within group (order by AMOUNT_TYPE_1) over() as amt_p50 
     , percentile_disc(0.6) within group (order by AMOUNT_TYPE_1) over() as amt_p60 
     , percentile_disc(0.7) within group (order by AMOUNT_TYPE_1) over() as amt_p70 
     , percentile_disc(0.8) within group (order by AMOUNT_TYPE_1) over() as amt_p80 
     , percentile_disc(0.9) within group (order by AMOUNT_TYPE_1) over() as amt_p90 
     , percentile_disc(1.0) within group (order by AMOUNT_TYPE_1) over() as amt_p100 
     ----
     , percentile_disc(0.0) within group (order by CNT_TYPE_1) over() as cnt_p0
     , percentile_disc(0.1) within group (order by CNT_TYPE_1) over() as cnt_p10 
     , percentile_disc(0.2) within group (order by CNT_TYPE_1) over() as cnt_p20 
     , percentile_disc(0.3) within group (order by CNT_TYPE_1) over() as cnt_p30 
     , percentile_disc(0.4) within group (order by CNT_TYPE_1) over() as cnt_p40 
     , percentile_disc(0.5) within group (order by CNT_TYPE_1) over() as cnt_p50 
     , percentile_disc(0.6) within group (order by CNT_TYPE_1) over() as cnt_p60 
     , percentile_disc(0.7) within group (order by CNT_TYPE_1) over() as cnt_p70 
     , percentile_disc(0.8) within group (order by CNT_TYPE_1) over() as cnt_p80 
     , percentile_disc(0.9) within group (order by CNT_TYPE_1) over() as cnt_p90 
     , percentile_disc(1.0) within group (order by CNT_TYPE_1) over() as cnt_p100 
from ADH.TB_USER_USAGE
limit 1;
</code></pre>

1. 금액과 건수의 백분위수를 확인하여 값의 분포를 확인한다. 
* 유형1 금액의 백분위수

amt_p0 | amt_p10 | amt_p20 | amt_p30 | amt_p40 | amt_p50 | amt_p60 | amt_p70 | amt_p80 | amt_p90 | amt_p100
---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | 
-26500 | 1000 | 5000 | 24700 | 46900 | 74520 | 118170 | 183000 | 289000 | 509700 | 21751960

* 유형1 건수의 백분위수

cnt_p0 | cnt_p10 | cnt_p20 | cnt_p30 | cnt_p40 | cnt_p50 | cnt_p60 | cnt_p70 | cnt_p80 | cnt_p90 | cnt_p100
---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | 
0 | 1 | 2 | 3 | 4 | 5 | 7 | 9 | 13 | 20 | 147

---

2-1. [금액 기준] 20, 40, 60, 80분위수 범위에 해당하는 값으로 구간을 설정한다. 단, 값이 5단위로 떨어지게 한다. 여기서는 5천원, 5만원, 15만원, 30만원으로 설정했다. 그리고 금액 구간에 해당하는 평균 건수도 함께 구해주었다. 

<pre><code>
SELECT grp
     , count(distinct user_id) as dnt
     , round(avg(cast(cnt_type_1 as float)),1) as avg_cnt
  from (
	SELECT *
	     , case when AMOUNT_TYPE_1 < 5000 	 then '1_5천원 미만'
		    when AMOUNT_TYPE_1 < 50000 	 then '2_5만원 미만'
		    when AMOUNT_TYPE_1 < 150000  then '3_15만원 미만'
		    when AMOUNT_TYPE_1 < 300000  then '4_30만원 미만'
		    when AMOUNT_TYPE_1 >= 300000 then '5_30만원 이상' end as grp
          from ADH.TB_USER_USAGE
	)
group by 1
order by 1
;
</code></pre>

* 금액 기준 구간별 통계

grp | 유저수 | 평균 건수
---- | ---- | ---- |
1_5천원 미만 | 2953	| 1.0
2_5만원 미만 | 3222	| 3.8
3_15만원 미만 | 3639 | 6.6
4_30만원 미만 | 2306 | 10.7
5_30만원 이상 | 2880 | 21.8

2-2. [건수 기준] 건수는 미이용과 1건 이용이 의미가 있기 때문에 20, 40, 60, 80분위수 범위에 해당하지는 않지만 구간에 포함시킨다. 따라서 미이용, 1건, 2 ~ 4건, 5 ~ 9건, 10건이상으로 설정했다. 마찬가지로 건수 구간에 해당하는 평균 이용금액도 함께 구해주었다.

<pre><code>
SELECT grp
     , count(distinct user_id) as dnt
     , avg(AMOUNT_TYPE_1) as avg_amt
  from (
	SELECT *
	     , case when CNT_TYPE_1 = 0  then '1_미이용'
		    when CNT_TYPE_1 = 1  then '2_1건'
		    when CNT_TYPE_1 < 5  then '3_2~4건'
		    when CNT_TYPE_1 < 10 then '4_5~9건'
		    when CNT_TYPE_1 >= 10 then '5_10건 이상' end as grp
          from ADH.TB_USER_USAGE
	)
group by 1
order by 1
;
</code></pre>

* 건수 기준 구간별 통계

구간 | 유저수 | 평균 금액
---- | ---- | ---- |
1_미이용 | 1260 | 0
2_1건 | 1690 | 15965
3_2~4건 | 3973 | 60647
4_5~9건 | 3620 | 151511
5_10건 이상 | 4457 | 484656
===

## 문제_4
#### transaction 데이터에서 동일 유저가 발생시킨 tx에 대해, 해당 tx의 svc_type과 직전 tx의 svc_type별로 평균 경과 기간과, 건수를 집계하는 쿼리를 작성해 주세요. (즉, 어떤 유저가 서비스를 이용할 때, 직전에 사용한 서비스가 무엇이며, 얼마의 주기로 사용하는지를 집계해 주시면 됩니다)
<pre><code>
select svc_type			as svc_type_before
     , svc_type_after
     , round(avg(datediff),1) 	as avg_date_diff
     , count(tid) 	 	as tx_cnt
  from (select *
	     , cast(datediff('day',date,date_after) as float) as datediff
	  from (select *
		     , lead(svc_type) over (partition by user_id order by tid) as svc_type_after
		     , lead(date) over (partition by user_id order by tid) as date_after
		  from ADH.TB_TRANSACTION
		)
	 where svc_type_after is not null
	)
group by 1,2
order by 1,2
;
</code></pre>

* 결과

svc_type_before | svc_type_after | avg_date_diff | tx_cnt
---- | ---- | ---- | ---- |
A | A | 5.3 | 47041
A | B | 7.7 | 6395
A | C | 4.6 | 1028
A | D | 4.8 | 142
A | E | 1.9 | 21444
B | A | 7.4 | 6880
B | B | 4.9 | 21835
B | C | 4.5 | 325
B | D | 4.7 | 85
B | E | 7.3 | 1233
C | A | 4.5 | 1169
C | B | 4.1 | 343
C | C | 2.3 | 1649
C | D | 18.8 | 5
C | E | 4.6 | 290
D | A | 8.0 | 146
D | B | 3.2 | 98
D | C | 9.8 | 4
D | D | 4.7 | 252
D | E | 7.6 | 28
E | A | 3.0 | 21038
E | B | 7.1 | 1672
E | C | 4.9 | 401
E | D | 7.8 | 34
E | E | 5.9 | 4203
