# Replatforming Workshop
Movie Fun is a simple movie catalog management application. It is currently implemented using Java Enterprise. It was designed to run on TomEE, and uses EJBs and Servlets. Database configuration is done through JNDI.

In this workshop we will first try to deploy Movie Fun to Cloud Foundry using the TomEE buildpack. We will then replatform Movie Fun by wrapping it in a Spring Boot container. By the end of the lab we will have a cloud-ready, working application deployed to Cloud Foundry.

![](https://github.com/rm511130/ReplatformingWorkshop/blob/master/ReplatformNModernize.jpg)

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
Open a Terminal Window on your local machine, then create a workspace directory, and navigate to it.
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

# 7 - Create a MySQL binding & restage "Movie Fun"
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
This will allow Spring to find our [Java Server Pages](https://en.wikipedia.org/wiki/JavaServer_Pages) in the ````WEB-INF```` folder.

- Finally, move ````index.jsp```` from the ````webapp/```` folder into the ````webapp/WEB-INF/```` folder.

- Run the app to test that it still runs locally:
````
mvn spring-boot:run
````
You will be able to navigate the [home page](http://localhost:8080), but other pages will not work yet. We will get the rest working in the upcoming labs.

![](https://github.com/rm511130/ReplatformingWorkshop/blob/master/localhost8080.jpg)

# 14 - Make MoviesBean injectable
Let's add more functionality to our Spring Boot app by mapping the setup endpoint. Begin by making the ````MoviesBean```` injectable.

- Remove the persistence unit specification from the ````@PersistenceContext```` annotation.

- Replace ````@Stateless```` with ````@Repository````:
````
- @Stateless
+ @Repository
public class MoviesBean {

-    @PersistenceContext(unitName = "movie-unit")
+    @PersistenceContext
    private EntityManager entityManager;

    //...
}
````
MovieBeans.java should now look like this:
###### _src/main/java/org/superbiz/moviefun/MoviesBean.java_
````
src/main/java/org/superbiz/moviefun/MoviesBean.java
/**
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 * <p/>
 * http://www.apache.org/licenses/LICENSE-2.0
 * <p/>
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package org.superbiz.moviefun;

import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.TypedQuery;
import javax.persistence.criteria.*;
import javax.persistence.metamodel.EntityType;
import java.util.List;

@Repository
public class MoviesBean {

    @PersistenceContext
    private EntityManager entityManager;

    public Movie find(Long id) {
        return entityManager.find(Movie.class, id);
    }

    public void addMovie(Movie movie) {
        entityManager.persist(movie);
    }

    public void editMovie(Movie movie) {
        entityManager.merge(movie);
    }

    public void deleteMovie(Movie movie) {
        entityManager.remove(movie);
    }

    public void deleteMovieId(long id) {
        Movie movie = entityManager.find(Movie.class, id);
        deleteMovie(movie);
    }

    public List<Movie> getMovies() {
        CriteriaQuery<Movie> cq = entityManager.getCriteriaBuilder().createQuery(Movie.class);
        cq.select(cq.from(Movie.class));
        return entityManager.createQuery(cq).getResultList();
    }

    public List<Movie> findAll(int firstResult, int maxResults) {
        CriteriaQuery<Movie> cq = entityManager.getCriteriaBuilder().createQuery(Movie.class);
        cq.select(cq.from(Movie.class));
        TypedQuery<Movie> q = entityManager.createQuery(cq);
        q.setMaxResults(maxResults);
        q.setFirstResult(firstResult);
        return q.getResultList();
    }

    public int countAll() {
        CriteriaQuery<Long> cq = entityManager.getCriteriaBuilder().createQuery(Long.class);
        Root<Movie> rt = cq.from(Movie.class);
        cq.select(entityManager.getCriteriaBuilder().count(rt));
        TypedQuery<Long> q = entityManager.createQuery(cq);
        return (q.getSingleResult()).intValue();
    }

    public int count(String field, String searchTerm) {
        CriteriaBuilder qb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Long> cq = qb.createQuery(Long.class);
        Root<Movie> root = cq.from(Movie.class);
        EntityType<Movie> type = entityManager.getMetamodel().entity(Movie.class);

        Path<String> path = root.get(type.getDeclaredSingularAttribute(field, String.class));
        Predicate condition = qb.like(path, "%" + searchTerm + "%");

        cq.select(qb.count(root));
        cq.where(condition);

        return entityManager.createQuery(cq).getSingleResult().intValue();
    }

    public List<Movie> findRange(String field, String searchTerm, int firstResult, int maxResults) {
        CriteriaBuilder qb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Movie> cq = qb.createQuery(Movie.class);
        Root<Movie> root = cq.from(Movie.class);
        EntityType<Movie> type = entityManager.getMetamodel().entity(Movie.class);

        Path<String> path = root.get(type.getDeclaredSingularAttribute(field, String.class));
        Predicate condition = qb.like(path, "%" + searchTerm + "%");

        cq.where(condition);
        TypedQuery<Movie> q = entityManager.createQuery(cq);
        q.setMaxResults(maxResults);
        q.setFirstResult(firstResult);
        return q.getResultList();
    }

    public void clean() {
        entityManager.createQuery("delete from Movie").executeUpdate();
    }
}
````
If you are not familiar with dependency injection in Spring, now is a good time to read [this](http://docs.spring.io/autorepo/docs/spring-boot/current/reference/html/using-boot-spring-beans-and-dependency-injection.html). Note that as of Spring 4.3, you no longer need to specify an explicit injection annotation on beans with a single constructor, so we will omit such annotations in our labs.

# 15 - Update HomeController
Now enable the ````HomeController```` to setup the application using our newly-configured ````MoviesBean````.

- Inject the ````MoviesBean```` into the ````HomeController````:
```java
private final MoviesBean moviesBean;

public HomeController(MoviesBean moviesBean) {
    this.moviesBean = moviesBean;
}
```
- Create the ````setup```` controller action in ````HomeController```` by adding the following method to the ````HomeController```` class:
```java
//...
@Transactional
@GetMapping("/setup")
public String setup(Map<String, Object> model) {
    moviesBean.addMovie(new Movie("Wedding Crashers", "David Dobkin", "Comedy", 7, 2005));
    moviesBean.addMovie(new Movie("Starsky & Hutch", "Todd Phillips", "Action", 6, 2004));
    moviesBean.addMovie(new Movie("Shanghai Knights", "David Dobkin", "Action", 6, 2003));
    moviesBean.addMovie(new Movie("I-Spy", "Betty Thomas", "Adventure", 5, 2002));
    moviesBean.addMovie(new Movie("The Royal Tenenbaums", "Wes Anderson", "Comedy", 8, 2001));
    moviesBean.addMovie(new Movie("Zoolander", "Ben Stiller", "Comedy", 6, 2001));
    moviesBean.addMovie(new Movie("Shanghai Noon", "Tom Dey", "Comedy", 7, 2000));

    model.put("movies", moviesBean.getMovies());

    return "setup";
}
//...
```
Make sure to import ````org.springframework.transaction.annotation.Transactional```` for the ````@Transactional```` annotation.

The end result of these changes should look like this:

###### _src/main/java/org/superbiz/moviefun/HomeController.java_
```java
package org.superbiz.moviefun;

import org.springframework.stereotype.Controller;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.Map;

@Controller
public class HomeController {

    private final MoviesBean moviesBean;

    public HomeController(MoviesBean moviesBean) {
        this.moviesBean = moviesBean;
    }

    @GetMapping("/")
    public String index() {
        return "index";
    }

    @Transactional
    @GetMapping("/setup")
    public String setup(Map<String, Object> model) {
        moviesBean.addMovie(new Movie("Wedding Crashers", "David Dobkin", "Comedy", 7, 2005));
        moviesBean.addMovie(new Movie("Starsky & Hutch", "Todd Phillips", "Action", 6, 2004));
        moviesBean.addMovie(new Movie("Shanghai Knights", "David Dobkin", "Action", 6, 2003));
        moviesBean.addMovie(new Movie("I-Spy", "Betty Thomas", "Adventure", 5, 2002));
        moviesBean.addMovie(new Movie("The Royal Tenenbaums", "Wes Anderson", "Comedy", 8, 2001));
        moviesBean.addMovie(new Movie("Zoolander", "Ben Stiller", "Comedy", 6, 2001));
        moviesBean.addMovie(new Movie("Shanghai Noon", "Tom Dey", "Comedy", 7, 2000));

        model.put("movies", moviesBean.getMovies());

        return "setup";
    }
}
```

# 16 - Prepare setup.jsp
We will now remove setup logic from the `setup.jsp` view since it is now taken care of in the `HomeController`.

- Move `setup.jsp` from the `webapp/` folder into the `webapp/WEB-INF/` folder.

- Next, we will make the following changes to `setup.jsp`
    - Remove all references to `MoviesBean`.
    - Update the iteration over `movies` to use `movies` on the `requestScope`.
    - Clean up some unnecessary code.

- Copy the solution below into your `setup.jsp` to make these changes.

###### _src/main/webapp/WEB-INF/setup.jsp_
```html
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/functions" prefix="fn" %>
<c:set var="language" value="${pageContext.request.locale}"/>
<fmt:setLocale value="${language}"/>

<!DOCTYPE html>
<html lang="${language}">
<head>
    <meta charset="utf-8">
    <title>Moviefun</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <link href="../assets/css/bootstrap.css" rel="stylesheet">
    <link href="../assets/css/movie.css" rel="stylesheet">
    <style>
        body {
            padding-top: 60px;
        }
    </style>
    <link href="../assets/css/bootstrap-responsive.css" rel="stylesheet">
</head>
<body>

<div class="navbar navbar-inverse navbar-fixed-top">
    <div class="navbar-inner">
        <div class="container">
            <a class="btn btn-navbar" data-toggle="collapse"
               data-target=".nav-collapse"> <span class="icon-bar"></span> <span
                    class="icon-bar"></span> <span class="icon-bar"></span>
            </a> <a class="brand" href="#">Moviefun</a>
        </div>
    </div>
</div>

<div class="container">

    <h1>Moviefun</h1>

    <h2>Seeded Database with the Following movies</h2>
    <table width="500">
        <tr>
            <td><b>Title</b></td>
            <td><b>Director</b></td>
            <td><b>Genre</b></td>
        </tr>
        <c:forEach items="${requestScope.movies}" var="movie">
            <tr>
                <td>${ movie.title }</td>
                <td>${ movie.director }</td>
                <td>${ movie.genre }</td>
            </tr>
        </c:forEach>
    </table>

    <h2>Continue</h2>
    <a href="moviefun">Go to main app</a>
</div>
</body>
</html>
```

- Now update `index.jsp` to use the new route:
```html
- <a href="setup.jsp">Setup</a> - Sets up the application with some sample data<br/>
+ <a href="setup">Setup</a> - Sets up the application with some sample data<br/>
```

- Run the App
````
mvn spring-boot:run
````
Test that the app still runs locally. You will be able to navigate the home page and the setup page, but the movies list page still returns a 404.



