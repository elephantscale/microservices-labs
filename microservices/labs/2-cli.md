# Spring CLI


## Step 1: Install Spring Boot CLI tool

Install the Spring Boot command-line tool by downloading the CLI tool located [here](http://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.1.8.RELEASE/spring- boot-cli-2.1.8.RELEASE-bin.zip).

Unzip the file into a directory of your choice. Open a terminal window and change the terminal prompt to the `bin` folder.

Ensure the `.bin` folder is added to the system path so that Spring Boot can be run frm any location.

Verify the installation with the following command. If successful, the Spring CLI version will be printed in the console:

```java
 $ spring –-version

Spring CLI v2.1.8.RELEASE
```

As the next step, a quick REST service will be developed in Groovy, which is supported out of the box in Spring Boot. To do so, copy and paste the following code using any editor of

```java
@RestController
class HelloworldController {
    @RequestMapping("/")
    String sayHello() {
      "Hello World!"
    }
}
```

In order to run this Groovy application, go to the folder where `myfirstapp.groovy` is saved and execute the following command. The last few lines of the server start-up log will be similar to the following:

  
```console
 $ spring run myfirstapp.groovy

2016-05-09 18:13:55.351  INFO 35861 --- [nio-8080-exec-1]
o.s.web.servlet.DispatcherServlet        : FrameworkServlet
'dispatcherServlet': initialization started
2016-05-09 18:13:55.375  INFO 35861 --- [nio-8080-exec-1]
o.s.web.servlet.DispatcherServlet        : FrameworkServlet
'dispatcherServlet': initialization completed in 24 ms
```

Open a browser window and go to `http://localhost:8080` ; the browser will display
the following message:

```console
Hello World!
```

There is no `.war` file created, and no Tomcat server was run. Spring Boot automatically picked up Tomcat as the webserver and embedded it into the application. This is a very basic, minimal microservice. The `@RestController` annotation, used in the previous code, will be examined in detail in the next example.
   
