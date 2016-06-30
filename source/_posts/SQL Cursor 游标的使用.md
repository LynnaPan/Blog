---
title:  SQL Cursor 游标的使用
date:   2016-05-31 21:54:53 +0800
categories: mssql 数据库
---
## SQL Cursor 游标的使用

这两天在做新老系统间的data migration，接触到sql的游标，记录总结一下。
    
我们的需求是要求map多张表，并把计算结果分别更新到一张目标表中, 新旧系统要做A/B Testing, 所以当旧表有任何更新，比如新增，删除，改动， 都要更新到新表中。

原本我选择的方案是采用批量Insert, 但碰到一个需要插入map关系的表， 其中一个field是另外一张表刚刚插入数据的id， 因此只能用循环来解决。 看了一圈SQL的for循环，实现起来略显费力。当然也有用`while do`或临时表等其他解决方案，欢迎发邮件探讨，这里分享一下游标的使用心得。

以我的理解，可以把游标想象成一个指针，我们使用一个指针的时候需要做的事情：

* 声明指针
* 把指针指向一个地址
* 获取指针内容
* 移动指针指向新的地址

这样理解起来游标就比较方便， 下面举个例子来说明游标的使用方法：

```` sql
--声明一些后面需要用到的面临
DECLARE @vm_bucket_id bigint, @group_id bigint, @name varchar(50)
--声明游标 
DECLARE MyCursor CURSOR 
--查询出数据结果集合，游标将指向这个集合	 
FOR (SELECT id, group_id, name FROM dbo.vm_bucket WHERE id NOT IN(
SELECT object_id from dbo.resource_group_map where object=29)) ORDER BY id 
--打开游标	
OPEN MyCursor 		
--从游标中取出内容放到声明好的变量中 
FETCH NEXT FROM MyCursor INTO @vm_bucket_id, @group_id, @name
--对取出的数据进行判断和处理
WHILE @@FETCH_STATUS = 0
		BEGIN
			--插入数据
			INSERT INTO dbo.resource_group
			(name, group_id, user_id, created_date)
			VALUES (@name, @group_id, 0, getutcdate())
		
			--把刚刚插入数据的id作为结果插入到另一张表
			INSERT INTO dbo.resource_group_map
			(resource_group_id, object, object_id)
			VALUES (@@identity, 29, @vm_bucket_id)

			--读取下一行
			FETCH NEXT FROM MyCursor INTO @vm_bucket_id, @group_id, @name
		END
--关闭游标
CLOSE MyCursor
--释放游标
DEALLOCATE MyCursor
````

<br/>
<b>总结：</b> 游标是一种较易理解易读的程序写作模型，但如果使用不当，很容易扼杀性能，谨慎使用。



