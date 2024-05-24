### 一、检查配置时间

目前会根据数据createdAt和配置中指定的incrementalDataMappingDate区分为两类数据，

1. createdAt>incrementalDataMappingDate 的是增量数据，同步到Policy Interface、Recon Report相关表
2. createdAt<incrementalDataMappingDate 的是历史数据，同步到Data Migration相关表

所以在推送数据前需要检查incrementalDataMappingDate这个配置项的值，以此来控制数据走向

可以使用Postman调用接口来查看和修改这个配置：

~~~text
--- 查看
GET https://apibusiness3.uat.income.com.sg/sandbox/list/incrementalDataMappingDate/config

--- 修改
POST https://apibusiness3.uat.income.com.sg/sandbox/incrementalDataMappingDate/adjust
{
    "effDate": "10/11/2020"
}
~~~

> 注：这里可能要看看UAT3是不是启动了两个容器，如果是，上面的API就可能没办法保证同时更新了两个容器里面的这个配置，到时候两个容器交替处理，没办法保证数据走向

### 二、推送数据

> 注：最近的修改中涉及到了数据层级的修改，所以建议在UAT3重复推送数据前将原来的数据先删掉
>
> 1. 按policy no推数据，则按policy no将对应表中指定policy no的数据删掉后再重刷
> 2. 按时间整体刷数据，则建议直接把对应表的数据全清掉再重刷

推送数据的脚本如下：

1. 查看cdc_sync_rec表

   ~~~sql
   select count(*) from cdc_sync_rec;
   select * from cdc_sync_rec;
   delete from cdc_sync_rec where ref_class_name = "ri_risk_accu_trans";
   truncate table cdc_sync_rec; 
   select * from cdc_sync_rec_log;
   truncate table cdc_sync_rec_log; 
   ~~~

2. 全量刷SC（这个不用每次都刷，只有清库后第一次需要同步）

   ~~~sql
   # 全量刷Sc
   INSERT INTO cdc_sync_rec (rec_id, ref_id, ref_class_name, ref_type, retry_times, created_at, created_by, `level`, destination)
   select sc_id,sc_id,"Sc","sc",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",10000,'S3' from sc;
   ~~~

3. 按时间范围刷数据

   ~~~sql
   set @startTime='2023-08-01 00:00:00';
   set @endTime='2023-08-15 00:00:00';
   select @startTime,@endTime;
   
   # LA 全量刷数据
   INSERT INTO cdc_sync_rec (rec_id, ref_id, ref_class_name, ref_type, retry_times, created_at, created_by, `level`, destination)
   # ma
   select distinct ma_id,ma_id ,"Ma","ma",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8990,'S3' from ma where created_at between @startTime and @endTime union all
   # policy
   select distinct policy_id,policy_id,"Policy","policy",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8980,'S3' from policy where created_at between @startTime and @endTime union all
   # endo
   select distinct endo_id,endo_id,"Endo","endo",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8970,'S3' from endo where created_at between @startTime and @endTime union all
   # maEndo
   select distinct endo_id,endo_id,"MaEndo","ma_endo",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8965,'S3' from ma_endo where created_at between @startTime and @endTime union all
   # maDeclaration
   select distinct declaration_id,declaration_id,"MaDeclaration","ma_declaration",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8940,'S3' from ma_declaration where created_at between @startTime and @endTime union all
   # policyFeeBillingschedule
   select distinct schedule_id,schedule_id,"PolicyFeeBillingSchedule","policy_fee_billing_schedule",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8935,'S3' from policy_fee_billing_schedule where created_at between @startTime and @endTime union all
   # billingschedule
   select distinct schedule_id,schedule_id,"BillingSchedule","billing_schedule",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8930,'S3' from billing_schedule where created_at between @startTime and @endTime union all
   # lifepolicyrenewal
   select distinct policy_renewal_id,policy_renewal_id,"LifePolicyRenewal","life_policy_renewal",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8920,'S3' from life_policy_renewal where created_at between @startTime and @endTime union all
   # coareq
   select distinct req_id,req_id,"CoaReq","coa_req",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8910,'S3' from coa_req where created_at between @startTime and @endTime union all
   # coareqitem
   select distinct item_id,item_id,"CoaReqItem","coa_req_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8900,'S3' from coa_req_item where created_at between @startTime and @endTime union all
   # bill
   select distinct bill_id,bill_id,"Bill","bill",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5990,'S3' from bill where created_at between @startTime and @endTime union all
   # fee
   select distinct fee_id,fee_id,"Fee","fee",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5980,'S3' from fee where fee_type = "GWP" and created_at between @startTime and @endTime union all
   # collect
   select distinct collect_id,collect_id,"Collection","collect",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5970,'S3' from `collect` where created_at between @startTime and @endTime union all
   # collectItem
   select distinct item_id,item_id,"CollectionItem","collect_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5960,'S3' from collect_item where created_at between @startTime and @endTime union all
   # payment
   select distinct payment_id,payment_id,"Payment","payment",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5950,'S3' from payment where created_at between @startTime and @endTime union all
   # paymentItem
   select distinct item_id,item_id,"PaymentItem","payment_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5940,'S3' from payment_item where created_at between @startTime and @endTime union all
   # offset
   select distinct offset_id,offset_id,"Offset","off_set",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5930,'S3' from off_set where created_at between @startTime and @endTime union all
   # offsetItem
   select distinct item_id,item_id,"OffsetItem","offset_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5920,'S3' from offset_item where created_at between @startTime and @endTime;
   
   # LA 刷触发点数据
   INSERT INTO cdc_sync_rec (rec_id, ref_id, ref_class_name, ref_type, retry_times, created_at, created_by, `level`, destination)
   # ma
   select distinct ma_id,ma_id ,"Ma","ma",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8990,'S3' from ma where created_at between @startTime and @endTime union all
   # endo
   select distinct endo_id,endo_id,"Endo","endo",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8970,'S3' from endo where created_at between @startTime and @endTime union all
   # maEndo
   select distinct endo_id,endo_id,"MaEndo","ma_endo",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8965,'S3' from ma_endo where created_at between @startTime and @endTime union all
   # maDeclaration
   select distinct declaration_id,declaration_id,"MaDeclaration","ma_declaration",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8940,'S3' from ma_declaration where created_at between @startTime and @endTime union all
   # coareqitem
   select distinct item_id,item_id,"CoaReqItem","coa_req_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8900,'S3' from coa_req_item where created_at between @startTime and @endTime union all
   # fee
   select distinct fee_id,fee_id,"Fee","fee",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5980,'S3' from fee where fee_type = "GWP" and created_at between @startTime and @endTime union all
   # collectItem
   select distinct item_id,item_id,"CollectionItem","collect_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5960,'S3' from collect_item where created_at between @startTime and @endTime union all
   # paymentItem
   select distinct item_id,item_id,"PaymentItem","payment_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5940,'S3' from payment_item where created_at between @startTime and @endTime union all
   # offsetItem
   select distinct item_id,item_id,"OffsetItem","offset_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5920,'S3' from offset_item where created_at between @startTime and @endTime;
   
   # DM 全量刷数据
   INSERT INTO cdc_sync_rec (rec_id, ref_id, ref_class_name, ref_type, retry_times, created_at, created_by, `level`, destination)
   # ma
   select distinct ma_id,ma_id ,"Ma","ma",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8990,'S3' from ma where created_at between @startTime and @endTime union all
   # policy
   select distinct policy_id,policy_id,"Policy","policy",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8980,'S3' from policy where created_at between @startTime and @endTime union all
   # endo
   select distinct endo_id,endo_id,"Endo","endo",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8970,'S3' from endo where created_at between @startTime and @endTime union all
   # maDeclaration
   select distinct declaration_id,declaration_id,"MaDeclaration","ma_declaration",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8940,'S3' from ma_declaration where created_at between @startTime and @endTime union all
   # policyFeeBillingschedule
   select distinct schedule_id,schedule_id,"PolicyFeeBillingSchedule","policy_fee_billing_schedule",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8935,'S3' from policy_fee_billing_schedule where created_at between @startTime and @endTime union all
   # billingschedule
   select distinct schedule_id,schedule_id,"BillingSchedule","billing_schedule",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8930,'S3' from billing_schedule where created_at between @startTime and @endTime union all
   # lifepolicyrenewal
   select distinct policy_renewal_id,policy_renewal_id,"LifePolicyRenewal","life_policy_renewal",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8920,'S3' from life_policy_renewal where created_at between @startTime and @endTime union all
   # coareq
   select distinct req_id,req_id,"CoaReq","coa_req",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8910,'S3' from coa_req where created_at between @startTime and @endTime union all
   # coareqitem
   select distinct item_id,item_id,"CoaReqItem","coa_req_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8900,'S3' from coa_req_item where created_at between @startTime and @endTime union all
   # bill
   select distinct bill_id,bill_id,"Bill","bill",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5990,'S3' from bill where bill_type in ('DIRECT_PREM','DIRECT_COMM','OR_COMM','INCENTIVE') and created_at between @startTime and @endTime union all
   # fee
   select distinct fee_id,fee_id,"Fee","fee",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5980,'S3' from fee where fee_type in ('GWP','BASIC_COMM','OR_COMM','INCENTIVE') and created_at between @startTime and @endTime;
   
   # DM 刷触发点数据
   INSERT INTO cdc_sync_rec (rec_id, ref_id, ref_class_name, ref_type, retry_times, created_at, created_by, `level`, destination)
   # bill
   select distinct bill_id,bill_id,"Bill","bill",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5990,'S3' from bill where bill_type in ('DIRECT_PREM','DIRECT_COMM','OR_COMM','INCENTIVE') and created_at between @startTime and @endTime union all
   # fee
   select distinct fee_id,fee_id,"Fee","fee",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5980,'S3' from fee where fee_type in ('GWP','BASIC_COMM','OR_COMM','INCENTIVE') and created_at between @startTime and @endTime;
   
   
   # RR 全量刷数据
   INSERT INTO cdc_sync_rec (rec_id, ref_id, ref_class_name, ref_type, retry_times, created_at, created_by, `level`, destination)
   # ma
   select distinct ma_id,ma_id ,"Ma","ma",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8990,'S3' from ma where created_at between @startTime and @endTime union all
   # policy
   select distinct policy_id,policy_id,"Policy","policy",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8980,'S3' from policy where created_at between @startTime and @endTime union all
   # endo
   select distinct endo_id,endo_id,"Endo","endo",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8970,'S3' from endo where created_at between @startTime and @endTime union all
   # maDeclaration
   select distinct declaration_id,declaration_id,"MaDeclaration","ma_declaration",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8940,'S3' from ma_declaration where created_at between @startTime and @endTime union all
   # policyFeeBillingschedule
   select distinct schedule_id,schedule_id,"PolicyFeeBillingSchedule","policy_fee_billing_schedule",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8935,'S3' from policy_fee_billing_schedule where created_at between @startTime and @endTime union all
   # billingschedule
   select distinct schedule_id,schedule_id,"BillingSchedule","billing_schedule",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8930,'S3' from billing_schedule where created_at between @startTime and @endTime union all
   # lifepolicyrenewal
   select distinct policy_renewal_id,policy_renewal_id,"LifePolicyRenewal","life_policy_renewal",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8920,'S3' from life_policy_renewal where created_at between @startTime and @endTime union all
   # coareq
   select distinct req_id,req_id,"CoaReq","coa_req",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8910,'S3' from coa_req where created_at between @startTime and @endTime union all
   # coareqitem
   select distinct item_id,item_id,"CoaReqItem","coa_req_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8900,'S3' from coa_req_item where created_at between @startTime and @endTime union all
   # bill
   select distinct bill_id,bill_id,"Bill","bill",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5990,'S3' from bill where bill_type in ('DIRECT_COMM','OR_COMM','INCENTIVE') and created_at between @startTime and @endTime union all
   # fee
   select distinct fee_id,fee_id,"Fee","fee",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5980,'S3' from fee where fee_type in ('BASIC_COMM','OR_COMM','INCENTIVE') and created_at between @startTime and @endTime;
   
   # RR 刷触发点数据
   INSERT INTO cdc_sync_rec (rec_id, ref_id, ref_class_name, ref_type, retry_times, created_at, created_by, `level`, destination)
   # bill
   select distinct bill_id,bill_id,"Bill","bill",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5990,'S3' from bill where bill_type in ('DIRECT_COMM','OR_COMM','INCENTIVE') and created_at between @startTime and @endTime union all
   # fee
   select distinct fee_id,fee_id,"Fee","fee",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5980,'S3' from fee where fee_type in ('BASIC_COMM','OR_COMM','INCENTIVE') and created_at between @startTime and @endTime;
   
   INSERT INTO cdc_sync_rec (rec_id, ref_id, ref_class_name, ref_type, retry_times, created_at, created_by, `level`, destination)
   # fee
   select distinct fee_id,fee_id,"Fee","fee",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5980,'S3' from fee where fee_type in ('BASIC_COMM','OR_COMM','INCENTIVE') and created_at between @startTime and @endTime;
   ~~~

4. 按照policyNo刷数据（目前Policy Interface指定过需要刷新的policy no list会放在最后面）

   ~~~sql
   # 1.先准备DMS_policyNumbers 建一张表来存policyNo，用于后续的条件,每次刷之前先将需要的policyNo放入这个表里面
   drop table DMS_policyNumbers;
   CREATE table DMS_policyNumbers (
   	`policyNo` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL
   );
   insert into DMS_policyNumbers (policyno)
   SELECT policyNo COLLATE utf8mb4_general_ci policyNo
   FROM JSON_TABLE(  
     '["2100000086"]',  
     '$[*]'  
     COLUMNS (  
       policyNo VARCHAR(50) PATH '$'  
     )  
   ) jt;
   select * from DMS_policyNumbers;
   truncate table DMS_policyNumbers;
   
   # 2.执行查询语句将需要同步的数据插入cdc_sync_rec
   ## LA
   INSERT INTO cdc_sync_rec (rec_id, ref_id, ref_class_name, ref_type, retry_times, created_at, created_by, `level`, destination)
   # ma
   select distinct ma_id,ma_id,"Ma","ma",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8990,'S3' from trans_index where ma_no in (select * from DMS_policyNumbers)
   union all
   # policy
   select distinct policy_id,policy_id ,"Policy","policy",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8980,'S3' from trans_index where quote_no in (select * from DMS_policyNumbers) or quote_no in (
   	select quote_no from trans_index where ma_no in (select * from DMS_policyNumbers)
   )
   union all
   # endo
   select distinct endo_id,endo_id ,"Endo","endo",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8970,'S3' from trans_index where (quote_no in (select * from DMS_policyNumbers) or quote_no in (
   	select quote_no from trans_index where ma_no in (select * from DMS_policyNumbers)
   )) and endo_id is not null
   union all
   # maEndo
   select distinct endo_id,endo_id,"MaEndo","ma_endo",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8965,'S3' from ma_endo where ma_id in (select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers))
   union all
   # maDeclaration
   select distinct declaration_id,declaration_id ,"MaDeclaration","ma_declaration",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8940,'S3' from ma_declaration where ma_id  in (
   	select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers)
   )
   union all
   # policyFeeBillingschedule
   select distinct schedule_id,schedule_id,"PolicyFeeBillingSchedule","policy_fee_billing_schedule",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8935,'S3' from policy_fee_billing_schedule where 
   	ma_id in (select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers)) or policy_id in (select policy_id from trans_index where quote_no in (select * from DMS_policyNumbers))
   union all 
   # billingschedule
   select distinct schedule_id,schedule_id,"BillingSchedule","billing_schedule",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8930,'S3' from billing_schedule where ref_id in (
   	select distinct schedule_id from policy_fee_billing_schedule where 
   	ma_id in (select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers)) or policy_id in (select policy_id from trans_index where quote_no in (select * from DMS_policyNumbers))
   )
   union all
   # lifepolicyrenewal
   select distinct policy_renewal_id,policy_renewal_id,"LifePolicyRenewal","life_policy_renewal",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8920,'S3' from life_policy_renewal where policy_id in (
   	select policy_id from trans_index where quote_no in (select * from DMS_policyNumbers)
   )
   union all
   # coareq
   select distinct req_id,req_id,"CoaReq","coa_req",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8910,'S3' from coa_req where req_id in (
   	select distinct coa_req_id from coa_req_item where ref_id in (
   		select policy_id from trans_index where quote_no in (select * from DMS_policyNumbers) union
   		select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers)
   	)
   ) 
   union all
   # coareqitem
   select distinct item_id,item_id,"CoaReqItem","coa_req_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8900,'S3' from coa_req_item where ref_id in (
   	select policy_id from trans_index where quote_no in (select * from DMS_policyNumbers) union
   	select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers)
   )
   union all
   # bill
   select distinct bill_id,bill_id,"Bill","bill",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5990,'S3' from bill where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   union all
   # fee
   select distinct fee_id,fee_id,"Fee","fee",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5980,'S3' from fee where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   union all
   # collect
   select distinct collect_id,collect_id,"Collection","collect",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5970,'S3' from `collect` where collect_id in (
   	select distinct collect_id from collect_item where bill_id in (
   		select distinct bill_id from bill where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   	)
   )
   union all
   # collectItem
   select distinct item_id,item_id,"CollectionItem","collect_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5960,'S3' from collect_item where bill_id in (
   	select distinct bill_id from bill where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   )
   union all
   # payment
   select distinct payment_id,payment_id,"Payment","payment",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5950,'S3' from payment where payment_id in (
   	select distinct payment_id from payment_item where bill_id in (
   		select distinct bill_id from bill where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   	)
   )
   union all
   # paymentItem
   select distinct item_id,item_id,"PaymentItem","payment_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5940,'S3' from payment_item where bill_id in (
   	select distinct bill_id from bill where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   )
   union all
   # offset
   select distinct offset_id,offset_id,"Offset","off_set",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5930,'S3' from off_set where offset_id in (
   	select distinct offset_id from offset_item where from_bill_id in (
   		select distinct bill_id from bill where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   	) or to_bill_id in (
   		select distinct bill_id from bill where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   	)
   )
   union all
   # offsetItem
   select distinct item_id,item_id,"OffsetItem","offset_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5920,'S3' from offset_item where from_bill_id in (
   	select distinct bill_id from bill where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   ) or to_bill_id in (
   	select distinct bill_id from bill where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   );
   
   
   ## DM and RR
   INSERT INTO cdc_sync_rec (rec_id, ref_id, ref_class_name, ref_type, retry_times, created_at, created_by, `level`, destination)
   # ma
   select distinct ma_id rec_id,ma_id ref_id,"Ma" ref_class_name,"ma" ref_type,0 retry_times,now() created_at,"1ecc5295c5de60b4af63f9375c07ff594fe0a8" created_by,8990 `level`,'S3' destination from trans_index where ma_no in (select * from DMS_policyNumbers)
   union all
   # policy
   select distinct policy_id,policy_id ,"Policy","policy",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8980,'S3' from trans_index where quote_no in (select * from DMS_policyNumbers) or quote_no in (
   	select quote_no from trans_index where ma_no in (select * from DMS_policyNumbers)
   )
   union all
   # endo
   select distinct endo_id,endo_id ,"Endo","endo",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8970,'S3' from trans_index where (quote_no in (select * from DMS_policyNumbers) or quote_no in (
   	select quote_no from trans_index where ma_no in (select * from DMS_policyNumbers)
   )) and endo_id is not null
   union all
   # maDeclaration
   select distinct declaration_id,declaration_id ,"MaDeclaration","ma_declaration",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8940,'S3' from ma_declaration where ma_id  in (
   	select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers)
   )
   union all 
   # policyFeeBillingschedule
   select distinct schedule_id,schedule_id,"PolicyFeeBillingSchedule","policy_fee_billing_schedule",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8935,'S3' from policy_fee_billing_schedule where 
   	ma_id in (select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers)) or policy_id in (select policy_id from trans_index where quote_no in (select * from DMS_policyNumbers))
   union all 
   # billingschedule
   select distinct schedule_id,schedule_id,"BillingSchedule","billing_schedule",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8930,'S3' from billing_schedule where ref_id in (
   	select distinct schedule_id from policy_fee_billing_schedule where 
   	ma_id in (select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers)) or policy_id in (select policy_id from trans_index where quote_no in (select * from DMS_policyNumbers))
   )
   union all
   # lifepolicyrenewal
   select distinct policy_renewal_id,policy_renewal_id,"LifePolicyRenewal","life_policy_renewal",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8920,'S3' from life_policy_renewal where policy_id in (
   	select policy_id from trans_index where quote_no in (select * from DMS_policyNumbers)
   )
   union all
   # coareq
   select distinct req_id,req_id,"CoaReq","coa_req",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8910,'S3' from coa_req where req_id in (
   	select distinct coa_req_id from coa_req_item where ref_id in (
   		select policy_id from trans_index where quote_no in (select * from DMS_policyNumbers) union
   		select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers)
   	)
   ) 
   union all
   # coareqitem
   select distinct item_id,item_id,"CoaReqItem","coa_req_item",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",8900,'S3' from coa_req_item where ref_id in (
   	select policy_id from trans_index where quote_no in (select * from DMS_policyNumbers) union
   	select ma_id from trans_index where ma_no in (select * from DMS_policyNumbers)
   )
   union all
   # bill
   select distinct bill_id,bill_id,"Bill","bill",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5990,'S3' from bill where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers)
   union all
   # fee
   select distinct fee_id,fee_id,"Fee","fee",0,now(),"1ecc5295c5de60b4af63f9375c07ff594fe0a8",5980,'S3' from fee where policy_no in (select * from DMS_policyNumbers) or ma_no in (select * from DMS_policyNumbers);
   ~~~

4. 检查数据同步进度（watchmen meta库）

   ~~~sql
   # DMS-meta
   select * from trigger_event order by created_at desc;
   select * from trigger_table order by last_modified_at desc;
   
   select * from change_data_json;
   select * from change_data_json_history order by last_modified_at desc;
   
   select count(*) from change_data_record;
   select count(*) from change_data_json;
   select count(*) from scheduled_task; 
   
   select * from data_sources;
   select * from collector_module_config;
   select * from collector_model_config;
   select * from collector_table_config;
   ~~~

5. watchmen处理完数据后可以去watchmen data库看一下对应几张表的数据处理的是否正确

> 注：因为Data Migration这边通常是按时间整体刷数据，所以他们会检查DM这几张表里面的PolicyNo是否都能对的上，然后有个问题就是UAT3上面不知道是有什么Job在跑还是什么，会大批量的更新Bill，所以经常可能会出现DMPOLICYHIERARCHY这个由Bill触发的表的数据会比另外几个由Fee触发的表的数据多，所以可能需要在确定本次数据推送完成对DM这几张表的数据做一次清理，首先先把incrementalDataMappingDate时间改到20年，确保后面的数据不会再进DM的表，然后用下面的脚本进行数据清理和检查
>
> ~~~sql
> # 按policyHeader里面的policyNum去删剩余几张表的数据
> delete from topic_dmpolicyproductdetails where policynum not in (select distinct policynum from topic_dmpolicyheader);
> delete from topic_dmpolicyclientdetails where policynum not in (select distinct policynum from topic_dmpolicyheader);
> delete from topic_dmpremium where policynum not in (select distinct policynum from topic_dmpolicyheader);
> delete from topic_dmcommission where policynum not in (select distinct policynum from topic_dmpolicyheader);
> delete from topic_dmpolicyhierarchy where policynum not in (select distinct policynum from topic_dmpolicyheader);
> 
> select * from topic_dmpolicyproductdetails where policynum not in (select distinct policynum from topic_dmpolicyheader);
> select * from topic_dmpolicyclientdetails where policynum not in (select distinct policynum from topic_dmpolicyheader);
> select * from topic_dmpremium where policynum not in (select distinct policynum from topic_dmpolicyheader);
> select * from topic_dmcommission where policynum not in (select distinct policynum from topic_dmpolicyheader);
> select * from topic_dmpolicyhierarchy where policynum not in (select distinct policynum from topic_dmpolicyheader);
> 
> # 统计数量，这个应该会要在邮件里面告诉他们我们推完之后有多少数据
> select "topic_dmpolicyheader",count(*) from topic_dmpolicyheader union all
> select "topic_dmpolicyproductdetails",count(*) from topic_dmpolicyproductdetails union all
> select "topic_dmpolicyclientdetails",count(*) from topic_dmpolicyclientdetails union all
> select "topic_dmpremium",count(*) from topic_dmpremium union all
> select "topic_dmcommission",count(*) from topic_dmcommission union all
> select "topic_dmpolicyhierarchy",count(*) from topic_dmpolicyhierarchy;
> ~~~

### 三 日志等数据清理

第一次发布版本升级后的Watchmen时可能需要把这些表数据都请一次

~~~sql
## watchmen meta
TRUNCATE change_data_json;
TRUNCATE change_data_json_history;
TRUNCATE change_data_record;
TRUNCATE change_data_record_history;
TRUNCATE scheduled_task;
TRUNCATE scheduled_task_history;
TRUNCATE competitive_lock;
TRUNCATE trigger_event;
TRUNCATE trigger_module;
TRUNCATE trigger_model;
TRUNCATE trigger_table;

## watchemen data
truncate table topic_raw_pipeline_monitor_log; 
~~~

后面应该只要删这些log

~~~sql
## watchmen meta
TRUNCATE change_data_json_history;
TRUNCATE change_data_record_history;
TRUNCATE scheduled_task_history;

## watchemen data
truncate table topic_raw_pipeline_monitor_log; 
~~~

### 四 目前指定了的Policy Noumbers

> 目前只有Policy Interface是按Policy No是按指定的policy no刷的数据

~~~text
# 目前指定过需要测试的保单
"2100303751" ,"2100530392" ,"2100380632-03" ,"2100199211-14" ,"2100365779-09" ,
"2100459310" ,"2100399015" ,"2100400362" ,"2100293670-01" ,"2100397271-13" ,
"2100538558" ,"2100214464-02" ,"2100015256" ,"2100241989" ,"2100242002" ,
"2000219216" ,"2100000022" ,"2100120538" ,"2100134130-04" ,"2100241658" ,
"2100267063-01" ,"2100250826" ,"2100116490-03" ,"2100307893" ,"2100160121" ,
"2100022171" ,"2100301426" ,"2100256186" ,"2100377537" ,"2100154276-06" ,
"2100186625-01" ,"2100304150" ,"2100123575-01" ,"2100067670-12" ,"2100538000" ,
"2100586290" ,"2100538325" ,"2000353722" ,"2100116249-04" ,"2100379166-01" ,
"2100367406-01" ,"2100278208" ,"2100403014" ,"2100118065-02" ,"2100395286" ,
"2100532734" ,"2100209267-03" ,"2100130025-01" ,"2100025038-01" ,"2100121802-01" ,
"2100672402" ,"2100568190-01" ,"2100514469" ,"2100536661-01" ,"2100517844" ,
"2100357017-01" ,"2000194296" ,"2000137022" ,"2000216779" ,"2000218460" ,
"2000218479" ,"2000407743" ,"2000218817" ,"2000025054" ,"2000022642" ,
"2000004251" ,"2000110450" ,"2000027585" ,"2000195711" ,"2000119435" ,
"2000046375" ,"2000167066" ,"2000018175" ,"2000216797" ,"2000217310" ,
"2000217392" ,"2000371855" ,"2000217418" ,"2000217436" ,"2000211907" ,
"2000217463" ,"2000216242" ,"2000178783" ,"2000019752" ,"2000007421" ,
"2000216457" ,"2000030993" ,"2000137317" ,"2000078028" ,"2100271487-01" ,
"2100521580-01" ,"2100499387-01" ,"2100357811-01" ,"2100519044-01" ,
"2100088526-02" ,"2100129086-02" ,"2000363997" ,"2000423087" ,"2000364118" ,
"2100640937" ,"2100653130-01" ,"2000208026" ,"2000211943" ,"2000213885" ,
"2000215236" ,"2000227656" ,"2000229794" ,"2000255552" ,"2000255972" ,
"2000261247" ,"2000273523" ,"2000295967" ,"2000297050" ,"2000325988" ,
"2000330989" ,"2000338921" ,"2000379244" ,"2000436084" ,"2000457814" ,
"2000482737" ,"2100021307-02" ,"2100064946-04" ,"2100065372-03" ,
"2100065658-03" ,"2100066271-04" ,"2100124465-02" ,"2100144010-02" ,
"2100237707-02" ,"2100290928-04" ,"2100328589-04" ,"2100346952-07" ,
"2100348901-06" ,"2100379166-02" ,"2100395759-01" ,"2100399499-01" ,
"2100453783-01" ,"2100490302" ,"2100492182" ,"2100492842" ,"2100494391" ,
"2100496671" ,"2100583100" ,"2100584730" ,"2100587050" ,"2100627160" ,
"2100630930" ,"2100649580" ,"2000012557" ,"2000111108" ,"2000186712" ,
"2100015381-01" ,"2100017045-01" ,"2100025494-01" ,"2100066600-02" ,
"2100085641-13" ,"2100094668-01" ,"2100132903-03" ,"2100225047-01" ,
"2100250826-01" ,"2100298817-02" ,"2100308283" ,"2100313597" ,"2100343184" ,
"2100346112" ,"2100361226" ,"2100371606" ,"2100374205" ,"2100384149" ,
"2100393111" ,"2100418373" ,"2100475681","2100130794-02", "2100131246-02"

# 目前UAT3存在的Affnity Policy，可能也会需要刷
"2300000704" ,"2300001158" ,"2300001247" ,"2300001265" ,"2300001532" ,"2300001907" ,"2300002164"
~~~



