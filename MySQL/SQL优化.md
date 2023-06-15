### 插入优化
* 批量插入
* 批量插入前取消事务自动提交
* 大批量插入用load

### 更新优化
* InnoDB行锁是针对索引加锁的，如果索引失效或者没有这个索引的话，就会从行锁升级成表锁

### 重构查询
* 切分查询：将一个大查询切分成多个小查询。每个查询的功能一样，但是每次只返回一部分数据。
* 分解多表查询：将多表查询的逻辑放在应用层做。好处如下
    * 减少锁竞争
    * 在应用层做关联，方便对数据库进行拆分
    * 减少冗余记录的查询


### 锁
* 全局锁：对整个数据库加锁，通常用于全库逻辑备份。
* 表级锁：为整张表加锁。表级锁分为表锁和元数据锁(meta data lock, MDL)。表锁是增删改查用到的锁。元数据锁负责对表做结构变更操作时的数据安全。
* 行级锁：

意向锁：意向锁是表锁，用于协调行锁和表锁的关系，支持多粒度的锁并存。当一个表中有行锁存在时，MySQL会自动为表添加意向锁。此时如果有事务想申请整个表的写锁，那么就不需要遍历每一行判断是否存在行锁，直接判断是否存在意向锁即可。

行锁：锁住一整行。
间隙锁：锁住一段范围内的索引记录。锁定的是区间，在这个区间内不允许被插入数据
临键锁：记录所和间隙锁的组合，即封锁区间，也封锁记录。


<!-- # 学生
insert into labdb_mock.accounts
(name,field,img_path,link,info,email,username,password)
select name,field,img_path,link,info,email,username,password from labdb.students where is_graduate="NOT_GRADUATED";

update labdb_mock.accounts set status="STUDENT" where username in (select username from labdb.students where is_graduate="NOT_GRADUATED");

insert into labdb_mock.students
(id, grade, title)
select a.id,s.grade,s.title from labdb_mock.accounts a inner join labdb.students s on a.username = s.username where s.is_graduate="NOT_GRADUATED";

# 毕业生
insert into labdb_mock.accounts
(name,field,img_path,link,info,email,username,password)
select name,field,img_path,link,info,email,username,password from labdb.students where is_graduate="GRADUATED";

update labdb_mock.accounts set status="GRADUATED" where username in (select username from labdb.students where is_graduate="GRADUATED");

insert into labdb_mock.graduated
(id, grade, title, unit, thesis_id)
select a.id,s.grade,s.title,s.unit,s.thesis_id from labdb_mock.accounts a inner join labdb.students s on a.username = s.username where s.is_graduate="GRADUATED";

# 老师
insert into labdb_mock.accounts
(name,field,img_path,link,info,email,username,password)
select name,field,img_path,link,info,email,username,password from labdb.teachers;

update labdb_mock.accounts set status="TEACHER" where username in (select username from labdb.teachers);

insert into labdb_mock.teachers
(id, title, recruit_info)
select a.id,s.title,s.recruit_info from labdb_mock.accounts a inner join labdb.teachers s on a.username = s.username; -->