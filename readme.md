Copyright (c) 2014 Oracle and/or its affiliates. All rights reserved.

You may not modify, use, reproduce, or distribute this software except in
compliance with  the terms of the License at:
http://java.net/projects/javaeetutorial/pages/BerkeleyLicense

Please find the Oracle dukes-forest tutorial documentation at [Duke's Forest Case Study Example](https://docs.oracle.com/javaee/7/tutorial/dukes-forest.htm#GLNPW) 

# dukes-forest

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
  
* Changed entities/src/main/resources/META-INF/persistence.xml to use java:jboss/ForestDataSource instead of
  java:global/ForestDataSource. Created the appropriate datasource in Wildfly.
  
* Changed dukes-shipment/src/main/java/com.forest.shipment.ejb.OrderBrowser to use jboss compatible jndi name 
  for jms message queue. Created appropriate queue in Wildfly and made Eclipse run Wildfly with standalone-full.xml
  so that JMS services would be available.
  
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
                    <module-option name="dsJndiName" value="java:jboss/ForestDataSource"/>
                    <module-option name="rolesQuery" value="select name as 'roles', 'Roles' as 'rolegroup' from forest.groups g inner join forest.person_groups pg on g.id = pg.GROUPS_ID join forest.person p on p.EMAIL = pg.EMAIL where p.email = ?"/>
                    <module-option name="hashAlgorithm" value="MD5"/>
                    <module-option name="hashEncoding" value="HEX"/>
                    <module-option name="principalsQuery" value="SELECT password from forest.person WHERE email=?"/>
                </login-module>
            </authentication>
            <authorization>
                <policy-module code="org.jboss.security.auth.spi.DatabaseServerLoginModule" flag="required">
                    <module-option name="dsJndiName" value="java:jboss/ForestDataSource"/>
                    <module-option name="rolesQuery" value="select name as 'role', 'Roles' as 'rolegroup' from forest.groups g inner join forest.person_groups pg on g.id = pg.GROUPS_ID join forest.person p on p.EMAIL = pg.EMAIL where p.email = ?"/>
                    <module-option name="hashAlgorithm" value="MD5"/>
                    <module-option name="hashEncoding" value="HEX"/>
                    <module-option name="principalsQuery" value="SELECT password from forest.person WHERE email=?"/>
                </policy-module>
            </authorization>
        </security-domain>

* Added java:jboss/ForestDataSource datasource and mysql driver in standalone-full.xml.

        <datasource jta="true" jndi-name="java:jboss/ForestDataSource" pool-name="ForestDataSource" enabled="true" use-ccm="true">
            <connection-url>jdbc:mysql://localhost:3306/forest</connection-url>
            <driver-class>com.mysql.jdbc.Driver</driver-class>
            <driver>mysql</driver>
            <security>
                <user-name>forest</user-name>
                <password>forest</password>
            </security>
            <validation>
                <valid-connection-checker class-name="org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLValidConnectionChecker"/>
                <background-validation>true</background-validation>
                <exception-sorter class-name="org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLExceptionSorter"/>
            </validation>
        </datasource>
  
  and ..
  
        <driver name="mysql" module="com.mysql">
            <xa-datasource-class>
                com.mysql.jdbc.jdbc2.optional.MysqlXADataSource
            </xa-datasource-class>
        </driver>
  
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
  
  