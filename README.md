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
transaction, user_useage 데이터 업로드
<pre><code>
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


## 문제_2
#### 두 데이터에서의 user_type 그룹으로 간단한 리포트를 작성해주세요


## 문제_3
#### user_usage 데이터에서 금액 혹은 건수의 구간대별 통계를 구하는 쿼리를 2개 작성해 주세요. 각 구간대를 나눈 기준과, 어떤 통계값을 왜 사용했는지 결정한 과정에 대해 설명해 주세요.


## 문제_4
#### transaction 데이터에서 동일 유저가 발생시킨 tx에 대해, 해당 tx의 svc_type과 직전 tx의 svc_type별로 평균 경과 기간과, 건수를 집계하는 쿼리를 작성해 주세요. (즉, 어떤 유저가 서비스를 이용할 때, 직전에 사용한 서비스가 무엇이며, 얼마의 주기로 사용하는지를 집계해 주시면 됩니다)
<pre><code>
select svc_type			as svc_type_before
     , svc_type_after
     , round(avg(datediff),1) 	as avg_date_diff
     , count(tid) 	 	as tot_cnt
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



### 예측결과
<img width="400" src="https://user-images.githubusercontent.com/68849635/99972209-983da380-2de1-11eb-8b93-1a2b044e369f.png">

