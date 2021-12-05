# Table of Contents

_**[GDC MIN 2.3.7.1.4 1](#_Toc89543418)**_

_**[How to determine MIN violation? 2](#_Toc89543419)**_

**[Determine OJDBC Jar for WebLogic 2](#_Toc89543420)**

**[Determine OJDBC Jar for Standalone Java Apps 2](#_Toc89543421)**

_**[How to remediate MIN violation? 4](#_Toc89543422)**_

**[Oracle OJDBC Jars 4](#_Toc89543423)**

**[How to choose right JDBC driver? 5](#_Toc89543424)**

**[Key features of JDBC Thin Driver 5](#_Toc89543425)**

**[Supported JDK and JDBC Versions 5](#_Toc89543426)**

[When to Use ojdbc8.jar File 6](#_Toc89543427)

[When to Use ojdbc11.jar File 6](#_Toc89543428)

**[Upgrade JDBC Driver on WebLogic 8](#_Toc89543429)**

**[Upgrading JDBC Driver for Standalone apps. 8](#_Toc89543430)**

_**[Remediation Example 8](#_Toc89543431)**_

**[Upgrading OJDBC Jar on WebLogic 9](#_Toc89543432)**

[Step 1 9](#_Toc89543433)

[Step 2 9](#_Toc89543434)

**[Upgrading OJDBC Jar for Standalone App 10](#_Toc89543435)**

#

# GDC MIN 2.3.7.1.4

OJDBC library version must be of same or higher version than the version of the Oracle database.

Current (at the time when this cookbook is being written) Oracle Database Version at FedEx is 19c, but in future it may be upgraded, so we will provide a generic recipe which can be applied to current version as well as well as future versions.

FedEx applications using OJDBC jar can be grouped in following broad categories:

- Application running in WebLogic server as WARs or EARs
- legacy java apps using JDBC,
- java batch using JDBC,
- Spring MVC or more recently SpringBoot apps using JPA.

In all these cases JDBC is used to connect to database, however in case of WebLogic, it&#39;s server which creates and manages pool of database connections which are used by apps running on server, whereas in case of standalone apps such as java batch/legacy java/Spring MVC/SpringBoot apps, it&#39;s the responsibility of an app to create and manage connections.

Whether an app is running on WebLogic or Standalone, they all need OJDBC jar to connect to Oracle database. We will provide recipes to discover and remediate OJDBC jar MIN for both categories – WebLogic and Standalone.

Most common and widely used version of WebLogic at FedEx is 12.x, so this cookbook will describe steps for 12.x. Other versions of WebLogic may have slightly different directory structure, but overall concept will be same.

# How to determine MIN violation?

## Determine OJDBC Jar for WebLogic

Different versions of WebLogic server may have slightly different directory structure.

For WebLogic 12c, The JDBC drivers (jar files) can be found in $ORACLE\_HOME/oracle\_common/modules

The 12c version of the Oracle Thin driver is installed with Oracle WebLogic Server.

- jar, ojdbc7\_g.jar, and ojdbc7dms.jar for JDK7
- jar, ojdbc6\_g.jar, and ojdbc6dms.jar for JDK 6

## Determine OJDBC Jar for Standalone Java Apps

Batch/Legacy Java/Spring MVC/SpringBoot etc. are considered as standalone Java apps.

Common build tools for standalone Java apps are Ant, Maven or Gradle.

Since, applications built using Ant needs to be migrated to Maven or Gradle, so we will assume that this application is either built by Maven or Gradle.

For Maven projects, locate POM file and look for following:

<dependency>
    <groupId>com.oracle.ojdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>12.3.0.0</version>
</dependency>

For Gradle Projects, look for following:

implementation group: &#39;com.oracle.ojdbc&#39;, name: &#39;ojdbc8&#39;, version: &#39;12.3.0.0&#39;

If you are not able to find an entry in POM or Gradle build file, OJDBC jar may be part of local library.

In that case, JDBC driver version can be determined by executing the following commands:

Java -jar ojdbc.jar

See below for e.g.

syed.a.moid@AMAC02FD380MD6R oracle % java -jar ojdbc8.jar

Oracle 19.3.0.0.0 JDBC 4.2 compiled with javac 1.8.0\_201 on Thu\_Apr\_04\_20:28:01\_PDT\_2019
#Default Connection Properties Resource
#Wed Dec 01 20:18:01 EST 2021

syed.a.moid@AMAC02FD380MD6R oracle % java -jar ojdbc11-21.1.0.0.jar

Oracle 21.1.0.0.0 JDBC 4.3 compiled with javac 11.0.1 on Fri\_Oct\_09\_09:24:33\_PDT\_2020
#Default Connection Properties Resource
#Wed Dec 01 20:19:33 EST 2021

JDBC driver version can also be determined programmatically. Call the getDriverVersion method of the OracleDatabaseMetaData class as shown in the following sample code:

import java.sql.\*;
import oracle.jdbc.\*;
import oracle.jdbc.pool.OracleDataSource;

class FindDBCVersion {

public static void main (String args[]) throws SQLException {
OracleDataSource ods = new OracleDataSource();
ods.setURL("jdbc:oracle:thin:HR/hr@<host>:<port>:<service>");

Connection conn = ods.getConnection();
// Create Oracle DatabaseMetaData object
DatabaseMetaData meta = conn.getMetaData();
// gets driver info:
System.out.println(&quot;JDBC driver version is &quot; + meta.getDriverVersion());
}
}

# How to remediate MIN violation?

Remediation involves picking up the correct version of OJDBC jar, which depends on database and JDK version. Let us first understand that what are OJDBC jars, and then we will use a table to determine correct version for a given database and JDK.

## Oracle OJDBC Jars

Oracle OJDBC Jars contains JDBC Drivers. The JDBC Drivers not only implements Java JDBC specifications, but also provides extensions for enhancing performance and support for oracle specific data types.

Oracle provides the following JDBC drivers:

- Thin driver **(Recommended by Architecture team)**

The JDBC Thin driver is a pure Java, Type IV driver that can be used in applications. It is platform-independent and does not require any additional Oracle software on the client-side. The JDBC Thin driver communicates with the server using Oracle Net Services to access Oracle Database.

The JDBC Thin driver enables a direct connection to the database by providing an implementation of Oracle Net Services on top of Java sockets. The driver supports the TCP/IP protocol and requires a TNS listener on the TCP/IP sockets on the database server.

- Oracle Call Interface (OCI) driver

It is used on the client-side with an Oracle client installation. The JDBC OCI driver is a Type II driver used with Java applications. It requires platform specific OCI libraries.

- Server-side Thin driver

It is used for code that runs on the database server and needs to access another session either on the same server or on a remote server on any tier.

- Server-side internal driver

It is used for code that runs on the database server and accesses the same session. That is, the code runs and accesses data from a single Oracle session.

## How to choose right JDBC driver?

Consider the following when choosing a JDBC driver for your application:

In general, unless you need OCI-specific features, such as support for non-TCP/IP networks, use the JDBC Thin driver.

- Fox maximum portability and performance, use the JDBC Thin driver. Application can connect to Oracle Database using the JDBC Thin driver.

- When using Lightweight Directory Access Protocol (LDAP) over Secure Sockets Layer (SSL)/Transport Layer Security (TLS), then also use the JDBC Thin driver.

- For a client application which need OCI-driver-specific features, such as support for non-TCP/IP networks, then use the JDBC OCI driver.

- For code that runs in the database server and needs to access a remote database or another session within the same database instance, use the JDBC server-side Thin driver.

- For code running inside the database server that needs to access data locally within the session, then use the JDBC server-side internal driver to access that server.

## Key features of JDBC Thin Driver

- Row count per iteration
- Support for promoting a local transaction to a global transaction
- Transaction Guard
- Transparent Application Continuity and Application Continuity
- Support for the Reactive Streams Ingestion library
- JDBC Reactive Extensions

## Supported JDK and JDBC Versions

In Oracle Database 21c, all the JDBC drivers are compatible with JDK 8, JDK 11, JDK 12, JDK 13, JDK 14, and JDK 15, and the ojdbc8.jar and ojdbc11.jar files provide the support to these JDK versions.

### When to Use ojdbc8.jar File

Use the ojdbc8.jar file when you want JDBC 4.2 features and need to compile your code with JDK 8, JDK11, JDK12, JDK13, JDK14, and JDK15.

### When to Use ojdbc11.jar File

Use the ojdbc11.jar file when you want JDBC 4.3 features and need to compile your code with JDK 11, JDK 12, JDK13, JDK14, and JDK15.

Oracle Database and JDK Version Compatibility for Oracle JDBC Drivers

Oracle Database Release 21c JDBC drivers are certified with all supported Oracle Database releases (21c, 19c, 18c, 12_c_, and 11_g_ Release 2). However, they are not certified to work with older, unsupported database releases, such as 10_g_ and 9_i_.

The following table describes the JDBC and Oracle Database interoperability matrix or the certification matrix:

| **JDBC Driver Version** | **Database 21.x** | **Database 19.x** | **Database 18.3** | **Database 12.2 and 12.1** | **Database 11.2.0.4** |
| --- | --- | --- | --- | --- | --- |
| **JDBC 21.x** | Yes | Yes | Yes | Yes | Yes |
| **JDBC 19.x** | Yes | Yes | Yes | Yes | Yes |
| **JDBC 18.3** | Yes | Yes | Yes | Yes | Yes |
| **JDBC 12.2 and 12.1** | Yes | Yes | Yes | Yes | Yes |
| **JDBC 11.2.0.4** | Yes | Yes | Yes | Yes | Yes |

Oracle JDBC Drivers are always compliant to the latest JDK version for every new release. For some versions, JDBC drivers support multiple JDK versions. The following table describes the release specific JDBC JAR files and supported JDK versions for various Oracle Database versions:

| **Oracle Database Version** | **Release-Specific JDBC JAR File with Supported JDK Version** |
| --- | --- |
| 21.x | ojdbc11.jar with JDK 11, JDK 12, JDK 13, JDK 14 and JDK 15ojdbc8.jar with JDK 8, JDK 11, JDK 12, JDK 13, JDK 14 and JDK 15 |
| 19.x | ojdbc10.jar with JDK 10, JDK 11ojdbc8.jar with JDK 8, JDK 9, JDK 11 |
| 18.3 | ojdbc8.jar with JDK 8, JDK 9, JDK 10, JDK 11 |
| 12.2 or 12cR2 | ojdbc8.jar with JDK 8 |
| 12.1 or 12cR1 | ojdbc7.jar with JDK 7, JDK 8ojdbc6.jar with JDK 6 |
| 11.2 or 11gR2 | ojdbc6.jar with JDK 6, JDK 7, JDK 8ojdbc5.jar with JDK 5 |

## Upgrade JDBC Driver on WebLogic

To update an existing JDBC driver:

1. Make backup copies of the OJDBC JAR files. Drivers installed with WebLogic Server are located in subdirectories of the $ORACLE\_HOME/oracle\_common/modules directory.
2. In the appropriate location, replace the existing JAR with the updated JAR.
3. Determine if you need to modify your CLASSPATH:
  - If the JDBC driver JAR was installed with WebLogic Server and the replacement JAR has the same name, you do not need to modify CLASSPATH. The manifest in the weblogic.jar file directly or indirectly includes these files so they will load automatically when the server starts.
  - If you are adding a new JDBC driver or updating a JDBC driver where the replacement JAR has a different name than the original JAR:
    - For all domains, edit the commEnv.cmd/sh script in _WL\_HOME_/common/bin and prepend your JAR file to the WEBLOGIC\_CLASSPATH environment variable. Your JAR must be located before any client JAR files.
    - For a specific WebLogic Server domain, edit the setDomainEnv.cmd/sh script in that domain&#39;s bin directory, and prepend the JAR file to the PRE\_CLASSPATH environment variable. Your JAR must be located before any client JAR files.

## Upgrading JDBC Driver for Standalone apps.

Determine FedEx compliant version of OJDBC jar using above tables. For Maven projects, update POM.xml and for Gradle project update gradle build file with compliant version of OJDBC jar.

# Remediation Example

## Upgrading OJDBC Jar on WebLogic

To upgrade the JDBC driver in weblogic you need to do two set of actions. The First one is Replace the existing JDBC driver and the second one is updating the Classpath

### Step 1

- Take a backup of the Existing JAR file in the $ORACLE\_HOME/oracle\_common/modules to somewhere in the file system ( NOT ON THE SAME DIRECTORY)
- Download the latest mysql JDBC driver from here
- The Downloaded driver would mostly come in the Zipped format or as tarball. So un compress it to find the JAR file
- Take the JAR file and Just place it in the same directory $ORACLE\_HOME/oracle\_common/modules

### Step 2

There are two options while updating the classpath, based on the scope of your change. Either a Single Domain (or) across all domains.

To Apply the Changes Across All the Domains

If you want the latest JDBC driver to be picked up across all the domains in the server. Perform the following steps

- Go to $ORACLE\_HOME/oracle\_common/common/bin
- Take a backup of commExtEnv.sh and open the file in VI
- Find the WEBLOGIC\_CLASSPATH variable declaration
- Append the new Fully Qualified Latest JDBC driver JAR file to the WEBLOGIC\_CLASSPATH variable. Append the following line at the end of the WEBLOGIC\_CLASSPATH :$ ORACLE\_HOME /oracle\_common/modules/ ojdbc8.jar

To Apply the Changes For a Single Domain

- Go to $DOMAIN\_HOME/bin
- Open setDomainEnv.sh script
- Find the variable declaration PRE\_CLASSPATH
- Prepend (or) Append the Fully Qualified JAR file location to the PRE\_CLASSPATH
- PRE\_CLASSPATH=$PRE\_CLASSPATH:$ ORACLE\_HOME /oracle\_common/modules/ ojdbc8.jar

## Upgrading OJDBC Jar for Standalone App

Suppose a given application is using Oracle Database 19.c and JDK 8.

So, using above guidelines and recipes we can upgrade POMs or Gradle build file as follows.

<dependency>
    <groupId>com.oracle.ojdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>19.3.0.0</version>
</dependency>

For Gradle Projects, upgrade as below:

implementation group: 'com.oracle.ojdbc', name: 'ojdbc8', version: '19.3.0.0'

Suppose a given application is using Oracle Database 21.c and JDK 11.

So, using above guidelines and recipes we can upgrade POMs or Gradle build file as follows.
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc11</artifactId>
    <version>21.3.0.0</version>
</dependency>

Gradle build file:
implementation group: 'com.oracle.database.jdbc', name: 'ojdbc11', version: '21.3.0.0'
