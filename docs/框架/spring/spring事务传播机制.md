### Spring 事务传播机制

**类型**

- PROPAGATION_REQUIRED（默认）：如果A有事务则B加入A事务，如果A没有事务则新B建一个事务;

- PROPAGATION_NEW:B总是开启一个新的事务，如果A有事务则将A事务挂起先执行B事务;

- PROPAGATION_NESTED:RUGU :如果A没事务则新建一个事务，如果A有事务则把B的事务当成A的一个子事务（A事务rolback，commit影响B，B事务rolback，commit不影响A）;

- PROPAGATION_SUPPORTS:如果A没事务，那就按普通方法执行，如果有A事务则用A的事务(B本身不具备事务)；

- PROPAGATION_NOT_SUPPORTED:B总是非事务地执行，如果A有事务则把A事务挂起，自己还是以普通方法执行(B本身不具备事务）;

- PROPAGATION_NEVER:如果A没事务，那就按普通方法执行，如果A有事务则抛出异常（(B本身不具备事务);

- PROPAGATION_MANDATORY:如果A没事务就抛异常，如果A有事务则使用A的事务(B本身不具备事务);

**原理**

传递的其实是一个Connection ，spring事务说到底其实底层还是对jdbc 的封装，回顾jdbc的处理流程，你会发现同一事务是和一个Connection绑定的。

spring源码也可以看出：

org.springframework.transaction.support.AbstractPlatformTransactionManager#handleExistingTransaction 参见该方法：

跟进这个条件if (definition.getPropagationBehavior() == 4)

suspendedResources = this.suspend(transaction);

```java
protected final AbstractPlatformTransactionManager.SuspendedResourcesHolder suspend(Object transaction) throws TransactionException {
    if (TransactionSynchronizationManager.isSynchronizationActive()) {
        List suspendedSynchronizations = this.doSuspendSynchronization();

        try {
            Object suspendedResources = null;
            if (transaction != null) {
                suspendedResources = this.doSuspend(transaction);//中断处理
            }

            .............
        }
        // 继续跟进：
        org.springframework.jdbc.datasource.DataSourceTransactionManager#doSuspend
            protected Object doSuspend(Object transaction) {
            DataSourceTransactionManager.DataSourceTransactionObject txObject = (DataSourceTransactionManager.DataSourceTransactionObject)transaction;
            txObject.setConnectionHolder((ConnectionHolder)null);
            ConnectionHolder conHolder = (ConnectionHolder)TransactionSynchronizationManager.unbindResource(this.dataSource);
            return conHolder;
        }
        // 此处应该有些眉目了
        // 在看 ConnectionHolder
        // 继续跟进
        org.springframework.transaction.support.TransactionSynchronizationManager#unbindResource
            // 顺便参考下其他方法 发现都是基于Connection进行操作的
            protected void doResume(Object transaction, Object suspendedResources) {
            ConnectionHolder conHolder = (ConnectionHolder)suspendedResources;
            TransactionSynchronizationManager.bindResource(this.dataSource, conHolder);
        }

        protected void doCommit(DefaultTransactionStatus status) {
            DataSourceTransactionManager.DataSourceTransactionObject txObject = (DataSourceTransactionManager.DataSourceTransactionObject)status.getTransaction();
            Connection con = txObject.getConnectionHolder().getConnection();
            if (status.isDebug()) {
                this.logger.debug("Committing JDBC transaction on Connection [" + con + "]");
            }

            try {
                con.commit();
            } catch (SQLException var5) {
                throw new TransactionSystemException("Could not commit JDBC transaction", var5);
            }
        }

        protected void doRollback(DefaultTransactionStatus status) {
            DataSourceTransactionManager.DataSourceTransactionObject txObject = (DataSourceTransactionManager.DataSourceTransactionObject)status.getTransaction();
            Connection con = txObject.getConnectionHolder().getConnection();
            if (status.isDebug()) {
                this.logger.debug("Rolling back JDBC transaction on Connection [" + con + "]");
            }

            try {
                con.rollback();
            } catch (SQLException var5) {
                throw new TransactionSystemException("Could not roll back JDBC transaction", var5);
            }
        }
```

