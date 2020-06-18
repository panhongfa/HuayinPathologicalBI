# 远程和中心病理数据库结构
## 远程病理数据库(HY_Telepathology)
```sql
select
 ic.IllnessCaseID '序号',
 ic.DiagnoseType '病种类别',
 ic.PathologyCode '病理编号',
 ic.BarCode '条码',
 ic.Name '姓名',
 ic.Sex '性别',
 ic.Age '年龄',
 ic.AgeUnit '年龄单位',
 ic.Tel '联系电话',
 h.HospitalName '送检单位',
 ic.OutpatientCode+'/'+ic.HospitalizeCode as '门诊/住院号',
 ic.DeliverDate '接收时间',
 ic.MaterialNumber '脱水盒',
 ic.SliceNumber '切片数量',
 ic.DeliverDepartment '送检科室',
 replace(replace(ic.DeliverMaterial, char(13), ''), char(10), '') as '送检部位',
 ic.CostItemName '项目名称',
 ic.BedCode '床位号',
 ic.DeliverDoctor '送检医生',
 cu.UserName '录入人员',
 ic.CreateDate '录入时间',
 au.UserName '初筛医生',
 ica.UpdateDate '初筛时间',
 replace(replace(ic.PreviousDiagnosis, char(13), ''), char(10), '') as '病理诊断(初审)',
 du.UserName '报告医生',
 su.UserName '复查医生',
 ru.UserName '签发医生',
 ics.DiagnosisDate '报告时间',
 replace(replace(ic.ClinicalDiagnosis , char(13), ''), char(10), '') as '临床诊断',
 replace(replace(ic.Perusal, char(13), ''), char(10), '') as '肉眼所见',
 replace(replace(ic.PreviousDiagnosis, char(13), ''), char(10), '') as '光镜所见',
 replace(replace(ica.Diagnose, char(13), ''), char(10), '') as '病理诊断',
 ic.CostTotal '标准费用',
 ic.IsFrozen '是否冰冻',
 ic.Remark '备注'
from T_IllnessCase ic
left join T_IllnessCase_Assistant ica on ica.IllnessCaseID=ic.IllnessCaseID
left join T_IllnessCaseSpecialist ics on ics.IllnessCaseID=ic.IllnessCaseID
left join T_Hospital h on h.HospitalID=ic.HospitalID
left join T_User cu on cu.UserID=ic.CreateUserID
left join T_User au on au.UserID=ica.UserID
left join T_User du on du.UserID=ica.UserID
left join T_User su on su.UserID=ics.SpecialistID
left join T_User ru on ru.UserID=ics.ReportSpecialist
where ic.DeliverDate >= '2018-1-1' and ic.DeliverDate <= '2020-5-31'
and h.HospitalName like '南方医科大学南海%'
and h.HospitalName like '广州市花都区胡忠%'
and h.HospitalName like '东莞广济%'
and h.HospitalName like '吴川市人民%'
and h.HospitalName like '%利辛县人民医院%'
order by ic.DeliverDate
```

## 中心病理数据库(HY_Centpathology)
- 汇总
```sql

DECLARE @begin_time varchar(64)
DECLARE @end_time varchar(64)
DECLARE @hospital_name varchar(512)

SET @begin_time='2018-1-1'
SET @end_time='2020-5-31'

/*
SET @hospital_name='南方医科大学南海%'
SET @hospital_name='广州市花都区胡忠%'
SET @hospital_name='东莞广济%'
SET @hospital_name='吴川市人民%'
*/
SET @hospital_name='南方医科大学南海%'
;

WITH
frist_diagnosis as
(
  select top 1 rq, blzd from  t_blcz_log WHERE t_blcz_log.ybid= ybid and t_blcz_log.czlx = '初筛签名' order by finterid desc
),
review_diagnosis as
(
  select top 1 blzd from  t_blcz_log WHERE t_blcz_log.ybid= ybid and t_blcz_log.czlx = '报告签名' order by finterid desc
)

(
SELECT
src_table.finterid '序号',
'病种类别'=NULL,
src_table.blbh '病理编号',
src_table.ybid '条码',
src_table.brxm '姓名',
src_table.brsex '性别',
src_table.brage '年龄',
'年龄单位'=NULL,
src_table.brlxfs '联系电话',
src_table.sjhospital '送检单位',
src_table.zyh '住院号',
src_table.jstime '接收时间',
src_table.hy_tshsl '脱水盒',
src_table.zzgs '切片数量',
src_table.sjks '送检科室',
replace(replace(src_table.sjbb, char(13), ''), char(10), '') as '送检部位',
src_table.xmmc_cn '项目名称',
src_table.cwh '床位号',
src_table.sjdoctor '送检医生',
src_table.lrr '录入人员',
src_table.lrsj '录入时间',
src_table.zddoctor '初筛医生',
frist_diagnosis.rq as '初筛时间',
replace(replace(frist_diagnosis.blzd, char(13), ''), char(10), '') as '病理诊断(初审)',
'报告医生'=NULL,
src_table.fcdoctor '复查医生',
replace(replace(review_diagnosis.blzd, char(13), ''), char(10), '') as '复查诊断',
src_table.qfdoctor '签发医生',
src_table.bgtime '报告时间',
replace(replace(src_table.lczd, char(13), ''), char(10), '') as '临床诊断',
replace(replace(src_table.rysj, char(13), ''), char(10), '') as '肉眼所见',
replace(replace(src_table.blsj, char(13), ''), char(10), '') as '光镜所见',
replace(replace(src_table.blzd, char(13), ''), char(10), '') as '病理诊断',
'标准费用'=NULL,
'是否冰冻'=NULL,
src_table.hy_zdlx '诊断类型',
src_table.ybzt '报告状态'
FROM t_bljbxxb as src_table, frist_diagnosis, review_diagnosis
where src_table.jstime >= @begin_time and src_table.jstime <= @end_time
and src_table.sjhospital like @hospital_name
)
UNION
(
SELECT
src_table.finterid '序号',
'病种类别'=NULL,
src_table.xbblxh '病理编号',
src_table.ybid '条码',
src_table.brxm '姓名',
src_table.brsex '性别',
src_table.brage '年龄',
'年龄单位'=NULL,
'联系电话'=NULL,
src_table.sjhospital '送检单位',
src_table.mzh '住院号',
src_table.jstime '接收时间',
'脱水盒'=NULL,
'切片数量'=NULL,
src_table.sjks '送检科室',
replace(replace(src_table.sjbb, char(13), ''), char(10), '') as '送检部位',
src_table.xmmc_cn '项目名称',
src_table.cwh '床位号',
src_table.sjdoctor '送检医生',
src_table.lrr '录入人员',
src_table.lrsj '录入时间',
src_table.zddoctor '初筛医生',
frist_diagnosis.rq as '初筛时间',
replace(replace(frist_diagnosis.blzd, char(13), ''), char(10), '') as '病理诊断(初审)',
'报告医生'=NULL,
src_table.fcdoctor '复查医生',
replace(replace(review_diagnosis.blzd, char(13), ''), char(10), '') as '复查诊断',
src_table.qfdoctor '签发医生',
src_table.bgtime '报告时间',
replace(replace(src_table.lczd, char(13), ''), char(10), '') as '临床诊断',
'肉眼所见'=NULL,
'光镜所见'=NULL,
replace(replace(cast(src_table.xbblxzd as varchar(max)), char(13), ''), char(10), '') as '病理诊断',
'标准费用'=NULL,
'是否冰冻'=NULL,
src_table.hy_zdlx '诊断类型',
src_table.ybzt '报告状态'
FROM t_tctbgd as src_table, frist_diagnosis, review_diagnosis
where src_table.jstime >= @begin_time and src_table.jstime <= @end_time
and src_table.sjhospital like @hospital_name
)
order by src_table.jstime

```



- 列重命名(活检)
```sql
WITH
frist_diagnosis as
(
  select top 1 rq, blzd from  t_blcz_log WHERE t_blcz_log.ybid= ybid and t_blcz_log.czlx = '初筛签名' order by finterid desc
),
review_diagnosis as
(
  select top 1 blzd from  t_blcz_log WHERE t_blcz_log.ybid= ybid and t_blcz_log.czlx = '报告签名' order by finterid desc
)

SELECT
src_table.finterid '序号',
'病种类别'=NULL,
src_table.blbh '病理编号',
src_table.ybid '条码',
src_table.brxm '姓名',
src_table.brsex '性别',
src_table.brage '年龄',
'年龄单位'=NULL,
src_table.brlxfs '联系电话',
src_table.sjhospital '送检单位',
src_table.zyh '住院号',
src_table.jstime '接收时间',
src_table.hy_tshsl '脱水盒',
src_table.zzgs '切片数量',
src_table.sjks '送检科室',
replace(replace(src_table.sjbb, char(13), ''), char(10), '') as '送检部位',
src_table.xmmc_cn '项目名称',
src_table.cwh '床位号',
src_table.sjdoctor '送检医生',
src_table.lrr '录入人员',
src_table.lrsj '录入时间',
src_table.zddoctor '初筛医生',
frist_diagnosis.rq as '初筛时间',
replace(replace(frist_diagnosis.blzd, char(13), ''), char(10), '') as '病理诊断(初审)',
'报告医生'=NULL,
src_table.fcdoctor '复查医生',
replace(replace(review_diagnosis.blzd, char(13), ''), char(10), '') as '复查诊断',
src_table.qfdoctor '签发医生',
src_table.bgtime '报告时间',
replace(replace(src_table.lczd, char(13), ''), char(10), '') as '临床诊断',
replace(replace(src_table.rysj, char(13), ''), char(10), '') as '肉眼所见',
replace(replace(src_table.blsj, char(13), ''), char(10), '') as '光镜所见',
replace(replace(src_table.blzd, char(13), ''), char(10), '') as '病理诊断',
src_table.hy_zdlx '诊断类型',
src_table.ybzt '报告状态'
FROM t_bljbxxb as src_table, frist_diagnosis, review_diagnosis
where src_table.jstime >= '2019-01-01' and src_table.jstime <= '2019-12-31'
and src_table.sjhospital like '南方医科大学南海%'
and src_table.sjhospital like '广州市花都区胡忠%'
and src_table.sjhospital like '东莞广济%'
and src_table.sjhospital like '吴川市%'
order by src_table.jstime
```

- 列重命名(TCT)
```sql
WITH
frist_diagnosis as
(
  select top 1 rq, blzd from  t_blcz_log WHERE t_blcz_log.ybid= ybid and t_blcz_log.czlx = '初筛签名' order by finterid desc
),
review_diagnosis as
(
  select top 1 blzd from  t_blcz_log WHERE t_blcz_log.ybid= ybid and t_blcz_log.czlx = '报告签名' order by finterid desc
)

SELECT TOP 100
src_table.finterid '序号',
'病种类别'=NULL,
src_table.xbblxh '病理编号',
src_table.ybid '条码',
src_table.brxm '姓名',
src_table.brsex '性别',
src_table.brage '年龄',
'年龄单位'=NULL,
'联系电话'=NULL,
src_table.sjhospital '送检单位',
src_table.mzh '住院号',
src_table.jstime '接收时间',
'脱水盒'=NULL,
'切片数量'=NULL,
src_table.sjks '送检科室',
replace(replace(src_table.sjbb, char(13), ''), char(10), '') as '送检部位',
src_table.xmmc_cn '项目名称',
src_table.cwh '床位号',
src_table.sjdoctor '送检医生',
src_table.lrr '录入人员',
src_table.lrsj '录入时间',
src_table.zddoctor '初筛医生',
frist_diagnosis.rq as '初筛时间',
replace(replace(frist_diagnosis.blzd, char(13), ''), char(10), '') as '病理诊断(初审)',
'报告医生'=NULL,
src_table.fcdoctor '复查医生',
replace(replace(review_diagnosis.blzd, char(13), ''), char(10), '') as '复查诊断',
src_table.qfdoctor '签发医生',
src_table.bgtime '报告时间',
replace(replace(src_table.lczd, char(13), ''), char(10), '') as '临床诊断',
'肉眼所见'=NULL,
'光镜所见'=NULL,
replace(replace(cast(src_table.xbblxzd as varchar(max)), char(13), ''), char(10), '') as '病理诊断',
src_table.hy_zdlx '诊断类型',
src_table.ybzt '报告状态'
FROM t_tctbgd as src_table, frist_diagnosis, review_diagnosis
where src_table.jstime >= '2019-01-01' and src_table.jstime <= '2020-05-31'
order by src_table.jstime
```

## 导出所有医院
```sql
select
h.HospitalName, h.Address, h.CreateDate
from T_Hospital h
```

```sql
(
  SELECT
  distinct src_table.sjhospital '送检单位'
  FROM t_bljbxxb as src_table
)
UNION
(
  SELECT
  distinct src_table.sjhospital '送检单位',
  FROM t_tctbgd as src_table
)

```
