# ReplatformingWorkshop
This material covers the steps to replatform a Struts App called "Movie Fun" to run on Pivotal Cloud Foundry

# Prerequisites
To get started you will need the following prerequisites on your machine.

- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
- [Maven](https://maven.apache.org/download.cgi)
- [Cloud Foundry CLI](https://github.com/cloudfoundry/cli#downloads)
- A Java IDE (e.g. [IntelliJ](https://www.jetbrains.com/idea/download/))

These labs will be much easier to complete if you use a Java IDE (as opposed to a text editor). All Java IDEs have the ability to auto-import dependencies which provides a much smoother development process. These labs do not provide instructions on imports, so if you must use a text editor you will need to refer to the code solutions often for direction on imports.

# 1 - Get the Movie Fun application
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
Email> John.Smith@nationwide.com
Password> enter_your_SSO_password
````
Ask your facilitator for the appropriate username, org, and space.
Note: if your PCF installation does not have a CA cert you will need to use ````cf api api.sys.testpcf.nwie.net --skip-ssl-validation````.

