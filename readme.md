Copyright (c) 2014 Oracle and/or its affiliates. All rights reserved.

You may not modify, use, reproduce, or distribute this software except in
compliance with  the terms of the License at:
http://java.net/projects/javaeetutorial/pages/BerkeleyLicense

Please find the Oracle dukes-forest tutorial documentation at [Duke's Forest Case Study Example](https://docs.oracle.com/javaee/7/tutorial/dukes-forest.htm#GLNPW) 

# dukes-forest tutorial example.

This document is divided into **Installation and running** and **Notes on changes to the original source**

#### Installation and running procedures.

* Date 02/2106

* Clone git repository `git clone https://github.com/karlnicholas/dukes-forest.git`.

##### Database procedures.

* Create a schema in MySQL named `forest` in lowercase. I used the workbench.

* Create the database user and password in your `MySQL` server. Give this user full privileges to the `forest` schema. This username and password will need to be added to your `Forest` datasource.

* Create the schema. There is an issue with having JPA auto create the schema and load the data. It works, but it works for both dukes-shipment and dukes-store. The create sql script won't create the database twice, but the load sql script will load twice, so let's just do it by hand because I have commented out the create and load properties in the entities persistence.xml file. The create.sql script is in `entities/src/main/resources/META-INF/sql/create.sql`.

* Load the default data. The load data.sql script is in `entities/src/main/resources/META-INF/sql/data.sql`.

##### Wildfly procedures.

* Pick a Wildfly configuration file to work with: **Be sure you are running the `standalone-full` version.**, or at least have JMS messaging enabled ( I used the standalone-full.xml, I don't know the exact configuration needed, but standalone-full.xml worked).

* You can either edit and run the wildfly-commands.cli command line file found in dukes-forest/wildfly-commands.cli, you can can manually do each step below. 

* Add `XA` mysql jdbc driver to Wildfly if you have not already done so. See [Example MySQL XA Datasource](https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6/html/Administration_and_Configuration_Guide/Example_MySQL_XA_Datasource1.html) for example.

* Add a `XA` datasource and driver to wildfly.  Set the jndi-name `java:jboss/ForestXADS` and name `ForestXADS`. This can be done through the management console, or by copying and pasting the XML snippet shown in the notes below. Be sure the datasource is enabled and you can sucessfully test the connection from the wildfly management 
console.

* Add a message queue wildfly named OrderQueue, as per specs below.

        <jms-queue name="OrderQueue">
            <entry name="java:global/jms/queue/OrderQueue"/>
            <durable>true</durable>
        </jms-queue>

* Add a security-domain to wildfly named dukes-forest, as per specs below. Note that on unix systems the queries are case sensitive.

        <security-domain name="dukes-forest" cache-type="default">
            <authentication>
                <login-module code="org.jboss.security.auth.spi.DatabaseServerLoginModule" flag="required">
                    <module-option name="dsJndiName" value="java:jboss/ForestXADS"/>
                    <module-option name="rolesQuery" value="select NAME as 'ROLES', 'Roles' as 'ROLEGROUP' from forest.GROUPS g inner join forest.PERSON_GROUPS pg on g.ID = pg.GROUPS_ID join forest.PERSON p on p.EMAIL = pg.EMAIL where p.EMAIL = ?"/>
                    <module-option name="hashAlgorithm" value="MD5"/>
                    <module-option name="hashEncoding" value="HEX"/>
                    <module-option name="principalsQuery" value="select PASSWORD from forest.PERSON where EMAIL=?"/>
                </login-module>
            </authentication>
            <authorization>
                <policy-module code="org.jboss.security.auth.spi.DatabaseServerLoginModule" flag="required">
                    <module-option name="dsJndiName" value="java:jboss/ForestXADS"/>
                    <module-option name="rolesQuery" value="select NAME as 'ROLE', 'ROLES' as 'ROLEGROUP' from forest.GROUPS g inner join forest.PERSON_GROUPS pg on g.ID = pg.GROUPS_ID join forest.PERSON p on p.EMAIL = pg.EMAIL where p.EMAIL = ?"/>
                    <module-option name="hashAlgorithm" value="MD5"/>
                    <module-option name="hashEncoding" value="HEX"/>
                    <module-option name="principalsQuery" value="select PASSWORD from forest.PERSON where EMAIL=?"/>
                </policy-module>
            </authorization>
        </security-domain>

##### dukes-forest steps.
  
* Run `mvn clean verify` from the dukes-forest directory. This will build 3 jar files and 3 war files, `events.jar`, `entities.jar`, `dukes-resources.jar`, and `dukes-payment.war`, `dukes-shipment.war`, and `dukes-store.war`, in that order.

* Run `mvn install` from the dukes-forest directory, so that the jars will be put into your repository.

* Deploy dukes-payment, dukes-shipment, and dukes-store to wildfly.
  **There was an issue here.** See [Schema not generated if Entities and Persistence.xml in another jar](https://issues.jboss.org/browse/WFLY-6151). Since dukes-forest has its entities in the entities.jar, the database create scripts would not get executed if you deployed from maven or copied the .war files manually. Only `Run on Server` from Eclipse worked. There was also a problem because both the dukes-shipment.war and the dukes-store.war have the entities.jar in them. The entities.jar has the persistence.xml file, which was configurated to tell the server to create the database and load the default data. That means it was done twice, which was problematic. The issue doesn't apply because the created the schema was created and loaded manually as per the database procedures above. 
  
* Open http://localhost:8080/dukes-store to run dukes-store. You will need the built-in administrator account to login to dukes-shipment, which is username=admin@example.com and password=1234.

* Refer to the [Duke's Forest Case Study Example](https://docs.oracle.com/javaee/7/tutorial/dukes-forest.htm#GLNPW) tutorial for further information on using the example.

# Notes on changes to original source for porting to Wildfly 9.

Port of Dukes-Forest tutorial to Wildfly 9 and MySql 5.6.

* Date: 10/2015

* Created projects for jboss developer studio 9.0 ga, eclipse 4.5 (Mars).
  dukes-payment, dukes-resources, dukes-shipment, dukes-store, entities, events.
  
* Changed entities/src/main/resources/META-INF/persistence.xml to comment out the drop-create properties. 
   
* Fixed up Database scripts and model. I used the dukes-forest-model.mwb, the MySQL workbench EER Diagram.
  The diagram was out of sync with the entities/src/main/resources/META-INF/sql/create.sql script. Mostly the
  differences included auto increment of indexes, non-null fields, and, more importantly, the length of 
  various var-char fields. Also, there was an issue with a Foreign Key Index on the Product table being
  named the same as the Foreign Key.    
  
* Changed entities/src/main/resources/META-INF/persistence.xml to use java:jboss/ForestXADS instead of
  java:global/ForestXADS. 
  
* Added java:jboss/ForestXADS datasource and mysql driver in standalone-full.xml. 
  
        <datasources>
           <xa-datasource jndi-name="java:jboss/ForestXADS" pool-name="ForestXADS" enabled="true">
           <driver>com.mysql</driver>
           <xa-datasource-property name="URL">jdbc:mysql://localhost:3306/forest</xa-datasource-property>
           <security>
             <user-name>forest</user-name>
             <password>forest</password>
           </security>
         </xa-datasource>
         <drivers>
           <driver name="mysql" module="com.mysql">
             <datasource-class>com.mysql.jdbc.Driver</datasource-class>
             <xa-datasource-class>com.mysql.jdbc.jdbc2.optional.MysqlXADataSource</xa-datasource-class>
           </driver>
         </drivers>
       </datasources>
       
  and ...
  
        <driver name="mysql" module="com.mysql">
            <xa-datasource-class>com.mysql.jdbc.jdbc2.optional.MysqlXADataSource</xa-datasource-class>
        </driver>
  
    
* Created appropriate queue in Wildfly and made Eclipse run Wildfly with standalone-full.xml
  so that JMS services would be available.
    
        <jms-queue name="OrderQueue">
            <entry name="java:global/jms/queue/OrderQueue"/>
            <durable>true</durable>
        </jms-queue>

* Changed dukes-shipment/src/main/java/com.forest.shipment.ejb.OrderBrowser to use jboss compatible jndi name 
  for jms message queue. 
  
        @Resource(mappedName = "java:global/jms/queue/OrderQueue")
        
* Changed dukes-store/src/main/java/com/forest/ejb/OderJMSManager.java to use jboss compatible jndi name
  for jms message queue.
  
        name = "java:jboss/jms/queue/OrderQueue",
        @Resource(mappedName = "java:global/jms/queue/OrderQueue")        
  
* Removed src/main/webapp/WEB-INF/glassfish-web.xml in dukes-payment, dukes-shipment and dukes-store projects. 
  Added jboss-web.xml set to dukes-forest security domain in WEB-INF directory. 
  
        <?xml version="1.0" encoding="UTF-8"?>
        <jboss-web>
            <security-domain>dukes-forest</security-domain>
        </jboss-web>

* Added dukes-forest security-domain in standalone-full.xml. Added both authentication and authorization sections.  
  Had to work out the correct rolesQuery and principalsQuery sql statements to use the database for authentication.  
  
        <security-domain name="dukes-forest" cache-type="default">
            <authentication>
                <login-module code="org.jboss.security.auth.spi.DatabaseServerLoginModule" flag="required">
                    <module-option name="dsJndiName" value="java:jboss/ForestXADS"/>
                    <module-option name="rolesQuery" value="select NAME as 'ROLES', 'Roles' as 'ROLEGROUP' from forest.GROUPS g inner join forest.PERSON_GROUPS pg on g.ID = pg.GROUPS_ID join forest.PERSON p on p.EMAIL = pg.EMAIL where p.EMAIL = ?"/>
                    <module-option name="hashAlgorithm" value="MD5"/>
                    <module-option name="hashEncoding" value="HEX"/>
                    <module-option name="principalsQuery" value="select PASSWORD from forest.PERSON where EMAIL=?"/>
                </login-module>
            </authentication>
            <authorization>
                <policy-module code="org.jboss.security.auth.spi.DatabaseServerLoginModule" flag="required">
                    <module-option name="dsJndiName" value="java:jboss/ForestXADS"/>
                    <module-option name="rolesQuery" value="select NAME as 'ROLE', 'ROLES' as 'ROLEGROUP' from forest.GROUPS g inner join forest.PERSON_GROUPS pg on g.ID = pg.GROUPS_ID join forest.PERSON p on p.EMAIL = pg.EMAIL where p.EMAIL = ?"/>
                    <module-option name="hashAlgorithm" value="MD5"/>
                    <module-option name="hashEncoding" value="HEX"/>
                    <module-option name="principalsQuery" value="select PASSWORD from forest.PERSON where EMAIL=?"/>
                </policy-module>
            </authorization>
        </security-domain>

* Changed /src/main/java/webapp/WEB-INF/web.xml for dukes-shipment and dukes-shop projects 
  to use dukes-forest security domain for login.
  
        <!--Defining security constraint for type of roles available -->     
        <security-constraint>
            <web-resource-collection>
                <web-resource-name>Secure Pages</web-resource-name>
                <description/>
                <url-pattern>/admin/*</url-pattern>
            </web-resource-collection>
            <auth-constraint>
                <role-name>ADMINS</role-name>
            </auth-constraint>
        </security-constraint>
        <!--Defining security constraint for type of roles available -->
        <!--Defining type of authentication mechanism -->
        <login-config>
            <auth-method>FORM</auth-method>
            <realm-name>dukes-forest</realm-name>
            <form-login-config>
                <form-login-page>/login.xhtml</form-login-page>
                <form-error-page>/login.xhtml</form-error-page>
            </form-login-config>
        </login-config>
        <!--Defining type of authentication mechanism -->
        <!--Defining security role -->    
        <security-role>
            <role-name>ADMINS</role-name>
        </security-role>
        <!--Defining security role -->
  
* Changed /src/main/java/webapp/WEB-INF/web.xml for dukes-payment project 
  to security-contraints for login.
  
        <security-constraint>
            <web-resource-collection>
                <web-resource-name>Secure payment service</web-resource-name>
                    <description/>
                <url-pattern>/*</url-pattern>
                <http-method-omission>GET</http-method-omission>
            </web-resource-collection>
            <auth-constraint>
                <role-name>USERS</role-name>
            </auth-constraint>
        </security-constraint>
        <login-config>
            <auth-method>BASIC</auth-method>
        </login-config>
        <security-role>
            <role-name>USERS</role-name>
        </security-role>
  
* Changed entities/src/main/java/com/forest/entity/Person.java to make fetch of groupsList EAGER instead of LAZY, 
  which is was by default.  

        @ManyToMany(cascade = CascadeType.ALL, fetch=FetchType.EAGER )
        protected List<Groups> groupsList;

* Commented out the "data-source" from web.xml for dukes-store project.

* Changed com.forest.handlers.PaymentHandler to use "Basic " instead of "BASIC " for the "Authorization" header.
  See WildFly issue [HTTP Authentication Basic header is case sensitive](http://issues.jboss.org/browse/WFLY-5618).

* Added @JsonIgnore to com.forest.entity.CustomerOrder.setCustomer(Person person) because it was a duplicate 
  signature with CustomerOrder.setCustomer(Order order) and causing problems with JSON serialization. 
   
        @JsonIgnore
        public void setCustomer(Person person) {
            this.customer = (Customer) person;
        }

* Added fetch=FetchType.EAGER to orderDetailList in com.forest.entity.CustomerOrder because lazy initialization was
  failing when used CustomerOrder was being used as a service by dukes-shipping.
   
        @OneToMany(cascade = CascadeType.ALL, mappedBy = "customerOrder", fetch=FetchType.EAGER)
        
* Added code to com.forest.ejb.UserBean in dukes-store project to add USERS role to new users, otherwise new 
  users would not be able to sign in.
  
        Query createNamedQuery = getEntityManager().createNamedQuery("Groups.findByName");
        createNamedQuery.setParameter("name", "USERS");
        if (createNamedQuery.getResultList().size() > 0) {
            customer.getGroupsList().add( (Groups)createNamedQuery.getSingleResult());
            super.create(customer);
            return true;
        } else {
            return false;
        }
  
* Issues about upper/lower case table and field names in MySQL when running on UNIX. COnverted everything to UPPERCASE. Tried lowercase, but hiberate generated names in uppercase.