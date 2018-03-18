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
The examples shown below, a Windows client and a MacOS client, indicate that both systems have met the first three prerequisites. You should execute the same commands on your machine and correct any errors before proceeding.

![](https://github.com/rm511130/ReplatformingWorkshop/blob/master/DOS.jpg)

![](https://github.com/rm511130/ReplatformingWorkshop/blob/master/Mac.jpg)

# 1 - Get the "Movie Fun" App
Clone the Movie Fun application from GitHub.
````
mkdir ~/workspace
cd ~/workspace
git clone https://github.com/rm511130/replatforming-workshop-code
cd replatforming-workshop-code
````
If you are unable to clone from Git, you may download the course [here](https://github.com/rm511130/ReplatformingWorkshop/blob/master/replatforming-workshop-code-master.zip).

# 2 - Connect to Cloud Foundry
Once you have installed the Cloud Foundry CLI you will need to log in to the correct organization and space for this lab with the following command
````
cf api api.sys.testpcf.nwie.net 
cf login
Email> Your.Email@nationwide.com
Password> enter_your_SSO_password
````
Ask your facilitator for the appropriate username, org, and space.
Note: if your PCF installation does not have a CA cert you will need to use ````cf api api.sys.testpcf.nwie.net --skip-ssl-validation````.

# 3 - Apps Manager
Throughout the execution of this workshop, you are encouraged to take a look at the statics and health of your "Movie Fun" App by accessing the [Apps Manager GUI](https://login.sys.testpcf.nwie.net).

# 4 - Deploy "Movie Fun" using the TomEE Buildpack
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
# 5 - "Movie Fun" on Cloud Foundry
Once your app is deployed, a route to your running application will be displayed.

- Visit the deployed app and navigate to the setup page, which will populate the database with movies.
- Navigate to the main app to view these movies.
- Now re-stage your application by executing the following command:
````
cf restage movie-fun
````
- Refresh the movie list page in your browser. What do you notice?
- The movie list page is now blank because we are using an in-memory [HSQL database](http://hsqldb.org//). 
- To persist our data we will bind our app to a [MySQL database](https://www.mysql.com/).

# 5 - Create a MySQL binding & re-stage the "Movie Fun" App
Your Cloud Foundry installation comes with several on-demand services which you can use:
````
cf marketplace
````
Lets use the MySQL service by creating an instance and binding to it:
````
cf create-service cleardb spark my-mysql
cf bind-service movie-fun my-mysql
cf restage movie-fun
````
Refresh the movie list page again, and notice that our data is now persisted.

# 6 - A Word of Caution
Our "Movie Fun" App is now working, but we should test it thoroughly because the TomEE Buildpack is a community-developed buildpack which has no official support from Pivotal. As bug fixes or security fixes are needed, Pivotal commits to updating the officially supported buildpacks and pushing changes to PCF operators. Community buildpacks may be slow to get similar updates, or may not get them at all.

In the next lab you will take advantage of the officially supported Java buildpack and Spring's seamless Cloud Foundry integration.

# 7 - Clean up
Before moving on, it is very important to remember to clean up your Cloud Foundry space because you are using a shared PCF environment where resources are not free. You can clean up by using your [Apps Manager GUI](https://login.sys.testpcf.nwie.net) or by using the following commands:
````
cf delete movie-fun
cf delete-service my-mysql
````

