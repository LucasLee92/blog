---
title: 学习oracle存储过程
date: 2018-07-09 14:55:51
tags: 
    - oracle储存过程
---

## 存储过程
存储在数据库中，供所有用户程序调用的子程序

## 创建存储过程
语法：
```sql
Create [or replace] procedure procedure_name  
    [   
        (parameter[{in|in out}]) data_type，  
        (parameter[{in|in out}]) data_type，  
        ……  
    ]  
{is|as}  
    Decoration section  
Begin  
    Executable section;  
Exception  
    Exception handlers;  
End;  
```

* `Procedure_name`：存储过程的名称 
* `Parameter`：参数 
* `in`：向存储过程传递参数 
* `out`：从存储过程返回参数 
* `in out`：即可传递参数，也可返回参数
* `Data_type`：参数的类型 不能够指明长度 
* As|is后声明的变量主要过程体,且不能加declare语句

## 创建案例解析
建立一张测试表

```sql
-- 参数类型不能带长度
create or replace procedure acc_proc_order_jnl (v_ltime in varchar2,
                                                v_rtime in varchar2) as
    v_count number;
    v_busi_serial varchar2(20);
    v_work_date char(8);
    -- 创建游标
    cursor cursor_busi_serial is select busi_serial, work_date from acc_busi_acq_order_jnl where work_date >= v_ltime and work_date < v_rtime;
begin
    v_count := 1;
    -- for in 对游标的使用会自动开启和关闭游标
    for v_pos in cursor_busi_serial loop
        v_busi_serial := v_pos.busi_serial;
        v_work_date := v_pos.work_date;
        -- 类似于 java 中的 System.out.println
        dbms_output.put_line(v_work_date);
        update acc_busi_acq_order_jnl jnl set jnl.order_amt = jnl.acq_tran_amt, jnl.returned_order_amt = jnl.returned_trans_amt where jnl.busi_serial = v_busi_serial;
        v_count := v_count + 1;
        if mod(v_count, 100) = 0 then
            commit;
            dbms_output.put_line('commit');
        end if;
    end loop;
    commit;
end acc_proc_order_jnl;

-- call调用存储过程必须带()和exec不同
call acc_proc_order_jnl('20180501', '20180701');
```








