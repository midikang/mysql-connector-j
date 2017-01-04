# TestCase ResultSetTest

ResultSetTest extends BaseTestCase 


## BaseTestCase#setUp

```
this.conn = DriverManager.getConnection(dbUrl, props);

// ConnectionImpl#createStatement
this.stmt = this.conn.createStatement();
this.rs = this.stmt.executeQuery("SELECT VERSION()");

```

## ConnectionImpl#createStatement
```
    private static final int DEFAULT_RESULT_SET_TYPE = ResultSet.TYPE_FORWARD_ONLY;

    private static final int DEFAULT_RESULT_SET_CONCURRENCY = ResultSet.CONCUR_READ_ONLY;
    
    StatementImpl stmt = new StatementImpl(getMultiHostSafeProxy(), this.database);

``` 

## StatementImpl#StatementImpl
```
    this.connection.registerStatement(this);
```

```
    /**
     * An array of currently open statements.
     * Copy-on-write used here to avoid ConcurrentModificationException when statements unregister themselves while we iterate over the list.
     */
    private final CopyOnWriteArrayList<Statement> openStatements = new CopyOnWriteArrayList<Statement>();

    this.openStatements.addIfAbsent(stmt);

```


# executeQuery

```
    MySQLConnection locallyScopedConn = this.connection;
    statementBegins();

    this.results = locallyScopedConn.execSQL(this, sql, this.maxRows, null, this.resultSetType, this.resultSetConcurrency,
        createStreamingResultSet(), this.currentCatalog, cachedFields);

```

## ConnectionImpl#execSQL

```
    if (packet == null) {
        ...
        return this.io.sqlQueryDirect(callingStatement, sql, encoding, null, maxRows, resultSetType, resultSetConcurrency, streamResults, catalog,
                cachedMetadata);
    }

    return this.io.sqlQueryDirect(callingStatement, null, null, packet, maxRows, resultSetType, resultSetConcurrency, streamResults, catalog,
            cachedMetadata);
```

## MysqlIO#sqlQueryDirect
```
    // Send query command and sql query string
    Buffer resultPacket = sendCommand(MysqlDefs.QUERY, null, queryPacket, false, null, 0);
    
    // Checks for errors in the reply packet, and if none, returns the reply
    // packet, ready for reading
    checkErrorPacket()
    
    ResultSetInternalMethods rs = readAllResults(callingStatement, maxRows, resultSetType, resultSetConcurrency, streamResults, catalog, resultPacket,
        false, -1L, cachedMetadata);

    
```

