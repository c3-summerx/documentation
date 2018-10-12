## Disclaimer

This is an unofficial documentation on my own explorations of Kerberos authentication with Hadoop/HBase during the work of setting up new features for client. PLease note that there’s no guarantee that my information is absolutely correct at the time of writing, as myself is still learning it.

## Kerberos - What, How
Kerberos is a server-client authentication protocol. Distributed systems tend to use it as a secure process.  
It's based on symmetric key cryptography and there's an authority called **KDC** (Key Distribution Center) to manage **TGT**s (Ticket Granting Tickets), which are used to request services/exchange for new tickets.

[Here](https://www.hack2secure.com/blogs/how-kerberos-authentication-works) has an overview of Kerberos authentication flows.

Some more important concepts are:  
**principal**: the user identity in the systemand.  
**realm**: like the subnet where all principals live.  
**keytab**: a binary file contains the secrets needed to login. That being said, this file **must absolutely** be stored in somewhere secure.

#### Kerberos Shell Commands
1. klist - list the current status
```
Summer-Xia:c3server summerxia$ klist -v
Credentials cache: API:7B9B80D5-7543-4DD8-83B7-078358283FB4
        Principal: zcdhgrpc3gendev@RISORSE.ENEL
    Cache version: 0

Server: krbtgt/RISORSE.ENEL@RISORSE.ENEL
Client: zcdhgrpc3gendev@RISORSE.ENEL
Ticket etype: aes256-cts-hmac-sha1-96, kvno 3
Ticket length: 1119
Auth time:  Oct  9 12:19:46 2018
End time:   Oct  9 22:19:34 2018 (expired)
Ticket flags: enc-pa-rep, pre-authent, initial, forwardable
Addresses: addressless
```
2. kinit - creates an initial ticket for the principal

## Java API 
1. Kerberos support is built into the Java JRE.
2. All necessary tools are built and tested in org.apache.hadoop . 
3. If you need some sofisticated stuff like programmatically geneating keytabs, look into org.apache.directory.server APIs.
4. When estabishing HBase/HDFS connections using Java API, the configuration is expected. You can load configurations from xmls or explicitly set the key-values. Note that in my experience, if you choose the formal method then all 3 xmls (**core-site.xml**, **hdfs-site.xml**, **hbase-site.xml**) are required or you'll get authentication errors. You don't have to do that using latter method. Below configuration code would estabilish a basic connection:
```
config.set("hbase.zookeeper.quorum", "127.0.0.1");
config.set("hbase.zookeeper.property.clientPort", "2181");
config.set("zookeeper.znode.parent", "/hbase");
(More code exmaples can be checked out [here](https://community.hortonworks.com/articles/120858/connecting-to-kerberos-secured-hbase-cluster-from.html)).
```
5. For Kerberized cluster, a **krb5.conf** file is needed, where all the hostnames/realms being defined. Note that you can only setup the default kdc `java.security.krb5.kdc` and default realm `java.security.krb5.kdc` programmatically, anything beyond this are not supported in native JDK. Most of time, if you want to configure more than one kdc/realm, a **krb5.conf** is unavoided.
6. UserGroupInformation (UGI) - it's one place for almost all Kerberos/User authentication to live. It's used in the entire Hadoop stack. A few things to know about it:
* It's a singleton. Once initialized, it stays initialized and cannot be reset. However, there is actually a UGI.reset() call, but it is package scoped and purely to allow tests to reset the information.
* It uses some 'magic' calls to look at local filesystem to find the current user/principal.

to be continued.
