# Reading mysql jdbc driver source code

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


### NonRegisteringDriver
