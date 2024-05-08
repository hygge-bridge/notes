# 25 连接查询

创建表：

```mysql
create table student(
	uid int unsigned primary key not null auto_increment,
    name varchar(50) not null,
    age int unsigned not null,
    sex enum('m','w') not null
);

create table course(
	cid int unsigned primary key not null auto_increment,
    cname varchar(50) not null,
    credit int unsigned not null
);

create table exame(
	uid int unsigned not null,
    cid int unsigned not null,
    time date not null,
    score float not null,
    primary key(uid, cid)
);

insert into student(name, age, sex) values
("tom", 20, 'm'), ("rose", 30, 'w'), ("jack", 18, 'm');

insert into course(cname, credit) values
("c++", 5), ("python", 3), ("shell", 2), ("java", 5);

insert into exame(uid, cid, time, score) values
(1, 1, "2002-10-1", 99), (1, 2, "2002-10-2", 80),
(2, 1, "2002-10-1", 44), (2, 3, "2002-10-2", 77),
(3, 1, "2002-10-1", 22), (3, 2, "2002-10-2", 55), 
(3, 3, "2002-10-3", 50);
```

查询：

```mysql
select s.name, s.age, s.sex, e.score from student s
inner join exame e on e.uid = s.uid where s.uid = 1;

select s.name, s.age, s.sex, c.cname, c.credit, e.score from exame e
inner join student s on s.uid = e.uid
inner join course c on c.cid = e.cid
where s.uid = 1;

select s.name, s.age, s.sex, c.cname, c.credit, e.score from exame e
inner join student s on s.uid = e.uid
inner join course c on c.cid = e.cid
where e.score > 90;

select c.cname, count(*) cnt from exame e
inner join course c on c.cid=e.cid
where c.cname="python"
group by c.cid
order by cnt;

select c.cname, max(e.score) from exame e 
inner join student s on s.uid=e.uid 
inner join course c on c.cid=e.cid 
group by c.cid;
```

