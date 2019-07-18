#### 1.为用户Wang定义一个学生-课程模式


    create schema"S-T" authorization Wang;

----------

#### 2.同时创建模式的时候还可以创建一个表/视图。

	create schema"S-T" authorization Wang
	create table tab1( col1 int,
	col2 char(20));

----------

#### 3.删除操作的 cascade和restrict

- cascade :全部下属对象都删除


- restrict :如果有下属对象的存在，就拒绝此条操作。


	drop schema S-T cascade

​	关键字直接跟在语句后面就行了

----------
#### 4.建立一个表，并且建立完整性约束

​	以下表时课程表，其中有一个先行课，必须是已有的课程，所以被参照表就是Course表中的cno字段

	create table Course(
		Cno char(4) primary key,
		cpno char(4),
		foreign key(cpno) references Course(Cno)
	);

----------
#### 5.以下是修改基本表的操作

​	(1)增加一列

	alter table Student add s_entrance DATE;

​	(2)修改数据类型

	alter table Student alter column Sage int; --教材写法，即SQL Server写法
	
	alter table test1 modify  sage int; --此为Oracle、MySQL写法，不用column，alter改为modify

​	(3)增加约束

	alter table student add unique(Cname);

----------
#### 6.以下是删除表的操作

​	删除分为两种：一种是删除表的数据，另一种是删除表的数据及删除表的结构。

​	1. in在oracle

	delete  test1 --删除表的数据 。 可加from也可不加
	
	drop table test1 --删除表的数据及结构

- cascade 和 restrict 的特性适用于以上的delete语句。

- truncate 跟delete类似，但是是永久删除，效率高过delete。

- oracle中drop table时 restrict 跟 cascade不可用。


​	2. in在mysql中

	 delete from test1 where --删除表数据 ,where语句必须要加;级联操作需修改safe-updates模式即可。
	
	delete from test3  -- 必须加个from，
	
	drop table test1 --删除表的数据及结构
	
	drop table test1 restrict --可以删除表，视图还在，但视图已经不能用了
	
	drop table test1 cascade --可以删除表，视图不在，视图变为not fetch...

----------
#### 7.建立、删除索引

​	索引能加快查询速度

​	索引的类型：

- 顺序文件上的索引：针对按指定属性值升序或降序存储，索引文件由属性值和相应的元组指针组成。
- B+树索引：将索引属性组织成B+树形式，B+树的叶节点为属性值和相应的元组指针。B+树具有动态平衡的优点。
- 散列索引：建立若干个桶，将索引属性按照其散列函数值映射到相应桶中，桶中存放索引属性值和相应的元组指针。具有查找速度快的优点。
- 位图索引：用位向量记录索引属性可能出现的值，每个位向量对应一个可能值。
  - 位图索引介绍：https://www.cnblogs.com/LBSer/p/3322630.html

	create unique index Stusno on Student(Sno);
	
	alter index Stusno rename to ssno;
	
	drop index Stusno;

----------
#### 8.数据查询

```
select [all|distinct] <目标列表达式>[,<目标列表达式>]

from <表名或视图名>

[where <条件表达式>]

[group by<列名1>[having <条件表达式>]]

[order by<列名2>[asc|desc]];
```


​	(1)消除重复行/增加一段标识/查询经过计算的值/列别名/大小写

	select distinct name,'Year of Birth',2019-age Birthday,lower(dept) from student; --计算出生年份

---

where子句:

​	(2)between and 的使用-包括两端

	select sname,sage from student where sage [not] between 20 and 23

​	(3) where not 的使用

	select sname,sage from student where not sage = 20;

​	(4)mysql和oracle都不支持!<   !>

​	(5)字符匹配

	[not] like'<匹配串>' [escape'换码字符']

​	换码字符的意思是，“换码字符就是在匹配串中的转义字符”，一般是用在要查的字段中就带有“%”、“_ ” ,从而使"%"、"_ "失去“癞子”功能。

- %：表示任意长度的字符，包括0


- _:表示任意的单个字符


	select * from student where sname [not] like '佟_%' and cname like 'DB\_%D'escape '\'; -- 这里的cname 有 DB_ABCD、DB_D、DB_GD等

​	占位符的模糊查询：

	SELECT * FROM User WHERE Name like ？
	
	ps.setString(1,"%刘%")

​	(6)is null / is not null

	select * from student where sname is [not] null;

​	(7)[not] in (x,y)

```
select * from student where sage [not] in('20','23'); --若数据库表中的数据是int类型，加不加单引号都行。
```

​	(8)or 的优先级小于 and

	   select * from student where sdept='IS' or ssex = '男' and sage < '22' or ssex = '女';
---

​	(9)order by 子句：

	select * from student order by sdept,sage desc/asc

​	(10)聚集函数：

​	count(*) 统计元祖个数

	select count(sno) from sc; --全部统计
	
	select count(distinct sno) from sc; --重复元素只计算一次

​	另外，oracle注释 --后面不需要空格；mysql的注释 --后面必须要有一个空格

- sum() --计算一列值的总和

- avg() --计算一列值的平均值

- max()--计算一列值的最大值

- min()--计算一列值的最小值


​	(11)group by 子句 / having 子句

	select sno,count(sno) from sc group by sno; --查询每个学生每人选修了多少门课程
	
	select sno from sc group by sno having count(sno/*)>2 --查询选修课程数大于3的学生

​	having子句跟where子句的区别：

​		作用的对象不一样，where子句作用于基本表或视图；having子句作用于组，从中选择满足条件的组。另外where子句不能使用聚集函数作为条件。



​	（12）连接查询

​	①等值与非等值连接查询

​		当运算符是=时，称为等值连接，其他运算符称为非等值连接

	select s.*,sc.* from student s,sc where s.sno = sc.sno; --student与sc表连接查询，sno出现两列

​		嵌套循环连接算法的基本思想：首先在student找到第一个元祖，然后在sc表中扫描，如果相等，就拼接起来形成结果表的一个元祖；然后student的第二个元祖，重新扫描sc...

​		如果在sc表sno上建立了索引，就不用每次全表扫描sc表了，而是根据sno值通过索引找到相应的sc元组。

​	② 自身连接

	select first.cno,second.cno from course first,course second where first.cno = second.cpno;

​	③ 外连接

	select s.*,sc.* from student s left outer join sc on(sc.sno = s.sno);

​	左外连接列出左边关系中的所有元组，右外连接列出右边关系中的所有元组。

​	④多表连接

​	两个以上的表进行连接。



​	（13）嵌套查询

​			(1 带有in的子查询

​				一个select-from-where 语句称为一个查询块，将一个查询块的where子句或having短语的条件中的查询称为嵌套查询。

	select * from student where sno in (select sno from sc where cno='2')

​					①不相关子查询 ：子查询的条件不依赖于父查询。

	select * from student where sdept in (select sdept from student where sname = '刘晨') -- 查询与刘晨一个系的学生


	select s1.sname from student s1,student s2 where s1.sdept = s2.sdept and s2.sname='刘晨'; -- 与上一条sql有相同的功能

​					②相关子查询：如果子查询的查询条件依赖于父查询。

​				(2 带有比较运算符的子查询

	select sno,cno from sc x where grade > (select avg(grade) from sc y where y.sno=x.sno) -- 找出每个学生超过他自己选修课程平均成绩的课程号

​				(3 带有any(some)或all的子查询

​					any 大于小于其中一个值就行。

	-- 查询非计算机科学系中比计算机科学系任意（所有）一个学生年龄小的学生姓名和年龄。
	select sname,sage from student where sage < any/all(select sage from student where sdept = 'CS') and sdept <> 'CS';

​				(4 带有exists的子查询

​						exists代表存在量词E，带有exists谓词的子查询不返回任何数据，只产生逻辑真或假。

​						no exists 与上面相反。

	select * from student where exists(select *from sc where sc.sno = student.sno and cno='1') --查询所有选修了1号课程的学生

​			使用存在量词not exist后，若内层查询结果为空，则外层的where子句返回真值，否则返回假值。

	select sname from student
		where no exists(
			select cname from course
				where no exists(
					select * from sc 
						where sno = student.sno and cno = student.cno))
	--查询选修了全部课程的学生
	--没有一个课程是他不选修的


	select distinct sno from sc scx
		where no exists(
			select * from sc scy
				where scy.sno='1' and no exists (
					select * from sc scz
						where scz.sno = scx.sno and scy.cno = scz.sno))
	-- 查询至少选修了学生1选修的全部课程的学生号码
	-- 不存在课程，学生1选修了y,而学生y没有选



​	（14）集合查询

- 并操作 union
- 交操作 intersect
- 差操作 except

	select * from student where dept = 'cs' 
	union 
	select * from student where sage > 19;
	-- 查询计算机系学生和年龄大于19的学生
	
	select * from student where dept = 'cs' 
	intersect
	select * from student where sage > 19;
	--查询计算机系并且年龄小于19的学生

​	（15）基于派生表的查询

​		可以出现在where、from子句中

	select * from sc ,(select sno,avg(grade) from sc group by sno) as avg_sc(ssno,ssgrade)
	where avg_sc.ssno = ...

​		注意别名写法

----------
#### 9.数据插入

​	插入子查询结果

​	对每一个系，求学生平均年龄，并把结果存入数据库。

	create table dept_age(
		sdept char(15),
		avg_age smallint
	)
	
	insert into dept_age(sdept,avg_age)
	select sdept,avg(sage) from student group by sdept;

----------
#### 10.修改数据

​	(1）修改一个元祖的值
​	update student set age = 20 where sno = '1';

​	(2) 修改多个元祖的值
​	update student set age = age + 1;

​	MySql 执行 DELETE FROM Table 时，报 

	Error Code: 1175. You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column To disable safe mode, toggle the option in Preferences - SQL Editor and reconnect. 错误，

这是因为 MySql 运行在 safe-updates模式下，该模式会导致非主键条件下无法执行update或者delete命令，执行命令如下命令


SET SQL_SAFE_UPDATES = 0;

修改下数据库模式，然后就可以继续执行 DELETE/UPDATE 了

如果想改会 safe-updates模式，执行如下命令即可

SET SQL_SAFE_UPDATES = 1;

(3)带子查询的修改语句

	update sc set grade = 0
		where sno in (select sno from student where sdept = 'cs')



----------
#### 11.删除数据

​	同样支持删除一个，删除多个，还有带子查询的删除。

----------

#### 12.空值处理

​	oracle 和mysql都可以用is null 啊。

----------
#### 13.创建视图

	create view student_ 
	as
	select sno,sname,sage from student ;

​	create view 语句的结果只是把视图的定义存入数据字典，并不执行其中的select语句，只是在对视图进行查询的时候，才按视图的定义从基本表中将数据查出。

​	原来也可以针对某个表对应的视图进行插入数据，在视图中插入，对应表中也会增添数据。

	create view is_test2 as select * from test2 where dept = 'cs' with check option;

​	加上了 with check option后，以后对该视图插入、修改、删除操作时，关系数据库管理系统会自动加上dept='cs' 的条件。就是插入的数据不是cs的就报错。

​	行列子集视图：从单个基本表导出的，并且只是去掉了基本表党的某些行，某些列，但保留了主码。

​	不是所有视图都可以更新。

14. 视图的作用：

- 简化用户的操作
- 以多种角度看待数据
- 对数据重构提供了一定的逻辑独立性
- 对机密数据提供保护
- 更清晰的表达查询
