# Wildfly
## Set Wildfly's Home Path
* JBOSS_HOME=D:\Services\wildfly-10.1.0.Final
* ;%JBOSS_HOME%\bin

## Add User in wildfly
```shell
D:\DATA\JAVA_program\program\server\wildfly\wildfly-26.1.1.Final\bin>add-user

What type of user do you wish to add?
 a) Management User (mgmt-users.properties)
 b) Application User (application-users.properties)
(a):

Enter the details of the new user to add.
Using realm 'ManagementRealm' as discovered from the existing property files.
Username : admin
User 'admin' already exists and is disabled, would you like to...
 a) Update the existing user password and roles
 b) Enable the existing user
 c) Type a new username
(a): a
Password recommendations are listed below. To modify these restrictions edit the add-user.properties configuration file.
 - The password should be different from the username
 - The password should not be one of the following restricted values {root, admin, administrator}
 - The password should contain at least 8 characters, 1 alphabetic character(s), 1 digit(s), 1 non-alphanumeric symbol(s)
Password :
WFLYDM0099: Password should have at least 8 characters!
Are you sure you want to use the password entered yes/no? yes
Re-enter Password :
What groups do you want this user to belong to? (Please enter a comma separated list, or leave blank for none)[  ]:
Updated user 'admin' to file 'D:\DATA\JAVA_program\program\server\wildfly\wildfly-26.1.1.Final\standalone\configuration\mgmt-users.properties'
Updated user 'admin' to file 'D:\DATA\JAVA_program\program\server\wildfly\wildfly-26.1.1.Final\domain\configuration\mgmt-users.properties'
Updated user 'admin' with groups  to file 'D:\DATA\JAVA_program\program\server\wildfly\wildfly-26.1.1.Final\standalone\configuration\mgmt-groups.properties'
Updated user 'admin' with groups  to file 'D:\DATA\JAVA_program\program\server\wildfly\wildfly-26.1.1.Final\domain\configuration\mgmt-groups.properties'
Is this new user going to be used for one AS process to connect to another AS process?
e.g. for a slave host controller connecting to the master or for a Remoting connection for server to server Jakarta Enterprise Beans calls.
yes/no? yes
To represent the user add the following to the server-identities definition <secret value="MTIzNDU2" />
Press any key to continue . . .

D:\DATA\JAVA_program\program\server\wildfly\wildfly-26.1.1.Final\bin>
```
* account: admin
* password: 123456

```shell
$ ./add-user.sh -u 'admin' -p '123456'
```

## Adjust Wildfly's port
standalone\configuration\standalone.xml


## Remote Access
* Web interface: http://<host>:9990/console
* Management operations: http://<host>:9990/management

By default, **the web console is only accessible from localhost**. That is to say, our configuration files contain only local interfaces to interact with a web console.

In WildFly jargon, an interface consists of a network interface with selection criteria. In most cases, a selection criterion is the bound IP address to the interface. The local interface is declared as follow :

```xml
<interface name="management">
    <inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
</interface>
<!--127.0.0.1 is the localhost IP address. -->
```

As a result, this management local is attached to socket listener **management-http** receiving connections for web console from port 9000:

```xml
<socket-binding-group name="standard-sockets" default-interface="public" 
  port-offset="${jboss.socket.binding.port-offset:0}">
    <socket-binding name="ajp" port="${jboss.ajp.port:8009}"/>
    <socket-binding name="http" port="${jboss.http.port:8080}"/>
    <socket-binding name="https" port="${jboss.https.port:8443}"/>
    <socket-binding name="management-http" interface="management" 
      port="${jboss.management.http.port:9990}"/>
    <socket-binding name="management-https" interface="management" 
      port="${jboss.management.https.port:9993}"/>
    <socket-binding name="txn-recovery-environment" port="4712"/>
    <socket-binding name="txn-status-manager" port="4713"/>
    <outbound-socket-binding name="mail-smtp">
       <remote-destination host="localhost" port="25"/>
    </outbound-socket-binding>
</socket-binding-group>
```
To allow access from a remote machine, we first need to create the remote management interface in the appropriate configuration file. If we're configuring a standalone server, we'll change ``standalone/configuration/standalone.xml``, and for domain-managed, we'll change ``domain/configuration/host.xml``:

```xml
<interface name="remoteManagement">
    <inet-address value="${jboss.bind.address.management:REMOTE_HOST_IP}"/> 
</interface> 
<!--REMOTE_HOST_IP is the remote host IP address. (e.g 192.168.1.2) -->
```

We must also modify the socket binding of management-http to delete the previous local interface  and add the new one:

```xml
<socket-binding-group name="standard-sockets" default-interface="public" 
  port-offset="${jboss.socket.binding.port-offset:0}">
    <!-- same as before -->
    <socket-binding name="management-http" interface="remoteManagement" 
      port="${jboss.management.http.port:9990}"/>
    <socket-binding name="management-https" interface="remoteManagement" 
      port="${jboss.management.https.port:9993}"/>
    <!-- same as before -->
</socket-binding-group>
```

In the above configuration, we bind the new remoteManagement interface to our HTTP (9990) and HTTPS (9993) ports. It'll allow the remote host IP to connect to the web interface through HTTP/HTTPS ports.


# [Getting started with Arquillian](http://www.mastertheboss.com/jboss-frameworks/arquillian/arquillian-tutorial/)
* source-code: https://github.com/fmarchioni/mastertheboss/tree/master/arquillian/sample-jakartaee-webapp-archetype  

* This project was created from the archetype "wildfly-jakartaee-webapp-archetype".


* To build it:

```shell
mvn  clean package -DskipTests
```

* To deploy it:
Run the maven goals "install wildfly:deploy"

```shell
mvn  clean package wildfly:deploy -DskipTests
```
> Before deployin, you have to adjust the configuration of org.wildfly.plugins in pom.xml.

> ```xml
>  <!-- The WildFly plugin deploys your war to a local JBoss AS container -->
>             <plugin>
>                 <groupId>org.wildfly.plugins</groupId>
>                 <artifactId>wildfly-maven-plugin</artifactId>
>                 <version>${version.wildfly.maven.plugin}</version>
>                 <configuration>
>           <!--        <hostname>192.168.18.30</hostname> -->
>                        <hostname>localhost</hostname>
>                        <port>9990</port>
>                        <username>admin</username>
>                        <password>123456</password>
>                        <!-- <jbossHome>C:/tools/wildfly-8.0.0.Final</jbossHome> -->
>               </configuration>
>            </plugin>
> ```


* To undeploy it:
Run the maven goals "wildfly:undeploy"



### DataSource:
This sample includes a "persistence.xml" file in "src/main/resources/META-INF". This file defines
a persistence unit "sample-jakartaee-webapp-archetypePersistenceUnit" which uses the JakartaEE default database.

In production environment, you should define a database in WildFly config and point to this database
in "persistence.xml".

> If you don't use entity beans, you can delete "persistence.xml".


### JSF:

The web application is prepared for JSF 2.3 by bundling an empty "faces-config.xml" in "src/main/webapp/WEB-INF".
In case you don't want to use JSF, simply delete this file and "src/main/webapp/beans.xml" and "src/main/java/com/masterheboss/Jsf23Activator.java"


### Testing:

This sample is prepared for running unit tests with the Arquillian framework.

#### Configuration in pom.xml 
The configuration can be found in "sample-jakartaee-webapp-archetype/pom.xml":

Three profiles are defined:
* **default**: no integration tests are executed.
* **arq-remote**: you have to start a WildFly server on your machine. The tests are executed by deploying 
 the application to this server.
 Here the "maven-failsafe-plugin" is enabled so that integration tests can be run.
 Run maven with these arguments: "``clean verify -Parq-remote``"
* **arq-managed**: this requires the environment variable "``JBOSS_HOME``" to be set: 
 The server found in this path is started and the tests are executed by deploying the application to this server.
 Instead of using this environment variable, you can also define the path in "arquillian.xml".
 Here the "maven-failsafe-plugin" is enabled so that integration tests can be run.
 Run maven with these arguments: "``clean verify -Parq-managed``"

```xml
<profiles>
        <profile>
            <!-- All the modules that require nothing but WildFly or JBoss EAP -->
            <id>default</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        
        <profile>
            <!-- An optional Arquillian testing profile that executes tests in your WildFly / JBoss EAP instance.
                 This profile will start a new WildFly / JBoss EAP instance, and execute the test, shutting it down when done.
                 Run with: mvn clean verify -Parq-managed -->
            <id>arq-managed</id>
            <dependencies>
                <dependency>
                    <groupId>org.wildfly.arquillian</groupId>
                    <artifactId>wildfly-arquillian-container-managed</artifactId>
                    <scope>test</scope>
                </dependency>
            </dependencies>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-failsafe-plugin</artifactId>
                        <version>${version.failsafe.plugin}</version>
                        <executions>
                            <execution>
                                <goals>
                                    <goal>integration-test</goal>
                                    <goal>verify</goal>
                                </goals>
                            </execution>
                        </executions>
                        <configuration>
                            <!-- Configuration for Arquillian: -->
                            <systemPropertyVariables>
                                <!-- Defines the container qualifier in "arquillian.xml" -->
                                <arquillian.launch>managed</arquillian.launch>
                            </systemPropertyVariables>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>

        <profile>
            <!-- An optional Arquillian testing profile that executes tests in a remote JBoss EAP instance.
                 Run with: mvn clean verify -Parq-remote -->
            <id>arq-remote</id>
            <dependencies>
                <dependency>
                    <groupId>org.wildfly.arquillian</groupId>
                    <artifactId>wildfly-arquillian-container-remote</artifactId>
                    <scope>test</scope>
                </dependency>
            </dependencies>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-failsafe-plugin</artifactId>
                        <version>${version.failsafe.plugin}</version>
                        <executions>
                            <execution>
                                <goals>
                                    <goal>integration-test</goal>
                                    <goal>verify</goal>
                                </goals>
                            </execution>
                        </executions>
                        <configuration>
                            <!-- Configuration for Arquillian: -->
                            <systemPropertyVariables>
                                <!-- Defines the container qualifier in "arquillian.xml" -->
                                <arquillian.launch>remote</arquillian.launch>
                            </systemPropertyVariables>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>

```


#### Configuration in arquillian.xml
The Arquillian test runner is configured with the file "``src/test/resources/arquillian.xml``" 
(duplicated in EJB and WEB project, depending where your tests are placed).
The profile "``arq-remote``" uses the container qualifier "``remote``" in this file.
The profile "``arq-managed``" uses the container qualifier "``managed``" in this file.

The project contains an integration test "SampleIT" which shows how to create the deployable WAR file using the ShrinkWrap API.
You can delete this test file if no tests are necessary.

Why integration tests instead of the "maven-surefire-plugin" testrunner?
The Arquillian test runner deploys the WAR file to the WildFly server and thus you have to build it yourself with the ShrinkWrap API.
The goal "verify" (which triggers the maven-surefire-plugin) is executed later in the maven build lifecyle than the "test" goal so that the target 
artifact ("sample-jakartaee-webapp-archetype.war") is already built. You can build
the final WAR by including those files. The "maven-surefire-plugin" is executed before the WAR file
are created, so this WAR files would have to be built in the "@Deployment" method, too. 


````xml
<arquillian xmlns="http://jboss.org/schema/arquillian" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://jboss.org/schema/arquillian
    http://jboss.org/schema/arquillian/arquillian_1_0.xsd">
    <defaultProtocol type="Servlet 3.0" />

    <container qualifier="managed">
    </container>   
    
    <container qualifier="remote">
        <!-- Arquillian will deploy to this WildFly server. -->
        <configuration>
           <!--  <property name="managementAddress">192.168.18.30</property> -->
            <property name="managementAddress">localhost</property>
            <property name="managementPort">9990</property>
            <!-- If deploying to a remote server, you have to specify username/password here -->
            <!-- -->
             <property name="username">admin</property>
            <property name="password">123456</property>
        </configuration>
    </container>
</arquillian>
````

#### Configuration in CDI service
JBossAS checks that file to start the CDI service with the file "``src/main/webapp/WEB-INF/beans.xml``" .
You maybe ran into trouble that said "WELD-001408 Unsatisfied dependencies when injecting EntityManager".

````xml
<?xml version="1.0" encoding="UTF-8"?> 
<!--
<beans version="2.0"
   xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="
      http://xmlns.jcp.org/xml/ns/javaee
      http://xmlns.jcp.org/xml/ns/javaee/beans_2_0.xsd"
   bean-discovery-mode="annotated">
    -->
<beans version="2.0"
   xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="
      http://xmlns.jcp.org/xml/ns/javaee
      http://xmlns.jcp.org/xml/ns/javaee/beans_2_0.xsd"
    bean-discovery-mode="all">
   <!-- This descriptor configures Context and Dependeny Injection.
        Actually, CDI 1.1 does not require this file. But the archetype contains it anyway to avoid deloyment errors for blank projects (WFLY-13306)   -->

</beans>
````