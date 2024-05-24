### 1-Policy count identified for migration (*not required*)

### 2-Policy Status wise counts

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
		when fundtype = '05' then '05-SIF'
		when fundtype = '06' then '06-OIF'
		else '03-Singapore Life Non Par Fund'
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
		when c.fundtype = '05' then '05-SIF'
		when c.fundtype = '06' then '06-OIF'
		else '03-Singapore Life Non Par Fund'
	end) Fund,
	f.product_code 'Product Code',
	c.commissionstatus 'Commission Status',
	count(c.primarykey) 'DN/CN Count' ,
	year(c.commissiondate) 'Comm Gen Dt Year',
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

### 24-Data Migration total count

~~~sql
select 'dmpolicyheader' as 'Table', count(1) as 'Count' from topic_dmpolicyheader td union all
select 'dmpolicyhierarchy' as 'Table', count(1) as 'Count' from topic_dmpolicyhierarchy td union all
select 'dmpolicyproductdetails' as 'Table', count(1) as 'Count' from topic_dmpolicyproductdetails union all 
select 'dmpolicyclientdetails' as 'Table', count(1) as 'Count' from topic_dmpolicyclientdetails union all 
select 'dmcommission' as 'Table', count(1) as 'Count' from topic_dmcommission union all
select 'dmpremium' as 'Table', count(1) as 'Count' from topic_dmpremium;
~~~

~~~sql
use `bixdb`;

### 1-Policy count identified for migration (*not required*)

### 2-Policy Status wise counts
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
 

### 3-Agent wise Policy Status wise
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


### 4-Policy Issue Year wise Policy Counts
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
 

### 5-Policy Submission Year wise Policy Counts
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
 

### 6-Policy Commencement Year wise Policy Counts
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
 

### 7-Policy Effective End Date-Year wise Policy Counts
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
 

### 8-Source Of Submission wise Counts
select
	sourceofsubmission 'Source Of Submission',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyheader
group by
	sourceofsubmission ;
 

### 9-Endorsement  Commission Flag  wise Policy Counts
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
 

### 10-Commission Rate Change wise Policy Counts
select
	commissionratechangeflag 'Commission Rate Change Flag',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyproductdetails
group by
	commissionratechangeflag ;
 

### 11-Stream Status wise Policy Product count
select
	streamstatus 'Stream Status',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyproductdetails
group by
	streamstatus ;
 

### 12-Renewal Flag wise Counts
select
	renewalfl 'Renewal Flag',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyheader
group by
	renewalfl ;
 

### 13-Product wise Count & Sum of Premium fields
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
 

### 14-Coverage Code wise  Counts
select
	coveragecode 'Coverage Code',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyproductdetails
group by
	coveragecode ;
 

### 15-Policy Product Submission Year wise Counts
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
 

### 16-Policy Product Issue Date-Year wise Counts
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
 

### 17-Policy Product Commencement Date- Year wise Counts
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
 

### 18-Client Type Counts
select
	clienttype 'Client Type',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpolicyclientdetails
group by
	clienttype ;
 

### 19-Appropriation Type wise Premium
select
	appropriationtypecd 'Appropriation Type',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpremium
group by
	appropriationtypecd ;
 

### 20-Alteration Type wise Premium Counts
select
	alterationtype 'Alteration Type',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmpremium
group by
	alterationtype ;
 

### 21-Commission Type wise Counts
select
	commissiontype 'Commission Type',
	count(*) 'Migrated System Count',
	count(*) 'Staging Count',
	0 'Difference'
from
	topic_dmcommission
group by
	commissiontype ;
 

### 22-Appropriation Type wise Commission Counts
select
	appropriationtypecd 'Appropriation Type',
	commissiontype 'Commission Type',
	(case
		when fundtype = '05' then '05-SIF'
		when fundtype = '06' then '06-OIF'
		else '03-Singapore Life Non Par Fund'
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
 

### 23-Agent Wise Commission Type wise count and Sum of Commission amount
select
	c.agentcode 'Agent Code',
	c.agentcategorycode 'Designation Code(GI Category Code)',
	c.commissiontype 'Commission Type',
	(case
		when c.fundtype = '05' then '05-SIF'
		when c.fundtype = '06' then '06-OIF'
		else '03-Singapore Life Non Par Fund'
	end) Fund,
	f.product_code 'Product Code',
	c.commissionstatus 'Commission Status',
	count(c.primarykey) 'DN/CN Count' ,
	year(c.commissiondate) 'Comm Gen Dt Year',
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
 

### 24-Data Migration total count
select 'dmpolicyheader' as 'Table', count(1) as 'Count' from topic_dmpolicyheader td union all
select 'dmpolicyhierarchy' as 'Table', count(1) as 'Count' from topic_dmpolicyhierarchy td union all
select 'dmpolicyproductdetails' as 'Table', count(1) as 'Count' from topic_dmpolicyproductdetails union all 
select 'dmpolicyclientdetails' as 'Table', count(1) as 'Count' from topic_dmpolicyclientdetails union all 
select 'dmcommission' as 'Table', count(1) as 'Count' from topic_dmcommission union all
select 'dmpremium' as 'Table', count(1) as 'Count' from topic_dmpremium;
 

select * from topic_dmcommission h where not exists (
	select '1'  from topic_dmpolicyheader p where p.policynum=h.policynum
);

~~~
