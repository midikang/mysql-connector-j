# Reading mysql jdbc driver source code

09-30-16 - Version 5.1.40

## Generate api documentation to help code reading
```
javadoc -d docs -sourcepath /home/ubuntu/workspace/java/mysql-connector-j/src -subpackages com:org
```

### com.mysql.jdbc.Driver

The DriverManager will try to load as many drivers as it can find and then for any given connection request, it will ask each driver in turn to try to connect to the target URL.

It is strongly recommended that each Driver class should be small and standalone so that the Driver class can be loaded and queried without bringing in vast quantities of supporting code.

When a Driver class is loaded, it should create an instance of itself and register it with the DriverManager. This means that a user can load and register a driver by doing Class.forName("foo.bah.Driver")

```
class Driver extends NonRegisteringDriver implements java.sql.Driver {
    // Register ourselves with the DriverManager
    java.sql.DriverManager.registerDriver(new Driver());
    
}

```


### com.mysql.jdbc.NonRegisteringDriver#connect
```
    static {
        AbandonedConnectionCleanupThread referenceThread = new AbandonedConnectionCleanupThread();
        referenceThread.setDaemon(true);
        referenceThread.start();
    }
```

Entry method 
connect()
```
    if (StringUtils.startsWithIgnoreCase(url, LOADBALANCE_URL_PREFIX)) {
        return connectLoadBalanced(url, info);
    } else if (StringUtils.startsWithIgnoreCase(url, REPLICATION_URL_PREFIX)) {
        return connectReplicationConnection(url, info);
    }
    
    if (!"1".equals(props.getProperty(NUM_HOSTS_PROPERTY_KEY))) {
            return connectFailover(url, info);
    }
    
    Connection newConn = com.mysql.jdbc.ConnectionImpl.getInstance(host(props), port(props), props, database(props), url);
```

### com.mysql.jdbc.ConnectionImpl#getInstance
Entry method
getInstance()

A connection represents a session with a specific database. Within the context of a Connection, SQL statements are executed and results are returned.

If jdbc3 (or older), call ConnectionImpl(), otherwise Util.handleNewInstance(), to creates a connection to a MySQL Server.
```
        if (!Util.isJdbc4()) {
            return new ConnectionImpl(hostToConnectTo, portToConnectTo, info, databaseToConnectTo, url);
        }

        return (Connection) Util.handleNewInstance(JDBC_4_CONNECTION_CTOR,
                new Object[] { hostToConnectTo, Integer.valueOf(portToConnectTo), info, databaseToConnectTo, url }, null);
```
// Jdbc3 or older
ConnectionImpl()

// Creates an IO channel to the server
createNewIO(false);

connectOneTryOnly() => coreConnect()
connectWithRetries()

### Dive into connectOneTryOnly
Connect to the MySQL server and setup a stream connection.

Create io instance and doHandshake.
```
        this.io = new MysqlIO(newHost, newPort, mergedProps, getSocketFactoryClassName(), getProxy(), getSocketTimeout(),
                this.largeRowSizeThreshold.getValueAsInt());
        this.io.doHandshake(this.user, this.password, this.database);
```

### MysqlIO#MysqlIO()
```
        this.socketFactoryClassName = socketFactoryClassName;
        this.socketFactory = createSocketFactory();
        this.mysqlConnection = this.socketFactory.connect(this.host, this.port, props);
```

createSocketFactory()
```
            return (SocketFactory) (Class.forName(this.socketFactoryClassName).newInstance());
```

Where is socketFactoryClassName?
Inside ConnectionPropertiesImpl class, is StandardSocketFactory.
So the instance method connect of StandardSocketFactory will be call in new MysqlIO().
```

    private StringConnectionProperty socketFactoryClassName = new StringConnectionProperty("socketFactory", StandardSocketFactory.class.getName(),
            Messages.getString("ConnectionProperties.socketFactory"), "3.0.3", CONNECTION_AND_AUTH_CATEGORY, 4);
```

### com.mysql.jdbc.StandardSocketFactory#connect
Return java.net.Socket as connection

1. Create Socket instance.
2. Configures socket properties based on properties from the connection(tcpNoDelay, snd/rcv buf, traffic class, etc).
3. Create java.net.InetSocketAddress
```
    this.rawSocket = createSocket(props);

    configureSocket(this.rawSocket, props);

    InetSocketAddress sockAddr = new InetSocketAddress(possibleAddresses[i], this.port);
    // bind to the local port if not using the ephemeral port
    if (localSockAddr != null) {
        this.rawSocket.bind(localSockAddr);
    }

    this.rawSocket.connect(sockAddr, getRealTimeout(connectTimeout));
```


## Jdbc4 in Util.handleNewInstance

```
return (Connection) Util.handleNewInstance(JDBC_4_CONNECTION_CTOR,
                new Object[] { hostToConnectTo, Integer.valueOf(portToConnectTo), info, databaseToConnectTo, url }, null);
```

What is JDBC_4_CONNECTION_CTOR ?
ConnectionImpl static block
```
JDBC_4_CONNECTION_CTOR = Class.forName("com.mysql.jdbc.JDBC4Connection")
                        .getConstructor(new Class[] { String.class, Integer.TYPE, Properties.class, String.class, String.class });
```

check com.mysql.jdbc.JDBC4Connection
