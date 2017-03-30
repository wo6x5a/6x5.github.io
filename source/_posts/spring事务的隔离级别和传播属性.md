---
title: spring事务的隔离级别和传播属性
date: 2017-03-27 16:08:25
tags: [java,spring,事务]
categories: 技术
---
## 前言

对于事务的理解,在日常的开发过程中,相对来说还是比较重要的.不同的业务功能,所需要的事务传播属性以及隔离级别都是不一样的,当然,大多数情况下,我们使用默认的配置就行了,但是,某些特殊情境,我们就需要特殊配置.这里我就大概来说一说事务.

- 事务的传播属性:
```java

/**

* Support a current transaction, create a new one if none exists.

* Analogous to EJB transaction attribute of the same name.

* <p>This is the default setting of a transaction annotation.

*/

// 支持当前事务,如果没有,创建一个新事务.(默认属性,也是最常用的)
REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),


/**

* Support a current transaction, execute non-transactionally if none exists.

* Analogous to EJB transaction attribute of the same name.

* <p>Note: For transaction managers with transaction synchronization,

* PROPAGATION_SUPPORTS is slightly different from no transaction at all,

* as it defines a transaction scope that synchronization will apply for.

* As a consequence, the same resources (JDBC Connection, Hibernate Session, etc)

* will be shared for the entire specified scope. Note that this depends on

* the actual synchronization configuration of the transaction manager.

* @see  org.springframework.transaction.support.AbstractPlatformTransactionManager#setTransactionSynchronization

*/

//支持当前事务,如果没有事务,就以非事务状态执行
SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),


/**

* Support a current transaction, throw an exception if none exists.

* Analogous to EJB transaction attribute of the same name.

*/

//支持当前事务,如果没有事务,抛出异常
MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),


/**

* Create a new transaction, suspend the current transaction if one exists.

* Analogous to EJB transaction attribute of the same name.

* <p>Note: Actual transaction suspension will not work on out-of-the-box

* on all transaction managers. This in particular applies to JtaTransactionManager,

* which requires the {@code  javax.transaction.TransactionManager} to be

* made available it to it (which is server-specific in standard J2EE).

* @see  org.springframework.transaction.jta.JtaTransactionManager#setTransactionManager

*/

//新建事务,若当前存在事务,则挂起当前事务
REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),


/**

* Execute non-transactionally, suspend the current transaction if one exists.

* Analogous to EJB transaction attribute of the same name.

* <p>Note: Actual transaction suspension will not work on out-of-the-box

* on all transaction managers. This in particular applies to JtaTransactionManager,

* which requires the {@code  javax.transaction.TransactionManager} to be

* made available it to it (which is server-specific in standard J2EE).

* @see  org.springframework.transaction.jta.JtaTransactionManager#setTransactionManager

*/

//以非事务方式执行.若当前存在事务,则挂起当前事务
NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),



/**

* Execute non-transactionally, throw an exception if a transaction exists.

* Analogous to EJB transaction attribute of the same name.

*/

//以非事务方式执行.若当前存在事务,则跑出异常
NEVER(TransactionDefinition.PROPAGATION_NEVER),

/**

* Execute within a nested transaction if a current transaction exists,

* behave like PROPAGATION_REQUIRED else. There is no analogous feature in EJB.

* <p>Note: Actual creation of a nested transaction will only work on specific

* transaction managers. Out of the box, this only applies to the JDBC

* DataSourceTransactionManager when working on a JDBC 3.0 driver.

* Some JTA providers might support nested transactions as well.

* @see org.springframework.jdbc.datasource.DataSourceTransactionManager

*/

//执行嵌套事务,如果当前存在事务
NESTED(TransactionDefinition.PROPAGATION_NESTED);
```
- 事务的隔离级别:

```java
/**

* Use the default isolation level of the underlying datastore.

* All other levels correspond to the JDBC isolation levels.

* @see java.sql.Connection

*/

//使用数据库默认的隔离级别
DEFAULT(TransactionDefinition.ISOLATION_DEFAULT),


/**

* A constant indicating that dirty reads, non-repeatable reads and phantom reads

* can occur. This level allows a row changed by one transaction to be read by

* another transaction before any changes in that row have been committed

* (a "dirty read"). If any of the changes are rolled back, the second

* transaction will have retrieved an invalid row.

* @see java.sql.Connection#TRANSACTION_READ_UNCOMMITTED

*/

//读取未提交数据(级别最低,允许事务读取另一个事务未提交的数据)
READ_UNCOMMITTED(TransactionDefinition.ISOLATION_READ_UNCOMMITTED),


/**

* A constant indicating that dirty reads are prevented; non-repeatable reads

* and phantom reads can occur. This level only prohibits a transaction

* from reading a row with uncommitted changes in it.

* @see java.sql.Connection#TRANSACTION_READ_COMMITTED

*/

//读取提交数据(允许事务读取另一个事务提交的数据)
READ_COMMITTED(TransactionDefinition.ISOLATION_READ_COMMITTED),



/**

* A constant indicating that dirty reads and non-repeatable reads are

* prevented; phantom reads can occur. This level prohibits a transaction

* from reading a row with uncommitted changes in it, and it also prohibits

* the situation where one transaction reads a row, a second transaction

* alters the row, and the first transaction rereads the row, getting

* different values the second time (a "non-repeatable read").

* @see java.sql.Connection#TRANSACTION_REPEATABLE_READ

*/


//可重复读
REPEATABLE_READ(TransactionDefinition.ISOLATION_REPEATABLE_READ),


/**

* A constant indicating that dirty reads, non-repeatable reads and phantom

* reads are prevented. This level includes the prohibitions in

* {@code ISOLATION_REPEATABLE_READ} and further prohibits the situation

* where one transaction reads all rows that satisfy a {@code WHERE}

* condition, a second transaction inserts a row that satisfies that

* {@code WHERE} condition, and the first transaction rereads for the

* same condition, retrieving the additional "phantom" row in the second read.

* @see java.sql.Connection#TRANSACTION_SERIALIZABLE

*/


//顺序执行读取(级别最高,最可靠)
SERIALIZABLE(TransactionDefinition.ISOLATION_SERIALIZABLE);
```
下面整理一张表.

|       |丢失更新|	脏读 |非重复读| 覆盖更新| 幻读  |
| :--------:  | :-----:  | :----:  | :----:  | :----:  | :----: |
|未提交读|   N|	Y|	Y|	Y|	Y|
|已提交读|   N|	N|	Y|  Y|	Y|
|可重复读|   N|	N|	N|	N|	Y|
|串行化(顺序读)|	N|	N|	N|	N|	N|


- 名词解释


**未提交读**:允许读取未提交数据,如果一个事务已经开始写数据，则另外一个事务则不允许同时进行写操作，但允许其他事务读此行数据。

**已提交读**:当一个事务开始了,则只有等这个事务提交结束了,才允许其他事务读,

**可重复读**: 读取数据的事务将会禁止写事务（但允许读事务），写事务则禁止任何其他事务

**串行化读**:按照顺序,一个事务一个事务的执行.



**丢失更新**:撤销一个事务时，把其他事务已提交的更新数据覆盖（A和B事务并发执行，A事务执行更新后，提交；B事务在A事务更新后，B事务结束前也做了对该行数据的更新操作，然后回滚，则两次更新操作都丢失了）。

**脏读**:一个事务读到另一个事务未提交的更新数据（事务T1将某一值修改，然后事务T2读取该值，此后T1因为某种原因撤销对该值的修改，这就导致了T2所读取到的数据是无效的）。

**不可重复读**:一个事务读到另一个事务已提交的更新数据（一个事务范围内两个相同的查询却返回了不同数据。这是由于查询时系统中其他事务修改的提交而引起的。比如事务T1读取某一数据，事务T2读取并修改了该数据，T1为了对读取值进行检验而再次读取该数据，便得到了不同的结果）。

**覆盖更新**:这是不可重复读中的特例，一个事务覆盖另一个事务已提交的更新数据（即A事务更新数据，然后B事务更新该数据，A事务查询发现自己更新的数据变了）。

**虚读（幻读）**:一个事务读到另一个事务已提交的新插入的数据（A和B事务并发执行，A事务查询数据，B事务插入或者删除数据，A事务再次查询发现结果集中有以前没有的数据或者以前有的数据消失了）。



oracle 支持**Read Committed**和 **Serializable**和**oracle特有的Read-only** 这三种事务隔离级别,**默认Read Committed**

mysql支持**Read Uncommitted, Read Committed, Repeatable Read,Serializable, 默认Repeatable Read**