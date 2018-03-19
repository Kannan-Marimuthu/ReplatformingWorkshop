# Replatforming Workshop
This material covers the steps to replatform a Struts App called "Movie Fun" to run on Pivotal Cloud Foundry

# Prerequisites
To get started you will need the following prerequisites on your machine.

- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) & check the JAVA_HOME environment variable
- [Maven](https://maven.apache.org/download.cgi) & follow the [installation instructions](https://maven.apache.org/install.html)
- [Cloud Foundry CLI](https://github.com/cloudfoundry/cli#downloads)
- [Git](https://git-scm.com/downloads)
- A Java IDE (e.g. [Eclipse Neon](http://www.eclipse.org/downloads/packages/release/Neon/3) or [IntelliJ](https://www.jetbrains.com/idea/download/))

These labs will be much easier to complete if you use a Java IDE (as opposed to a text editor). All Java IDEs have the ability to auto-import dependencies which provides a much smoother development process. These labs do not provide instructions on imports, so if you must use a text editor you will need to refer to the code solutions often for direction on imports.

# Checking Prerequisites
The examples shown below, both for a Windows client and a MacOS client, show client systems have met the first four prerequisites. You should execute the same commands on your machine and correct any errors before proceeding.

![](https://github.com/rm511130/ReplatformingWorkshop/blob/master/DOS.jpg)

![](https://github.com/rm511130/ReplatformingWorkshop/blob/master/Mac.jpg)

# 1 - Connect to Cloud Foundry
Once you have installed the Cloud Foundry CLI you will need to log in to the correct organization and space for this lab with the following command
````
cf api api.sys.testpcf.nwie.net 
cf login
Email> Your.Email@nationwide.com
Password> enter_your_SSO_password
````
Ask your facilitator for the appropriate username, org, and space.
Note: if your PCF installation does not have a CA cert you will need to use ````cf api api.sys.testpcf.nwie.net --skip-ssl-validation````.

# 2 - Apps Manager
Throughout the execution of this workshop, you are encouraged to take a look at the statics and health of your "Movie Fun" App by accessing the [Apps Manager GUI](https://login.sys.testpcf.nwie.net).

# 3 - Let's Start with a simple _cf push_
First you need to open a Command Window on your Windows Machine or a Terminal on your Mac, and then create a workspace directory/folder and navigate to it.
````
mkdir workspace
cd workspace
````
Clone the Chess App from GitHub and _cf push_ it using the following commands:
````
git clone https://github.com/Pivotal-Field-Engineering/chess
cd chess
cf push
````
Once the _cf push_ has successfully finished, you should see an output similar to the example shown below. Look for the URL.

![](https://github.com/rm511130/ReplatformingWorkshop/blob/master/chess.jpg)

Open a browser and check whether your Chess App is running.

# 4 - Get the "Movie Fun" App
Clone the Movie Fun application from GitHub. Assuming you are still in the _chess_ directory, execute the following commands:
````
cd ..
git clone https://github.com/rm511130/replatforming-workshop-code
cd replatforming-workshop-code
````
If you are unable to clone from Git, you may download the course [here](https://github.com/rm511130/ReplatformingWorkshop/blob/master/replatforming-workshop-code-master.zip).

# 5 - Deploy "Movie Fun" using the TomEE Buildpack
The first thing to try with a Java Enterprise app is to deploy a warfile using a Java Enterprise compatible [buildpack](https://docs.cloudfoundry.org/buildpacks/). We will try the [TomEE buildpack](https://github.com/cloudfoundry-community/tomee-buildpack) for Movie Fun.

Take a look at the buildpack entry in the manifest.yml for how we specify the TomEE buildpack for deployment.

###### _manifest.yml_
````
name: movie-fun
random-route: true
path: target/moviefun.war
buildpack: https://github.com/cloudfoundry-community/tomee-buildpack.git
````
Build and deploy the application by running
````
mvn clean package -DskipTests -Dmaven.test.skip=true
cf push
````
# 6 - "Movie Fun" on Cloud Foundry
Once your app is deployed, a route to your running application will be displayed.

- Visit the deployed app and navigate to the setup page, which will populate the database with movies.
- Navigate to the main app to view these movies.
- Now stop and start your application by executing the following commands:
````
cf stop movie-fun
cf start movie-fun
````
- Refresh the movie list page in your browser. What do you notice?
- The movie list page is now blank because we are using an in-memory [HSQL database](http://hsqldb.org//). 
- To persist our data we will bind our app to a [MySQL database](https://www.mysql.com/).

# 7 - Create a MySQL binding & re-stage "Movie Fun"
Your Cloud Foundry installation comes with several on-demand services which you can use:
````
cf marketplace
````
Lets use the MySQL service by creating an instance and binding to it:
````
cf create-service p-mysql 100mb movie-mysql
cf bind-service movie-fun movie-mysql
cf restage movie-fun
````
Refresh the movie list page again, and notice that our data is now persisted even if you _cf stop_ and _cf start_ your App in the previous step of this workshop/lab.

Now take a look at the results of the following command so you can see how the _cf bind-service_ passes connection string information using environment variables:
````
cf env movie-fun
````

# 8 - A Word of Caution
Our "Movie Fun" App is now working, but we should test it thoroughly because the [TomEE buildpack](https://github.com/cloudfoundry-community/tomee-buildpack) is a community-developed buildpack which has no official support from Pivotal. As bug fixes or security fixes are needed, Pivotal commits to updating the officially supported buildpacks and pushing changes to PCF operators. Community buildpacks may be slow to get similar updates, or may not get them at all.

In the next lab you will take advantage of the officially supported Java buildpack and Spring's seamless Cloud Foundry integration.

# 9 - Clean up
Before moving on, it is very important to remember to clean up your Cloud Foundry space because you are using a shared PCF environment where resources are not free. You can clean up by using your [Apps Manager GUI](https://login.sys.testpcf.nwie.net) or by using the following commands:
````
cf delete chess
cf delete movie-fun
cf delete-service my-mysql
````

# 10 - Update dependencies
We will introduce the [Spring Boot](https://projects.spring.io/spring-boot/) dependencies that will be needed for the following labs. Along the way we will also clean up some unnecessary configurations.
- In pom.xml, remove tomee.version:
````
- <tomee.version>7.0.2</tomee.version>
````
- Add the [Spring Boot Maven plugin](http://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins-maven-plugin.html) inside the ````<build>```` tag:
  
````
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
</plugins>
````
This plugin provides Spring Boot support in Maven, allowing you to package executable jar or war archives and run an application locally.
- Add the [Spring Boot starter parent](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-maven) inside the ````<project>```` tag:
  
````
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.2.RELEASE</version>
</parent>
````
The parent will manage versions of all dependencies needed by Spring Boot. We only need to specify the Spring Boot version number on this dependency. When importing additional starters, we can safely omit the version number.
- Remove the version from ````javax.servlet````, as this is provided by ````spring-boot-starter-parent````.
````
- <version>1.2</version>
````
- Add the following dependencies after ````commons-lang````.
````
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
````
The dependency on [h2](http://www.h2database.com/html/main.html) (an in-memory database) is only introduced for the convenience of this lab. We would normally recommend using a MySQL database for local development in order to get our development environment as close to production as possible.

- Remove the ````spring-web```` dependency in test scope since it is now provided by ````spring-boot-starter-web````.
````
- <dependency>
-     <groupId>org.springframework</groupId>
-     <artifactId>spring-web</artifactId>
-     <version>4.3.4.RELEASE</version>
-     <scope>test</scope>
- </dependency>
````

- Your solution should look like this:


###### _pom.xml_
````
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>org.superbiz</groupId>
    <artifactId>moviefun</artifactId>
    <packaging>war</packaging>
    <version>1.1.0-SNAPSHOT</version>
    <name>OpenEJB :: Web Examples :: Moviefun</name>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <build>
        <finalName>moviefun</finalName>
        <defaultGoal>package</defaultGoal>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.2.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.apache.tomee</groupId>
            <artifactId>javaee-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.4</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
````

# 11 - Create the Application class
Next, create the ````Application```` class which will be the basis of our Spring Boot application.

###### _src/main/java/org/superbiz/moviefun/Application.java_
````
package org.superbiz.moviefun;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String... args) {
        SpringApplication.run(Application.class, args);
    }
}
````
If you are unfamiliar with Spring Boot, take a minute to read [this](https://spring.io/guides/gs/spring-boot/#_create_an_application_class) about the different pieces of the ````Application```` class.

# 12 - Create application.yml
Provide some configuration for our simple database setup in application.yml.

###### _src/main/resources/application.yml_
````
spring:
  jpa:
    generate-ddl: true
    properties.hibernate.dialect: org.hibernate.dialect.MySQL5Dialect
  datasource:
    url: jdbc:h2:${java.io.tmpdir}/movie-fun
    username: root
````
At this point, the Spring Boot application should start, but we have not yet configured any endpoints.

Start the app with
````
mvn spring-boot:run
````
Navigate to the home page at [localhost:8080](http://localhost:8080/) and check that it returns a 404.

![](https://github.com/rm511130/ReplatformingWorkshop/blob/master/404.jpg)

# 13 - Map the index page
We will use [Spring MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html) to re-configure our endpoints.

- First, create the ````HomeController```` class that will load our ````index.jsp````.

###### _src/main/java/org/superbiz/moviefun/HomeController.java_
````
package org.superbiz.moviefun;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String index() {
        return "index";
    }
}
````
If you are unfamiliar with Spring MVC, take some time to read [this](https://spring.io/guides/gs/serving-web-content/#initial) about Spring MVC controllers.

- Next, update application.yml with view rendering config, add:
````
spring:
  #...
  mvc.view:
    prefix: /WEB-INF/
    suffix: .jsp
````

````src/main/resources/application.yml```` should look like this now:
````
spring:
  jpa:
    generate-ddl: true
    properties.hibernate.dialect: org.hibernate.dialect.MySQL5Dialect
  datasource:
    url: jdbc:h2:${java.io.tmpdir}/movie-fun
    username: root
  mvc.view:
    prefix: /WEB-INF/
    suffix: .jsp
````
This will allow Spring to find our [jsps](https://en.wikipedia.org/wiki/JavaServer_Pages) in the ````WEB-INF```` folder.

- Finally, move ````index.jsp```` from the ````webapp/```` folder into the ````webapp/WEB-INF/```` folder.

- Run the app to test that it still runs locally:
````
mvn spring-boot:run
````
You will be able to navigate the home page, but other pages will not work yet. We will get the rest working in the upcoming labs.



