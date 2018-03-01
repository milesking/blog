
# EBC migrate to tomcat

## Load configuration at EBCStartup.java
Before
``` java
GMTConfigProperty.loadProperties();
```
Add
``` java
System.setProperty("SYSTEM_PROPS_FILE", "C:\\Users\\e650728\\Dev\\ebc\\ebc-usrlocal\\cfg\\ebc-config.agsos195.properties");
```
## Modify ebc-config.agsos195.properties
Replace
```
log4j.rootLogger=DEBUG,GMT
```
With
```
log4j.rootLogger=DEBUG,GMT,stdout
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%t_%X{user}] - %m%n
```
Replace
```
PASSWORD_MATRIX_DATABASE=O01GRD0
PASSWORD_MATRIX_USERNAME=EBCDBA
```
With
```
JDBC_DATASOURCE_USERNAME=EBCDBA
JDBC_DATASOURCE_PASSWORD=EBCDBA
```
Replace
```
com.statestr.gmt.ebc.util.ds.DataSourceConnector.implClass=com.statestr.gmt.ebc.util.ds.DataSourceConnectorPwMatrixImpl
```
With
```
com.statestr.gmt.ebc.util.ds.DataSourceConnector.implClass=com.statestr.gmt.ebc.util.ds.DataSourceConnectorJndiImpl
```
Add
```
com.statestr.gmt.common.db.JDBC.Dialect=org.hibernate.dialect.OracleDialect
```
## Modify EBCDataService.java
Replace
```
InputStream ITs = lCl.getResourceAsStream(GMTConfigProperty.getProperty(CONFIG_FILE));
```
With
```
String xmlPath = GMTConfigProperty.getProperty(CONFIG_FILE);
InputStream ITs = lCl.getResourceAsStream(xmlPath);
```
## Modify DataSourceConnectorJndiImpl.java
Replace
```
InitialContext lIC = new InitialContext();
ds = (javax.sql.DataSource) lIC.lookup(jndiName);
```
With
```
InitialContext initContext = new InitialContext();
Context lIC = (Context)initContext.lookup("java:/comp/env");
ds = (javax.sql.DataSource) lIC.lookup(jndiName);
```
## Modify PwMatrix.class in PwMatrix.jar
```
package Matrix;

public class PwMatrix {
    private PwMatrix() {
    }

    public static String getPassword(String var0, String var1) {
        return "47K7ck%$";
    }
}
```
## Modify DatasourceConnectionProvider.class in hibernate3.jar
Replace
```
try {
        ...
} catch (Exception e) {
        ...
}
```
With
```
try {
    InitialContext initContext = NamingHelper.getInitialContext(props);
    Context lIC = (Context)initContext.lookup("java:/comp/env");
    this.ds = (DataSource)lIC.lookup(jndiName);
} catch (Exception var4) {
    log.fatal("Could not find datasource: " + jndiName, var4);
    throw new HibernateException("Could not find datasource", var4);
}
```
## Modify GMTDBUtils.class in gmt-common.jar
Replace
```
private static void lookupDataSource() throws Exception {
    ...
}
```
With
```
private static void lookupDataSource() throws Exception {
    GMTLogger.logInfo("GMTDBUtils", "About to look up the inital context");
    InitialContext initContext = new InitialContext();
    Context lIC = (Context)initContext.lookup("java:/comp/env");
    sDataSource = (DataSource)lIC.lookup(jndiName);
    GMTLogger.logInfo("GMTDBUtils", "Got JDBC DataSource '" + jndiName + "'");
}
```
## Using newbuild.xml to build EBC
```
ant all -f newbuild.xml
```
## Add datasource config in tomcat server.xml
```
<GlobalNamingResources>
	<Resource name="jdbc/ebcds" auth="Container"
              type="javax.sql.DataSource" driverClassName="oracle.jdbc.driver.OracleDriver"
              url="jdbc:oracle:thin:@dgrdd0.it:1521:O01GRD0"
              username="ebcdba" password="ebcdba" maxActive="20" maxIdle="10"
              maxWait="-1"/>
</GlobalNamingResources>
```
## Add config in web.xml
```
	<resource-ref>
		<description>DB Connection Pooling</description>
		<res-ref-name>jdbc/ebcds</res-ref-name>
		<res-type>javax.sql.DataSource</res-type>
		<res-auth>Container</res-auth>
	</resource-ref>
```
## Add config in context.xml
```
<ResourceLink global="jdbc/ebcds" name="jdbc/ebcds" type="javax.sql.DataSource" />
```
## Modify esapi.tld
function
```
<name>encodeForXMLAttribute</name>
```
add
```
<function-signature>java.lang.String encodeForXML(java.lang.String)</function-signature>
```
## Modify ebc web.xml
from
```
<url-pattern>common/callback</url-pattern>
```
to
```
<url-pattern>/common/callback</url-pattern>
```
from
```
<url-pattern>common/logout</url-pattern>
```
to
```
<url-pattern>/common/logout</url-pattern>
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzNTIyNTU0NjBdfQ==
-->