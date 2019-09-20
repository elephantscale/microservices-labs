# Spring Data REST with Mongodb

Here we will use Spring Data to create a restful web interface for a MongoDB

## Use Spring Initizr to create a new project

I have included a sample project output from Spring initialzr in this folder. However, it would be good to create your own.  You can do this in one of three ways:

 * From the Spring CLI utility
 * From STS or another IDE
 * From the web based Spring initializr at [start.spring.io](http://start.spring.io/)

You can call the group id `com.example` and the artifact id `accessing-mongodb-data-rest`.  You will need twooptional dependences which is Spring Data MongoDB, plus Rest Repositories:

1. Spring Data MongoDB
2. Rest Respositories

Load the result in your favorite IDE such as STS or Intellij.

## Create a Person POJO

Open up a new java class called `Person` in your IDE or text editor.  Store this in a new file called `Person.java`

Here is the contents of the file

```java
package com.example.accessingmongodbdatarest;

import org.springframework.data.annotation.Id;

public class Person {

	@Id private String id;

	private String firstName;
	private String lastName;

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
}
```

## Create a Person repository

Next, you need to create a simple repository, as the following listing (in `src/main/java/com/example/accessingmongodbdatarest/PersonRepository.java`) shows:

```java
package com.example.accessingmongodbdatarest;

import java.util.List;

import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.repository.query.Param;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource(collectionResourceRel = "people", path = "people")
public interface PersonRepository extends MongoRepository<Person, String> {

	List<Person> findByLastName(@Param("name") String name);

}

```

This repository is an interface and lets you perform various operations that involve Person objects. It gets these operations by extending MongoRepository, which in turn extends the PagingAndSortingRepository interface defined in Spring Data Commons.

At runtime, Spring Data REST automatically creates an implementation of this interface. Then it uses the @RepositoryRestResource annotation to direct Spring MVC to create RESTful endpoints at /people.

`@RepositoryRestResource` is not required for a repository to be exported. It is used only to change the export details, such as using /people instead of the default value of /persons.
Here you have also defined a custom query to retrieve a list of Person objects based on the lastName value. You can see how to invoke it further down in this guide.

By default, Spring Boot tries to connect to a locally hosted instance of MongoDB. Read the reference docs for how to point your application to an instance of MongoDB taht is hosted elsewhere.

`@SpringBootApplication` is a convenience annotation that adds all of the following:

 * `@Configuration`: Tags the class as a source of bean definitions for the application context.

 * `@EnableAutoConfiguration`: Tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings. For example, if spring-webmvc is on the classpath, this annotation flags the application as a web application and activates key behaviors, such as setting up a DispatcherServlet.

 * `@ComponentScan`: Tells Spring to look for other components, configurations, and services in the com/example package, letting it find the controllers.

The `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application. Did you notice that there was not a single line of XML? There is no web.xml file, either. This web application is 100% pure Java and you did not have to deal with configuring any plumbing or infrastructure.

## Build the Application JAR

You can run the application from the command line with Gradle or Maven. You can also build a single executable JAR file that contains all the necessary dependencies, classes, and resources and run that. Building an executable jar so makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you use Gradle, you can run the application by using ./gradlew bootRun. Alternatively, you can build the JAR file by using ./gradlew build and then run the JAR file, as follows:

```bash
java -jar build/libs/gs-accessing-mongodb-data-rest-0.1.0.jar
```

If you use Maven, you can run the application by using ./mvnw spring-boot:run. Alternatively, you can build the JAR file with ./mvnw clean package and then run the JAR file, as follows:

```bash
java -jar target/gs-accessing-mongodb-data-rest-0.1.0.jar
```

## Test the Application

Test the Application
Now that the application is running, you can test it. You can use any REST client you wish. The following examples use the *nix tool curl.

First you want to see the top level service, as the following example shows:

```bash
$ curl http://localhost:8080
{
  "_links" : {
    "people" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    }
  }
}
```
The preceding example provides a first glimpse of what this server has to offer. There is a people link located at http://localhost:8080/people. It has some options, such as ?page, ?size, and ?sort.

Spring Data REST uses the HAL format for JSON output. It is flexible and offers a convenient way to supply links adjacent to the data that is served.
When you use the people link, you see the Person records in the database (none at present):

```bash
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "page" : {
    "size" : 20,
    "totalElements" : 0,
    "totalPages" : 0,
    "number" : 0
  }
}
```
There are currently no elements and, hence, no pages. It is time to create a new Person!

If you run this guide multiple times, there may be leftover data. Refer to the MongoDB shell quick reference for commands to find and drop your database if you need a fresh start.
The following command creats a person named “Frodo Baggins”:

```bash
$ curl -i -X POST -H "Content-Type:application/json" -d "{  \"firstName\" : \"Frodo\",  \"lastName\" : \"Baggins\" }" http://localhost:8080/people
HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Location: http://localhost:8080/people/53149b8e3004990b1af9f229
Content-Length: 0
Date: Mon, 03 Mar 2014 15:08:46 GMT
```

 * `-i`: Ensures you can see the response message including the headers. The URI of the newly created Person is shown.

 * `-X POST`: Signals this a POST used to create a new entry.

 * `-H "Content-Type:application/json"`: Sets the content type so the application knows the payload contains a JSON object.

 * `-d '{ "firstName" : "Frodo", "lastName" : "Baggins" }'`: Is the data being sent.

Notice how the previous POST operation includes a Location header. This contains the URI of the newly created resource. Spring Data REST also has two methods (`RepositoryRestConfiguration.setReturnBodyOnCreate(…)` and `setReturnBodyOnUpdate(…))` that you can use to configure the framework to immediately return the representation of the resource just created/updated.

From this you can query for all people, as the following example shows:

```bash
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "_embedded" : {
    "persons" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
        }
      }
    } ]
  },
  "page" : {
    "size" : 20,
    "totalElements" : 1,
    "totalPages" : 1,
    "number" : 0
  }
}
```
The persons object contains a list with Frodo. Notice how it includes a self link. Spring Data REST also uses the Evo Inflector to pluralize the names of entities for groupings.

You can directly query for the individual record, as the following example shows:

```bash
$ curl http://localhost:8080/people/53149b8e3004990b1af9f229
{
  "firstName" : "Frodo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
    }
  }
}
```

This might appear to be purely web-based, but, behind the scenes, it is talking to the MongoDB database you started.
In this guide, there is only one domain object. With a more complex system, where domain objects are related to each other, Spring Data REST renders additional links to help navigate to connected records.

Find all the custom queries, as the following example shows:

```bash
$ curl http://localhost:8080/people/search
{
  "_links" : {
    "findByLastName" : {
      "href" : "http://localhost:8080/people/search/findByLastName{?name}",
      "templated" : true
    }
  }
}
```

You can see the URL for the query, including the HTTP query parameter, name. This matches the `@Param("name")` annotation embedded in the interface.

To use the `findByLastName` query, run the following command:

```bash
$ curl http://localhost:8080/people/search/findByLastName?name=Baggins
{
  "_embedded" : {
    "persons" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
        }
      }
    } ]
  }
}
```

Because you defined it to return List<Person> in the code, it returns all of the results. If you had defined it to return only Person, it picks one of the Person objects to return. Since this can be unpredictable, you probably do not want to do that for queries that can return multiple entries.

You can also issue PUT, PATCH, and DELETE REST calls to replace, update, or delete existing records, respectively. The following example uses a PUT call:

```bash
$ curl -X PUT -H "Content-Type:application/json" -d "{ \"firstName\": \"Bilbo\", \"lastName\": \"Baggins\" }" http://localhost:8080/people/53149b8e3004990b1af9f229
$ curl http://localhost:8080/people/53149b8e3004990b1af9f229
{
  "firstName" : "Bilbo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
    }
  }
}
```

The following example uses a PATCH call:

```bash
$ curl -X PATCH -H "Content-Type:application/json" -d "{ \"firstName\": \"Bilbo Jr.\" }" http://localhost:8080/people/53149b8e3004990b1af9f229
$ curl http://localhost:8080/people/53149b8e3004990b1af9f229
{
  "firstName" : "Bilbo Jr.",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
    }
  }
}
```

PUT replaces an entire record. Fields not supplied will be replaced with null. You can use PATCH to update a subset of items.
You can also delete records, as the following example shows:

```bash
$ curl -X DELETE http://localhost:8080/people/53149b8e3004990b1af9f229
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "page" : {
    "size" : 20,
    "totalElements" : 0,
    "totalPages" : 0,
    "number" : 0
  }
}
```
A convenient aspect of this hypermedia-driven interface is how you can discover all the RESTful endpoints by using curl (or whatever REST client you like). There is no need to exchange a formal contract or interface document with your customers.
