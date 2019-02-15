# WildFly Patch Generator for MariaDB 

An [Apache Maven](http://maven.apache.org) project that creates a patch for a WildFly application server that adds the MariaDB JDBC driver as a base layer module. Creating and installing a patch is an alternative to configuring the module manually (i.e. by writing a `module.xml` that is placed in the modules folder together with the jar file). Building this project will create a single file that includes the `module.xml`, the jar file with the driver and, some meta data that allows you to install the driver via the management web-interface or the command line interface.

## Building

Set the WildFly and MariaDB version using the `wildfly.version` and `mariadb.version` properties of the `pom.xml`. Then run:

~~~
mvn clean package
~~~

Be aware that building the patch will download WildFly into your local Maven repository and it will unpack two instances of WildFly in the build folder. So make sure that you have enough free space on your disk.

## Installation

Apply the generated patch file (`target/wildfly-${wildfly.version}-mariadb-${mariadb.version}.zip`) to your host(s) using the management web-interface of a standalone server or domain (see the patching section) or using the command line interface (`bin/jboss-cli`). For the latter, you would use the following commands:

~~~
connect <management-interface>	                        (e.g. connect http-remoting://localhost:9990)
patch apply <path-to-file-on-disk> --host=<host-name>   (e.g. patch apply /opt/patch.zip --host=master)
~~~

After patching, you will find the patch with some more meta data in the `.installation` folder. Don't forget to restart the patched host(s). 

## Usage

To use the JDBC driver, it must be added to the datasources subsystem in `standalone.xml` or `domain.xml`.  

~~~
<subsystem xmlns="urn:jboss:domain:datasources:5.0">
    <datasources>
        <datasource jndi-name="java:jboss/datasources/mysqldb" pool-name="mysqldb">
            <connection-url>jdbc:mysql://localhost:3306/mysqldb</connection-url>
            <driver>mysql</driver>
            <security>
                <user-name>admin</user-name>
                <password>password</password>
            </security>
        </datasource>
        <drivers>
            <driver name="h2" module="com.h2database.h2">
                <xa-datasource-class>org.h2.jdbcx.JdbcDataSource</xa-datasource-class>
            </driver>
            <driver name="mysql" module="org.mariadb.jdbc" />
        </drivers>
    </datasources>
</subsystem>
~~~

## Issues

When trying to use the patch on a WildFly 15.0.1.Final, I ran into several issues. My understanding of modules in WildFly and JDBC is not thorough enough to explain the underlying problems with certainty. I presume that module name must match the name given in the jar's manifest and that the jar actually contains several JDBC drivers for different API versions. In any case, here are my observations of spending a couple of hours "debugging":

 * Building this project with a different module name than `org.mariadb.jdbc` results in module loading errors. 
 * Using a different name for the driver than `mysql` results in module loading errors. When defining a datasource via the jboss-cli, the error description actually says that the driver calls itself mysql and thus, changing its name is not valid.
 * Explicitly specifiying the `driver-class` (e.g. `org.mariadb.jdbc.Driver`) results in module loading errors. Luckily, the driver is auto-detected properly, if the driver or xa datasource class is not specified at all.
 
## Attribution

 The build logic is based on this very educating [example from hibernate](https://github.com/hibernate/hibernate-demos/tree/master/other/wildfly-patch-creation) that creates a WildFly patch for some other module.
 
Maven, WildFly and MariaDB are most likely trademarks of their respective owners. This project is neither affilated with nor endorsed by Red Hat, Apache or MariaDB Corporation. I just like to use their cool products and mentioning Maven, WildFly and MariaDB seems to be necessary to describe the purpose and usage of this project.

## License

Public domain, after all, it's only 3 XML files that you could have written yourself in less than half an hour, right? Use the code as fits and please, don't sue me for making it available for free. Thank you.
