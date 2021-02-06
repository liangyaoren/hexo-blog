---
title: 记录一次 group by 查询的优化
date: 2020-10-26 08:48:46
tags: mysql
---
最近维护一个老系统，有一处查询非常慢，细细研究一番，优化了查询 sql。
该 sql 用来统计摄像机所有状态对应的数量，没加联合索引之前很慢，最后加了三个字段的联合索引优化到一秒左右。
check_day, camera_id, diagnosis_result
要注意，group by 的列也要加到组合索引里面去，这样查询才快。

```sql
SELECT
	ifnull(a.diagnosis_result, '') diagnosisResult,
	count(1) total
FROM
	video_camera aa
LEFT JOIN video_intact_diagnosis_status_201912 a ON (
	aa.id = a.camera_id
	AND a.check_day = '20191223'
)
LEFT JOIN unit_project p ON p.id = aa.project_id
WHERE
	aa.del_flag = 0
GROUP BY
	a.diagnosis_result
```