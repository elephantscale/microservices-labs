
## Use Spring Initizr to create a new project

I have included a sample project output from Spring initialzr in this folder. However, it would be good to create your own.  You can do this in one of three ways:

 * From the Spring CLI utility
 * From STS or another IDE
 * From the web based Spring initializr at [start.spring.io](http://start.spring.io/)

You can call the group id `com.example` and the artifact id `accessing-data-mongdb`.  You will need just one optional dependency which is Spring Data MongoDB.

## Create a new entity

In your IDE, add a new java class called Customer.  Put in a file called `Customer.java`.

Here is the class.

```java
package com.example.accessingdatamongodb;

import org.springframework.data.annotation.Id;


public class Customer {

    @Id
    public String id;

    public String firstName;
    public String lastName;

    public Customer() {}

    public Customer(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return String.format(
                "Customer[id=%s, firstName='%s', lastName='%s']",
                id, firstName, lastName);
    }

}
```

Here you have a `Customer` class with three attributes: `id`, `firstName`, and `lastName`. The id is mostly for internal use by MongoDB. You also have a single constructor to populate the entities when creating a new instance.

In this guide, the typical getters and setters have been left out for brevity.

`id` fits the standard name for a MongoDB ID, so it does not require any special annotation to tag it for Spring Data MongoDB.

The other two properties, firstName and lastName, are left unannotated. It is assumed that they are mapped to fields that share the same name as the properties themselves.

The convenient toString() method prints out the details about a customer.

MongoDB stores data in collections. Spring Data MongoDB maps the `Customer` class into a collection called `customer`. If you want to change the name of the collection, you can use Spring Data MongoDB’s @Document annotation on the class.

## Create Simple Query

```java

package com.example.accessingdatamongodb;

import java.util.List;

import org.springframework.data.mongodb.repository.MongoRepository;

public interface CustomerRepository extends MongoRepository<Customer, String> {

    public Customer findByFirstName(String firstName);
    public List<Customer> findByLastName(String lastName);

}
```


## Modify Application Class

Spring Initializr should have created a stub application class that looks something like this:

```java
package com.example.accessingdatamongodb;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AccessingDataMongodbApplication {

	public static void main(String[] args) {
		SpringApplication.run(AccessingDataMongodbApplication.class, args);
	}

}

```

You should modify this class `AccessingDataMongodbApplication` in order to access Spring Data and Mongodb.

```java
package com.example.accessingdatamongodb;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AccessingDataMongodbApplication implements CommandLineRunner {

	@Autowired
	private CustomerRepository repository;

	public static void main(String[] args) {
		SpringApplication.run(AccessingDataMongodbApplication.class, args);
	}

	@Override
	public void run(String... args) throws Exception {

		repository.deleteAll();

		// save a couple of customers
		repository.save(new Customer("Alice", "Smith"));
		repository.save(new Customer("Bob", "Smith"));

		// fetch all customers
		System.out.println("Customers found with findAll():");
		System.out.println("-------------------------------");
		for (Customer customer : repository.findAll()) {
			System.out.println(customer);
		}
		System.out.println();

		// fetch an individual customer
		System.out.println("Customer found with findByFirstName('Alice'):");
		System.out.println("--------------------------------");
		System.out.println(repository.findByFirstName("Alice"));

		System.out.println("Customers found with findByLastName('Smith'):");
		System.out.println("--------------------------------");
		for (Customer customer : repository.findByLastName("Smith")) {
			System.out.println(customer);
		}

	}

}
```


## Build your JAR file

You can run the application from the command line with Gradle or Maven. You can also build a single executable JAR file that contains all the necessary dependencies, classes, and resources and run that. Building an executable jar so makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you use Gradle, you can run the application by using `./gradlew bootRun`. Alternatively, you can build the JAR file by using `./gradlew build` and then run the JAR file, as follows:

```bash
java -jar build/libs/gs-accessing-data-mongodb-0.1.0.jar
```

If you use Maven, you can run the application by using `./mvnw spring-boot:run`. Alternatively, you can build the JAR file with `./mvnw clean package` and then run the JAR file, as follows:

```java
java -jar target/gs-accessing-data-mongodb-0.1.0.jar
```


## Run the App and observe the ouput

You should get output like this:

```console
== Customers found with findAll():
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
Customer[id=51df1b0a3004cb49c50210f9, firstName='Bob', lastName='Smith']

== Customer found with findByFirstName('Alice'):
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
== Customers found with findByLastName('Smith'):
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
Customer[id=51df1b0a3004cb49c50210f9, firstName='Bob', lastName='Smith']
```
