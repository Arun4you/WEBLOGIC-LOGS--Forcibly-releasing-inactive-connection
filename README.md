# WEBLOGIC-LOGS--Forcibly-releasing-inactive-connection
The issue here is that the JDBC connections are not closed in the application.

To enable this feature, go to Admin Console -> JDBC Data Source: Configuration: Connection Pool, and set "Inactive Connection Timeout" greater than 0.

Inactive Connection Timeout:	1200s

WebLogic Server will reclaim leaked connections. There are two kinds of leaked cases. One is when the customer's application does not return a connection to the data source connection pool by invoking javax.sql.Connection.close(), leaving it orphaned. The other is when the customer's application becomes stuck on a long-running SQL query and the connection has been used for the specified number of seconds.

Once the connection released forcibly by WebLogic Server, a BEA-001153: Forcibly releasing inactive connection message is written to the server log.

<BEA-001153> <Forcibly releasing inactive connection "weblogic.jdbc.wrapper.PoolConnection_weblogic_jdbc_sqlserverbase_ddah@41c3" back into the connection pool "BTPx02", currently reserved by: java.lang.Exception
  at weblogic.jdbc.common.internal.ConnectionEnv.setup(ConnectionEnv.java:318)
  at weblogic.common.resourcepool.ResourcePoolImpl.reserveResource(ResourcePoolImpl.java:344)
  at weblogic.common.resourcepool.ResourcePoolImpl.reserveResource(ResourcePoolImpl.java:322)
  at weblogic.jdbc.common.internal.ConnectionPool.reserve(ConnectionPool.java:438)
  at weblogic.jdbc.common.internal.ConnectionPool.reserve(ConnectionPool.java:317)
  at weblogic.jdbc.common.internal.ConnectionPoolManager.reserve(ConnectionPoolManager.java:93)
  at weblogic.jdbc.common.internal.RmiDataSource.getPoolConnection(RmiDataSource.java:342)
  
  The issue here is that the JDBC connections are not closed and returned to the connection pool as they should be. Eventually, the JDBC connections time out and are closed forcibly, and when that occurs, this message is written to the logs.
  
  The recommendation is always making sure that your application code explicitly closes all JDBC objects. See here for an example. It is possible to wait for the timeout to close these objects, but that will cause this message in your logs, and more importantly, it will cause significant performance problems because there are fewer connections available to do the work your application needs.

It is also possible to disable the inactive connection timeout by setting "Inactive connection timeout" to 0. See Automatically Recovering Leaked Connections for more details. Please note that this feature is disabled by default (that is, 0 is the default value for "Inactive connection timeout").

NOTE: Disabling the timeout will remove the "forcibly releasing connection" messages from your log, but will also mean that these leaked connections are not returned to your pool. These will cause performance problems, and eventually your system will run out of connections entirely, causing the entire system to hang. This is therefore NOT A SUBSTITUTE for resolving the leaked connections by ensuring that your application properly closes all JDBC objects.
