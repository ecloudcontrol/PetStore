## Petstore Java EE 7

### Migration process

* Source branch: **master**
* Target branch: **appz**

1. Generated ```web/WEB-INF/resources.xml```with datasource configuration
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!--
        Licensed to the Apache Software Foundation (ASF) under one or more
        contributor license agreements.  See the NOTICE file distributed with
        this work for additional information regarding copyright ownership.
        The ASF licenses this file to You under the Apache License, Version 2.0
        (the "License"); you may not use this file except in compliance with
        the License.  You may obtain a copy of the License at
           http://www.apache.org/licenses/LICENSE-2.0
        Unless required by applicable law or agreed to in writing, software
        distributed under the License is distributed on an "AS IS" BASIS,
        WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
        See the License for the specific language governing permissions and
        limitations under the License.
    -->
    <resources>
      <Resource id="DefaultDataSource" type="javax.sql.DataSource">
        JdbcDriver com.mysql.jdbc.Driver
        JdbcUrl ${mysql.jdbc.url}
        UserName ${mysql.user}
        Password ${mysql.password}
      </Resource>
    </resources>
    ``` 

2. Modified pom.xml to support simulations build for tomcat and other servers both
    ```xml
      <properties>
          <mysql.jdbc.url>jdbc:mysql://mysql-5-7:3306/petstore</mysql.jdbc.url>
          <mysql.user>petstore</mysql.user>
          <mysql.password>${mysql.password}</mysql.password>
      </properties>
    ```
    
    Added profiles to support both builds
    ```xml
    <profiles>
        <profile>
          <id>tomcat</id>
          <properties>
            <mysql.jdbc.url>jdbc:mysql://mysql-tomcat:3306/default</mysql.jdbc.url>
            <mysql.user>petstore</mysql.user>
            <mysql.password>${mysql.password}</mysql.password>
          </properties>
          <build>
            <finalName>applicationPetstore-tomcat</finalName>
          </build>
        </profile>
        <profile>
          <id>websphere</id>
          <properties>
            <jta.datasource.name>DefaultDataSource</jta.datasource.name>
            <mysql.jdbc.url>jdbc:mysql://localhost:3307/default</mysql.jdbc.url>
            <mysql.user>petstore</mysql.user>
            <mysql.password>${mysql.password}</mysql.password>
          </properties>
          <build>
            <finalName>applicationPetstore-websphere</finalName>
          </build>
        </profile>
        <profile>
          <id>appz-k8s</id>
          <properties>
            <mysql.jdbc.url>jdbc:mysql://mysql-5-7:3306/petstore</mysql.jdbc.url>
            <mysql.user>petstore</mysql.user>
            <mysql.password>${mysql.password}</mysql.password>
          </properties>
          <build>
            <finalName>applicationPetstore-appz-k8s</finalName>
          </build>
        </profile>
      </profiles>
    ```
    
5. Generated ```docker/tomcat/conf/conf.d``` with following files:
    ```
        /PetStore/docker/tomcat/conf/catalina.policy
        /PetStore/docker/tomcat/conf/catalina.properties
        /PetStore/docker/tomcat/conf/context.xml
        /PetStore/docker/tomcat/conf/jaspic-providers.xml
        /PetStore/docker/tomcat/conf/jaspic-providers.xsd
        /PetStore/docker/tomcat/conf/logging.properties
        /PetStore/docker/tomcat/conf/server.xml
        /PetStore/docker/tomcat/conf/server.xml.original
        /PetStore/docker/tomcat/conf/system.properties
        /PetStore/docker/tomcat/conf/tomcat-users.xml
        /PetStore/docker/tomcat/conf/tomcat-users.xml.original
        /PetStore/docker/tomcat/conf/tomcat-users.xsd
        /PetStore/docker/tomcat/conf/tomee.xml
        /PetStore/docker/tomcat/conf/web.xml
    ```
5. Build samples:
    * ```mvn clean```
    * ```mvn package``` — default to tomcat (custom)
    * ```mvn package -Ptomcat``` — default to tomcat (default)
    * ```mvn package -Pwebsphere``` — default to websphere
6. When using tomcat, pack war inside docker image or mount it like a volume. 
   Deploy it somewhere using Docker or Kubernetes. 
   
   When using WebSphere, deploy war manually on the server node.

7. Application url example: http://example.com/applicationPetstore

8. Generated [appz.yml](appz.yml) 
