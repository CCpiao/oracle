**一、创建表空间**
===============

1. 永久表空间的创建
-----------------------
```sql
SQL> create tablespace hostspace

  2  datafile 'E:\\CDU成都大学\\大三（2）\\oracle\\test6\\hostspace.dbf'

  3  size 50m

  4  autoextend on

  5  next 5m

  6  maxsize 100m;
```
2. 临时表空间的创建
-----------------------
```sql
SQL> create temporary tablespace hosttemp

  2  tempfile 'E:\\CDU成都大学\\大三（2）\\oracle\\test6\\hosttemp.dbf'

  3  size 10m

  4  autoextend on

  5  next 2m

  6  maxsize 20m;
```
3. 撤销表空间的创建
-----------------------
```sql
SQL> create undo tablespace hostundo

  2  datafile 'E:\\CDU成都大学\\大三（2）\\oracle\\test6\\hostundo.dbf'

  3  size 50m

  4  autoextend on

  5  next 5m

  6  maxsize 100m;
```
**二、修改表空间**
===========

1. 查看数据文件信息
------------------------
```sql
SQL> select tablespace_name,file_name,bytes
  2  from dba_data_files
   3  where tablespace_name='hostspace';
```
2. 修改数据文件大小
------------------------
```sql
SQL> alter database
  2  datafile 'E:\CDU成都大学\大三（2）\oracle\test6\hostspace.DBF'
  3  resize 40m;
```
3. 添加新的数据文件
-----------------------
```sql
SQL> alter tablespace hostspace
  2  add datafile
  3  'E:\CDU成都大学\大三（2）\oracle\test6\hostspace1.DBF'
  4  size 10m
  5  autoextend on next 5m maxsize 40m;
```
4. 删除新建的数据文件
-------------------------
```sql
SQL> alter tablespace hostspace
  2  drop datafile 'E:\CDU成都大学\大三（2）\oracle\test6\hostspace1.DBF';
```
**三、表操作**
=========

1. 表的创建
-------------------

### (1) 用户类表
```sql
SQL> create table user_type(

  2  typeid number(10) primary key,

  3  typename varchar2(10) not null

  4  )tablespace hostspace;
```
### (2) 用户信息表
```sql
SQL> create table users(

  2  userid varchar2(10) primary key,

  3  uname varchar2(10) not null,

  4  pwd varchar2(20) not null,

  5  typeid number(10) not null,

  6  constraint users_type foreign key (typeid)

  7  references type(typeid)

  8  )tablespace hostspace;
```
### (3) 主播类型信息表
```sql
SQL> create table type(

2  typeid number(10) primary key,

3  typename varchar(20) not null)

4  tablespace hostspace;
```
### (4) 平台信息表
```sql
SQL> create table platform(

2  pid number(10) primary key,

3  pname varchar2(10) not null,

4  typeid number(10) not null,

5  constraint platform_type foreign key(typeid)

6  ) tablespace hostspace;
```
### (5) 主播信息表
```sql
SQL> create table host(

  2  hostid number(10) primary key,

  3  hname varchar2(4) not null,

  4  sex char(2) not null

  5  check (sex in('男','女')),

  6  pid number(10) not null,

  7  typeid number(10) not null,

  8  constraint host_platform foreign key(pid)

  9  references platform(pid),

 10  constraint host_type foreign key(typeid)

 11  references type(typeid)

 12  )tablespace hostspace;
```
### (6) 运营信息表
```sql
SQL> create table operate(

  2  oid number(10) primary key,

  3  oname varchar2(4) not null,

  4  sex char(2) not null

  5  check (sex in('男','女')),

  6  typeid number(10) not null,

  7  constraint operate_type foreign key(typeid)references type(typeid)

8  )tablespace hostspace;
```
### (7) 公会信息表
```sql
SQL> create table sociaty(

  2  sid number(10) primary key,

  3  sname varchar(20) unique not null)tablespace hostspace;
```
### (8) 主播刷票表
```sql
SQL> create table ticket(

  2  hostid number(10) primary key,

  3  hname varchar2(10) not null,

  4  sid number(10) not null,

  5  sname varchar2(20) not null,

  6  ticket number(3) not null,

  7  constraint ticket_host foreign key(hostid)references host(hostid),

  8  constraint ticket_sociaty foreign key(sid)references sociaty(sid)

  9  )tablespace hostspace;
```
2. 索引
-----------------

*   在platform表中的pname列上创建pname_index的索引

SQL> create index pname_index

  2  on platform(pname)

  3  tablespace hostspace;

*   打开platform表中platform列上的pid_index索引的监控状态
```sql
SQL> alter index pname_index monitoring usage;
```
通过数字字典v$object_usage可以查看哪些索引正在被监控
```sql
SQL> column index_name format a15;
SQL> column table_name format a15;
SQL> select index_name,table_name,monitoring,
  2  used,start_monitoring,end_monitoring
  3  from v$object_usage;
```
3. 视图
-----------------
*   创建基于platform表和magor表的视图V1，在该视图的子查询中检索平台信息的同时显示其所在主播类型名称。
```sql
SQL> create view v1

  2  as

  3  select c.pid,c.pname,m.typename

  4  from platform c left join type m

  5  on c.typeid=m.typeid;
```
*   创建基于ticket表的视图V2,查询刷票不及格主播的信息
```sql
SQL> create view v2

  2  as

  3  select hostid,hname,sname,ticket from ticket where ticket<60;
```
4. 使用序列
-------------------

创建一个名为host_seq的序列
```sql
SQL> create sequence host_seq

  2  start with 1

  3  increment by 1

  4  nocache

  5  nocycle

  6  order;
```
5. 插入50000条数据
-----------------------------------------

host表数据插入50000
```sql
declare

i int;

hostid number(20);

hname VARCHAR2(100);

sex VARCHAR2(100);

pid number(20);

typeid number(20);

begin

i:=1;

while i<=50000

loop

hostid:=i;

hname:= ''|| i;

sex := '男'|| i;

pid := '123'|| i;

typeid := '123'|| i;

insert into train_( hostid,hname,sex,pid,typeid) values (hostid,hname,sex,pid,typeid);

i:=i+1;

end loop;

commit;

end;

/
```
ticket表数据插入50000
```sql
declare

i int;

hostid number(20);

hname VARCHAR2(100);

sid VARCHAR2(100);

sname number(20);

ticket number(20)

begin

i:=1;

while i<=50000

loop

hostid:=i;

hname:= ''|| i;

sex := '男'|| i;

pid := '123'|| i;

typeid := '123'|| i;

insert into train_( hostid,hname,sid,sname,ticket) values( hostid,hname,sid,sname,ticket);

i:=i+1;

end loop;

commit;

end;

/
```
**四、查询**
========

1. 基础查询
-------------------

统计各平台女性人数
```sql
SQL> select pid,count(*) from host

  2  where sex='女'

  3  group by typeid

  4  having count(*)>0

  5  order by count(*) desc;
```
2. 子查询和高级查询
---------------------------------------

*   查询平均刷票大于80元的主播的主播编号和平均刷票
```sql
SQL> select hostid, avg(ticket) from ticket 

2  group by hostid 

3  having avg(ticket)>80;
```
*   查询“1”公会比“2”公会刷票高的所有主播的主播编号
```sql
SQL> select a.hostid

  2  from (select * from ticket t where t.sid = 1) a,

  3      (select * from ticket s where t.sid = 2) b

  4  where a.hostid = b.hostid

  5  and a.ticket > b.ticket;
```
*   查询姓“刘”的主播名单
```sql
SQL> select * from host where hname like '刘%';
```
*   查询平台信息的同时显示其所在主播类型名称。
```sql
SQL> select p.pid,p.pname,m.typename

  2  from platform p left join type m

  3  on p .typeid=m.typeid;
```
*   查询所有主播的公会信息
```sql
SQL> select h.hostid, h.hname hostdenoname,

  2  s.sid, s.sname sociatyname

  3  from host h, ticket t, sociaty s

  4  where t.hostid=t.hostid and t.sid=s.sid;
```
**五、PL/SQL语句**
==============

1. 查询
-----------------

输出公会信息表中的公会编号为1的公会名：
```sql
SQL> set serveroutput on;

SQL> declare

  2  id constant number(10):=1;

  3  name varchar2(30);

  4  begin

  5  select sname into name

  6  from sociaty where sid=id;

  7  dbms_output.put_line(id||name);

  8  end;

  9  /
```

2. 判断
-----------------

查询所有主播的刷票是否有小于1元，如有就触发异常并输出。
```sql
SQL> declare

  2  cursor c1 is select hname from ticket where ticket<1;

  3  one ticket.hname%type;

  4  e1 exception;

  5  begin

  6  open c1;

  7  fetch c1 into one;

  8  if c1%found then raise e1;

  9  end if;

 10  exception

 11  when e1 then

 12  dbms_output.put_line(one||'不达标');

 13  close c1;

 14  end;

 15  /
```
**六、创建存储过程**
============

创建get_ticket_info，采取直接在存储过程中使用DBMS_OUTPUT.PUT_LINE过程输出相关内容。
```sql
SQL> create or replace procedure get_ticket_info

  2  (s_no number)

  3  as

  4  s_name varchar2(10);

  5  c_no number;

  6  c_name varchar2(20);

  7  s_ticket number(3);

  8  begin

  9  select hname,sid,sname,ticket

 10  into s\_name,c\_no,c\_name,s\_ticket

 11  from ticket where hostid=s_no;

 12  DBMS\_OUTPUT.PUT\_LINE('主播姓名：'||s_name);

 13  DBMS\_OUTPUT.PUT\_LINE('公会编号：'||c_no);

 14  DBMS\_OUTPUT.PUT\_LINE('公会名称：'||c_name);

 15  DBMS\_OUTPUT.PUT\_LINE('票数：'||s_ticket);

 16  end get\_ticket\_info;

 17  /
```
调用get\_ticket\_info存储过程。例如获取hostid为1401的主播的刷票信息，如下：
```sql
SQL> set serveroutput on

SQL> exec get\_ticket\_info(1402);
```
**七、函数**
========

1. 创建一个函数get_hname
--------------------------------------

创建一个函数get_hname，该函数实现按hostid获取hname，函数创建如下：
```sql
SQL> create function get_hname(host_num number)

  2  return varchar2 as

  3  host_name host.hname%type;

  4  begin

  5  select hname into host_name from host where hostid=host_num;

  6  return host_name;

  7  end get_hname;

  8  /
```
函数已创建。

因为函数是具有返回值的，所以它类似于一个表达式，调用函数可以直接使用select语句，如下：
```sql
SQL> select get_hname(1) from dual;
```

**2. ******创建一个函数re_********host********_info****
-------------------------------------------------

创建一个函数re_host_info,以平台号为参数，返回各个平台总平均票数。
```sql
SQL> create or replace function re_host_info(cl_id number) return varchar2 is

  2    v_result varchar2(100);

  3    cursor cur_platform is

  4     select a.pname as pname,

  5             round(avg(ticket), 2) as avg_score

  6       from platform a

  7       inner join host b

  8          on a.pid = b.pid

  9       inner join ticket c

 10          on c.hostid = b.hostid

 11       where a.pid = cl_id

 12       group by a.pname;

 13        c_row cur_platform%rowtype;

 14  begin

 15    for c_row in cur_platform loop

 16      dbms_output.put_line('平台：' || c_row.pname || '总平均票数为：' || c_row.avg_score);

 17    end loop;

 18    return v_result;

 19  end;

 20   /
```
调用函数：
```sql
SQL> declare

  2  t varchar2(50);

  3  v_number number(10);

  4  cursor cur is select pid from platform;

  5  begin

  6    open cur;

  7    fetch cur into v_number;

  8    while cur%found loop

  9      t:=re_host_info(v_number);

 10      fetch cur into v_number;

 11      end loop;

 12      close cur;

 13  end;

 14  /
```
**八、备份方案**
==========

1.  为目录创建一个单独的表空间
```sql
SQL>Create tablespace tools datafile ‘fielname’ size 50m;
```
2.  创建RMAN用户
```sql
SQL>Create user RMAN identified by RMAN default tablespace tools temporary tablespace temp;
```
3.  给RMAN授予权限
```sql
SQL>Grant connect , resource , recovery\_catalog\_owner to rman;
```
4.  打开RMAN
```sql
$>RMAN
```
5.  连接数据库
```sql
RMAN>connect catalog rman/rman
```
6.  创建恢复目录
```sql
RMAN>Create catalog tablespace tools
```
注册目标数据库，恢复目录创建成功后，就可以注册目标数据库了，目标数据库就是需要备份的数据库，一个恢复目录可以注册多个目标数据库，注册目标数据库的命令为：
```sql
$>RMAN target internal/password catalog rman/rman@rcdb;

RMAN>Register database;
```
数据库注册完成,就可以用RMAN来进行备份了，更多命令请参考ORACLE联机手册或《ORACLE8i备份与恢复手册》。

注销数据库不是简单的在RMAN提示下反注册就可以了，需要运行一个程序包，过程如下：

（1）. 连接目标数据库，获得目标数据库ID
```sql
$\> RMAN target internal/password catalog rman/rman@rcdb;
```
（2）. 查询恢复目录，得到更详细的信息
```sql
SQL> SELECT db\_key, db\_id FROM db WHERE db_id = 1231209694;
```
（3）.运行过程dbms_rcvcat.unregisterdatabase注销数据库，如
```sql
SQL> EXECUTE dbms_rcvcat.unregisterdatabase(1 , 1237603294)
```
（4）采用RMAN进行备份

1).备份整个数据库
```sql
 backup full tag ‘basicdb’ format ‘/bak/oradata/full_%u_%s_%p’ database;
```
2).备份一个表空间
```sql
 backup tag ‘tsuser’ format ‘/bak/oradata/tsuser_%u_%s_%p’ tablespace users;
```
3).备份归档日志
```sql
 backup tag ‘alog’ format ‘/bak/archivebak/arcbak_%u_%s_%p’ archivelog all delete input;
```