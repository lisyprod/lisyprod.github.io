## 一. 架构
![](../files/Pasted%20image%2020221102062503.png)
## 二. 并发控制

### 锁的类型
- **读锁**(**共享锁**)
- **写锁**(**排他锁**)
### 锁的粒度
- **表锁**
- **行锁**
## 三. 事务

### 事务的特性(ACID)
- **原子性(Atomicity)**:要么全部要么全不做
- **一致性(Consistency)**:事务执行后的结果跟分开一条一条执行的结果是一致的
- **隔离性(Isolation)**:事务相关隔离
- **持久性(Durability)**:持久到硬盘上,即使数据库奔溃了
## 隔离级别
- **读未提交**(**脏读**):事务可以看到其他事务没有提交的修改.可能会导致两个事务执行的结果不一致,有脏读的问题.
- **读已提交**(**不可重复读**):事务之间没有提交的数据互相看不见.事务提交后的数据可以看到,同一个事务中,两次执行相同的语句,可能会看到不通的结果.
- **可重复读**:同一个事务中,读取相同行数据的结果是一致的.但是有幻行问题,可能导致幻读(某个事务读取某个范围内的数据时,其他事务插入记录,当事务再次读取该范围时,产生幻行).mysql8 innodb解决了这个问题,
- **串行化**:强制事务按顺序执行
### 死锁

- 事务1
```
START TRANSACTION;
update tab1 set ab=12 where stock_id=4 and date = '2022-12-13';
update tab1 set ab=15 where stock_id=3 and date = '2022-11-16';
COMMIT;
```
- 事务2
```
START TRANSACTION;
update tab1 set ab=0 where stock_id=3 and date = '2022-11-16';
update tab1 set ab=11 where stock_id=4 and date = '2022-12-13';
COMMIT;
```

### mysql 事务
- 默认情况下,单个update,insert,delete 会隐式包含在事务中并在执行成功后自动提交
- 可以通过禁用自动提交(),执行一系列语句在执行完成后commit或者rollback
- SET autocommit=0/SET autocommit=1,SET autocommit=ON/SET autocommit=OFF(一旦使用SET AUTOCOMMIT=0 禁止自动提交，则在这个数据库内部的所有事务都不会自动提交，除非你手动的为每一个事务执行了commit或者rollback语句；而start transaction和begin/commit只能控制某一个事务)
- start transaction和begin/commit(存在一种set autocommit = 1/0 但是 对于某一个sql语句使用了 begin/commit的原子性操作，那么mysql会优先使用begin/commit命令控制被这组命令修饰的事务)
- 至于start transaction 和 begin的区别:两者的作用一摸一样，只是在begin可能成为关键字的时候，使用start transaction 可以 避免这种情况，start transaction或者begin开启一个事务，然后使用commit提交事务或者ROLLBACK回滚事务
- 正常来说，当我们开启一个事务之后，需要 commit 或者 rollback 来结束一个事务的，但是有时候，一些操作会自动帮我们提交事务.
  - 所有的 DDL 语句都会导致事务隐式提交
  - DCL语句:DCL 其实就是 Data Control Language,中文译作数据控制语言，我们日常授权或者回收数据库上的权限所使用的 GRANT、REVOKE 等
  - 新事务开启:一个事务还没提交，结果你又开启了一个新的事务，那么此时前一个事务也会隐式提交
  - 各种锁操作:给表上锁、解锁也会导致事务隐式提交.除了表锁，一些全局锁如 FTWRL 也会导致事务的隐式提交
  - 从机上执行的一些操作如 `start slave`、`stop slave`、`reset slave` 以及 `change master to` 等语句也会隐式提交事务
  - 其他的以下操作如刷新权限（flush privileges）、优化表（optimize table）、修复表（repair table）等操作，也会导致事务的隐式提交
  - 最佳实践:事务里只写增删改查（INSERT/DELETE/UPDATE/SELECT）