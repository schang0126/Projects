/* Create Path to connect the files */
Libname disk '/home/kevin_7140/my_content/Data';

proc sql;
create table Txn_Data as
select * from disk.txn_data_v9;
quit;

/*Use to distinct to overiew data*/
proc sql;
select distinct product, Txn_type from Txn_Data;
select distinct decision from Txn_Data;
select distinct fraud from Txn_Data;
select distinct cty from Txn_Data;
select distinct MCC from Txn_Data;
quit;


/* use case when to rename MCC*/
proc sql;
create table data_list_mcc as
select *, case
when MCC = 7055 then 'Grocery'
when MCC = 6011 then 'Transportation'
when MCC = 2314 then 'Financial'
when MCC = 2600 then 'Tax Service'
when MCC = 3300 then 'Hospitals'
when MCC = 4408 then 'Schools'
when MCC = 6237 then 'Automobile'
when MCC = 6845 then 'Travel'
when MCC = 9202 then 'Household Appliance Stores'
else 'None' end as MCC_Desc
from Txn_Data;
quit;


proc sql;
create table clean_data as
select a.*, case
when cty = 'North Yrko' then 'North York'
when substr(cty,1,10) = 'North York' then 'North York'
else cty end as City_New
from data_list_mcc as a ;
quit;

proc sql;
select distinct City_New from clean_data;
quit;


/* Calcualte fraud count and dollar grouped by different City and MCC */
proc sql;
create table summary as
select City_New as City,MCC_desc, count(*) as Fraud_count format=6.0, sum(amount) as Fraud_amount format=dollar9.
from clean_data
WHERE FRAUD = 'Yes' and product = 'POS'
group by City_New, MCC_desc
having count(*)>10

order by calculated fraud_count desc;
;
quit;




/* POS Rule Analysis*/
/* Extract Grocery data */
proc sql;
create table Grocery as
select * from clean_data
where MCC_desc = 'Grocery';
quit;


/* Self_join to calculate accmulative amount for selected amount */
proc sql;
create table t1 as
/* select a.txn_id, b.txn_id as txn_id_b, b.amount */
select a.*, b.txn_id as txn_id_b, b.amount
from Grocery a, Grocery b
where a.card_num = b.card_num
and a.timestamp >= b.timestamp
and a.timestamp - b.timestamp <24*5*3600;
quit;

proc sql;
create table t2 as
select a.txn_id, count(*) as count_deple_pos, sum(a.amount) as total_deple_pos
from t1 a
group by a.txn_id;
quit;

proc sql;
create table analysis1 as
select a.*, b.count_deple_pos, b.total_deple_pos
from Grocery a left join t2 b
on a.txn_id=b.txn_id;
quit;


/* Generate Rule with training dataset */
Proc sql;
create table alert as
select * from analysis1
where count_deple_pos>=8 and total_deple_pos>=800 and
decision = 'Approve' and 
datepart(timestamp) >= input('20200430',yymmdd8.) and 
datepart(timestamp) <= input('20200504',yymmdd8.);
quit;

/* Rule performance */
proc sql;
create table perform as
select count(*) as alert,
count(distinct datepart(timestamp)) as days,
sum(case when fraud = 'No' then 1 else 0 end) as legit,
sum(case when fraud = 'Yes' then 1 else 0 end) as fraud,
sum(case when fraud = 'Yes' then amount else 0 end) as benefit,
calculated alert/calculated fraud as hit_rate format=5.2
from alert;
quit;


/* Validation */
Proc sql;
create table alert_val as
select * from analysis1
where count_deple_pos>=8 and total_deple_pos>=800 and
decision = 'Approve' and 
datepart(timestamp) > input('20200504',yymmdd8.);
quit;

proc sql;
create table validation as
select count(*) as alert,
count(distinct datepart(timestamp)) as days,
sum(case when fraud = 'No' then 1 else 0 end) as legit,
sum(case when fraud = 'Yes' then 1 else 0 end) as fraud,
sum(case when fraud = 'Yes' then amount else 0 end) as benefit,
calculated alert/calculated fraud as hit_rate format=5.2
from alert_val;
quit;


