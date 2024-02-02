> 注：**Staging Count**的值与**Migrated System Count**都一样，所以没有单独列出

### 1-Policy count identified for migration 

> 从我们系统查询Total System Count，从Watchmen数据库查询Migrated Count
>
> 测试时可以添加时间范围来统计数据
>
> * 1.在Core系统这边选定一个时间段的数据同步到watchmen，如十月的数据，那么在Core查询时添加条件
>   * created_at between '2023-10-01 00:00:00' and '2023-10-31 23:59:59'
> * 2.在Watchmen查询数据时添加条件：
>   * createdat between '2023-10-01 00:00:00' and '2023-10-31 23:59:59'
>     		and update_time_ >= '2023-12-05 00:00:00'
>   * 说明：用Core数据createat和Watchmen数据更新时间update_time_ 两个条件来框选数据

~~~sql
# BIX
select
	quote_status Status,
	count(*) 'Total System Count'
from
	trans_index
where
	biz_type in ("NB", "RN")
group by
	quote_status
order by
	quote_status;
~~~

~~~sql
# Watchman
select
	policystatus Status,
	count(*) 'Migrated Count'
from
	topic_policy
group by
	policystatus
order by
	policystatus ;
~~~

### 2-Policy Status wise counts

> DM相关表也可以添加时间条件来框选数据
>
> * extractiondate >= '2023-12-05 00:00:00' 

~~~sql
select
	policystatuscd Status,
	policystatusreason 'Lapsed/Terminated Reason' ,
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0  'Difference'
from
	topic_dmpolicyheader
group by
	policystatuscd ,
	policystatusreason ;
~~~

### 3-Agent wise Policy Status wise

~~~sql
select
	agentid 'Agent Code',
	policystatuscd 'Policy Status' ,
	count(*) 'Count'
from
	topic_dmpolicyheader
group by
	agentid ,
	policystatuscd
order by
	agentid ;
~~~

### 4-Policy Issue Year wise Policy Counts

~~~sql
select
	year(issuedt) 'Year',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyheader
group by
	year(issuedt)
order by
	year(issuedt) desc;
~~~

### 5-Policy Submission Year wise Policy Counts

~~~sql
select
	year(logindt) 'Year',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyheader
group by
	year(logindt)
order by
	year(logindt) desc;
~~~

### 6-Policy Commencement Year wise Policy Counts

~~~sql
select
	year(commencementdt) 'Year',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyheader
group by
	year(commencementdt)
order by
	year(commencementdt) desc;
~~~

### 7-Policy Effective End Date-Year wise Policy Counts

~~~sql
select
	year(policyeffectiveenddt) 'Year',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyheader
group by
	year(policyeffectiveenddt)
order by
	year(policyeffectiveenddt) desc;
~~~

### 8-Source Of Submission wise Counts

> 可能存在batch_admin创建数据的情况，但平台用户没有对应映射逻辑，所以sourceofsubmission为空，看是否需要补充这个映射

~~~sql
select
	sourceofsubmission 'Source Of Submission',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyheader
group by
	sourceofsubmission ;
~~~

### 9-Endorsement  Commission Flag  wise Policy Counts

~~~sql
select
	(case
		when endorsementcommissionflag = 'Y' then 'YES'
		else 'NO'
	end) 'Endorsement Commission Flag',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyheader
group by
	endorsementcommissionflag;
~~~

### 10-Commission Rate Change wise Policy Counts

~~~sql
select
	commissionratechangeflag 'Commission Rate Change Flag',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyproductdetails
group by
	commissionratechangeflag ;
~~~

### 11-Stream Status wise Policy Product count

> streamstatus目前只处理了Affnity的policy，其余的情况都为null

~~~sql
select
	streamstatus 'Stream Status',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyproductdetails
group by
	streamstatus ;
~~~

### 12-Renewal Flag wise Counts

~~~sql
select
	renewalfl 'Renewal Flag',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyheader
group by
	renewalfl ;
~~~

### 13-Product wise Count & Sum of Premium fields

~~~sql
select
	productcd 'Product Code',
	sum(modalpremiumamt) 'Adjusted Net Premium',
	sum(app) APP,
	sum(standardpremium) 'Standard Premium',
	sum(extrapremiumamt) 'Extra Premium Amt'
from
	topic_dmpolicyproductdetails
group by
	productcd ;	
~~~

### 14-Coverage Code wise  Counts

~~~sql
select
	coveragecode 'Coverage Code',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyproductdetails
group by
	coveragecode ;
~~~

### 15-Policy Product Submission Year wise Counts

~~~sql
select
	year(productsubmissiondate) "Year",
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyproductdetails
group by
	year(productsubmissiondate)
order by
	year(productsubmissiondate) desc;
~~~

### 16-Policy Product Issue Date-Year wise Counts

~~~sql
select
	year(productissuedt) "Year",
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyproductdetails
group by
	year(productissuedt)
order by
	year(productissuedt) desc;
~~~

### 17-Policy Product Commencement Date- Year wise Counts

~~~sql
select
	year(productcommencementdt) "Year",
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyproductdetails
group by
	year(productcommencementdt)
order by
	year(productcommencementdt) desc;
~~~

### 18-Client Type Counts

~~~sql
select
	clienttype 'Client Type',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyclientdetails
group by
	clienttype ;
~~~

### 19-Appropriation Type wise Premium

~~~sql
select
	appropriationtypecd 'Appropriation Type',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpremium
group by
	appropriationtypecd ;
~~~

### 20-Alteration Type wise Premium Counts

~~~sql
select
	alterationtype 'Alteration Type',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpremium
group by
	alterationtype ;
~~~

### 21-Commission Type wise Counts

~~~sql
select
	commissiontype 'Commission Type',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmcommission
group by
	commissiontype ;
~~~

### 22-Appropriation Type wise Commission Counts

~~~sql
select
	appropriationtypecd 'Appropriation Type',
	commissiontype 'Commission Type',
	(case
		when fundtype = '05' then 'SIF'
		else 'OIF'
	end) Fund,
	year(commissiondate) 'Year',
	sum(commissionamt) 'Commission Amount',
	0 'Gst On Comm Amount',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmcommission
group by 
	appropriationtypecd ,
	commissiontype ,
	fundtype ,
	year(commissiondate);
~~~

### 23-Agent Wise Commission Type wise count and Sum of Commission amount

~~~sql
select
	c.agentcode 'Agent Code',
	c.agentcategorycode 'Designation Code(GI Category Code)',
	c.commissiontype 'Commission Type',
	(case
		when c.fundtype = '05' then 'SIF'
		else 'OIF'
	end) Fund,
	f.product_code 'Product Code',
	c.commissionstatus 'Commission Status',
	count(distinct premiumreferencenum) 'DN/CN Count' ,
	year(c.commissiondate) 'Comm Gen Dt Year',
	'NO' APPR,
	sum(c.commissionamt) 'Commission Amount',
	0 'Gst On Comm Amount',
	0 'Total Commission (Comm + GST on Comm)'
from
	topic_dmcommission c
left join topic_fee f on
	c.primarykey = f.fee_id
group by 
	c.agentcode ,
	c.agentcategorycode ,
	c.commissiontype ,
	c.fundtype ,
	f.product_code ,
	c.commissionstatus ,
	year(c.commissiondate);
~~~

​	
