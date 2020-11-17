# Kakaopay - Data Scientist 사전과제

## 과제 2번
```
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
```

