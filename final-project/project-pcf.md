## Final Project

This lab is intended to be used for those interested in PCF / PKS and Java with Spring Boot

## Setup

Please make sure that you have installed the following:

 1. IDE of choice (STS, VS Code, Intellij, etc)
 2. MongoDB Desktop (or other database of choice)

## Write the Code

Use Spring Initializr to create a new project.  Feel free to use one of the pre-existing sample projects such as limits-service, etc.

Test the application to make sure it works.

## Deploy the Code to PCF

Use the `cf` command to deploy your code to PCF.  Test the app and make sure it works on PCF.

As we have very limited memory on PCF, shut down when finished.

## Make a docker container 

Make a docker file as follows (or something like this)

```dockerfile
FROM java:8
VOLUME /tmp
ADD target/YOURJARNAMEHERE-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
RUN bash -c ‘touch /app.jar’
ENTRYPOINT [“java”,””, “”,”-jar”,”/app.jar”]
```

build the docker container and then run it. test it


## Push your docker container to public Docker Hub

Log into your personal Docker Hub account.  This will be the same that you set up when you installed docker.  If you do not have one, set up your own Docker Hub account.

Use `docker push` to push the container to Docker Hub.

## Deploy the Docker container to PCF

Use the `cf tool to push the container to PCF.

Please refer to the PCF Documentation:

https://docs.cloudfoundry.org/devguide/deploy-apps/push-docker.html 

Something like this will work:

```bash
cf push my-app --docker-image my-docker-hub-id/my-app-name
```

Test access to the PCF app and make sure it does what you expect.

## Add MongoDB supotrt to Application

Install and run mongodb locally on your computer.  

Create a new project with spring initializr using the spring-data-mongo or modify your existing pom.xml in your IDE to include the spring data mongo.


Also add the following to your pom.xml:

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.11.0</version>
</dependency>
```


## Modify code and Test it

Write code that uses Spring Data

Test the service connecting to your (local) mongo

Open MongoDB Compass and make sure you see the data there.


## Run the MongoDB container instead of a local mongodb

Stop the running mongodb instance on your desktop.


Add the following to your application.properties file in the `resources` folder:

```text
dockerspring.data.mongodb.uri= mongodb://mongo:27000/test
```

Now run mongodb as a docker component


```bash
docker run -d -p 27000:27017 --name mongo mongo
```

You can now run your spring boot this way:

```bash
docker run -p 8080:8080 --name springboot-mongo --link=mongo  springboot-mongo
```

Modify the Docker file as follows. Make sure that you replace YOURJARNAMEHERE with the correct file name.

```dockerfile
FROM java:8
VOLUME /tmp
ADD target/YOURJARNAMEHERE-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
RUN bash -c ‘touch /app.jar’
ENTRYPOINT [“java”,”-Dspring.data.mongodb.uri=mongodb://mongo/test”, “-Djava.security.egd=file:/dev/./urandom”,”-jar”,”/app.jar”]
```

## Test your application on your Docker Desktop

Make sure that this is working.

## Deploy your application to PCF

You will be deploying two containers to PCF. First do the data component.  Note we are just deploying an unmodified mongo container.

Here is the link to the [Official MongoDB container on Docker Hub](https://hub.docker.com/_/mongo)

```bash
cf push my-app --docker-image mongo
```

Also, delete your old application on PCF and re-push with the new code to Docker Hub.  Then, go ahead and re-deploy.


## BONUS: Optional: Deploy Application to Kubernetes

Optionally, we can take and deploy our application to Kubernetes.  To do this, we need to define the following Kubernetes objects

 * Deployment: To control the deployment
 * Pods: mongo, application
 * Service (for the front end application)
 * Service (for Mongo)
 * Ingress (to allow an ingress endpoint for hte application)



