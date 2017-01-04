# Handshake

## MysqlIO#doHandshake
Initialize communications with the MySQL server. Handles logging on, and
handling initial connection errors.

```
        //
        // switch to pluggable authentication if available
        //
        if ((this.serverCapabilities & CLIENT_PLUGIN_AUTH) != 0) {
            proceedHandshakeWithPluggableAuthentication(user, password, database, buf);
            return;
        }
```

SSL or not
```
    if (!this.connection.getUseSSL()) {
    secureAuth411()
    
    //Secure authentication for 4.1 and newer servers.
    secureAuth()
    } else {
        negotiateSSLConnection(user, password, database, packLength);
    }
```