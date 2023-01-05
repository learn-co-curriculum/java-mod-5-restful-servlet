# Completing the RESTful Servlet

## Learning Goals

- Override the `doPut()` method to update a resource
- Override the `doDelete()` method to delete a resource

## Code Along

## Introduction

Recall that RESTful routes fall into four familiar categories: Create, Read, Update, and Delete.
Let's update the `ContinentServlet` to implement the four categories and produce
a simple RESTful API.

The table below shows the HTTP request method and route we will use for this
RESTful app:

| HTTP Request Method | Route                          | Description                  |
|---------------------|--------------------------------|------------------------------|
| POST                | `/continents/`                 | Create a new continent.      |
| GET                 | `/continents/{continent_name}` | Show a specific continent.   |
| PUT                 | `/continents/{continent_name}` | Update a specific continent. |
| DELETE              | `/continents/{continent_name}` | Delete a specific continent. |

We've already implemented the `POST` and `GET` methods in the previous lessons.
This lesson will complete the API by implementing the `PUT` and `DELETE`.

## HTTP PUT - Updating a resource

We will update the data for an existing continent by overriding the `doPut()` method.
The implementation will combine some of the things we did in the `doGet()` and `doPost()`
methods:

1. The `continent_name` path parameter must be extracted from the URL.  This value provides
   the hashtable key for the object to update (i.e. the name `zealandia` in the URL below).
   `http://localhost:8080/continents/zealandia`
2. The updated `Continent` data must be extracted from the request body as a JSON string.
   For example, let's increase the area and population by 1:
   ```json
   {
       "name":"zealandia",
       "area":4900001,
       "population":1
   }
   ```

Edit `ContinentServlet` to override the `doPut()` method as shown:

```java
// PUT http://localhost:8080/continents/{continent_name}
    protected void doPut(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        // get the abbreviation from path http://localhost:8080/continents/{continent_name}
        String continent_name = request.getRequestURI().substring("/continents/".length());

        // lookup the continent from the hashmap using the name as the key
        Continent continent = map.get(continent_name);

        // generate the response
        if (continent == null) {
            // send error status and message as server response
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);  //status = 500
            response.setContentType("text/plain;charset=UTF-8");
            response.getOutputStream().println(String.format("Continent %s not found", continent_name));
        }
        else {
            // Read client data from request body
            String requestData = request.getReader().lines().collect(Collectors.joining());
            // create a new continent from JSON formatted string contained in the request body
            Continent updatedContinent = new ObjectMapper().readValue(requestData, Continent.class);

            // save the updated continent in the hashmap
            map.put(continent_name, updatedContinent);

            // send success status and state formatted as JSON as server response
            response.setStatus(HttpServletResponse.SC_OK);   //status = 200
            response.setContentType("application/json;charset=UTF-8");
            response.getOutputStream().print(new ObjectMapper().writeValueAsString(updatedContinent));
        }
    }
```

1. Stop and restart the `Tomcat9` plugin.
2. Issue a `POST` request in Postman to add the `Zealandia` continent back into the hashmap with the request body content as shown.    
   ![post continents request in postman](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-request-body/post_continents_request.png)
3. Issue a `PUT` request in Postman to update the new continent's area and population.    
   ![postman put request](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-restful-servlet/put_request.png)
4. Issue a `GET` request in Postman to confirm the continent was updated.   
   ![get after put](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-restful-servlet/get_after_put.png) 


## HTTP DELETE - Deleting a resource

Finally, we will delete an existing continent by overriding the `doDelete()` method.
The `continent_name` path parameter must be extracted from the URL in a similar
manner as the `doGet()` and `doPut()` methods.  

Edit `ContinentServlet` to override the `doDelete()` method as shown.  The response
will contain a JSON representation of the deleted continent, or an error message if the
continent name is not in the hashmap.

```java
// DELETE http://localhost:8080/continents/{continent_name}
    protected void doDelete(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        // get the continent name from path http://localhost:8080/continents/{continent_name}
        String continent_name = request.getRequestURI().substring("/continents/".length());

        Continent continent = map.remove(continent_name);

        // generate the response
        if (continent == null) {
            // send error status and message as server response
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);  //status = 500
            response.setContentType("text/plain;charset=UTF-8");
            response.getOutputStream().println(String.format("Continent %s not found", continent_name));
        }
        else {
            // send success status and message as server response
            response.setStatus(HttpServletResponse.SC_OK);   //status = 200
            response.setContentType("application/json;charset=UTF-8");
            response.getOutputStream().print(new ObjectMapper().writeValueAsString(continent));
        }
    }
```

1. Stop and restart the `Tomcat9` plugin.
2. Issue a `DELETE` request in Postman to delete one of the existing continents.  
   ![delete continent request in postman](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-restful-servlet/delete_request.png)
3. Issue a `GET` request in Postman to confirm the continent was deleted.    
   ![get after delete](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-restful-servlet/get_after_delete.png)


## Code Check

Let's review the final version of the web application.

The `pom.xml` file:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.example</groupId>
  <artifactId>servlets</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>

  <!-- Set compiler to Java11 -->
  <properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>

  <name>servlets Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/jakarta.servlet/jakarta.servlet-api -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
      <scope>provided</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.14.1</version>
    </dependency>

  </dependencies>

  <build>
    <finalName>servlets</finalName>

    <!-- Tomcat9 Maven Plugin -->
    <plugins>
      <plugin>
        <groupId>org.opoo.maven</groupId>
        <artifactId>tomcat9-maven-plugin</artifactId>
        <version>3.0.1</version>
        <configuration>
          <path>/</path>
        </configuration>
      </plugin>
    </plugins>

  </build>
</project>

```

The `Continent` class:

```java
public class Continent {
    private String name;
    private Integer area;
    private Long population;

    public Continent() {}

    public Continent(String name, Integer area, Long population) {
        this.name = name;
        this.area = area;
        this.population = population;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getArea() {
        return area;
    }

    public void setArea(Integer area) {
        this.area = area;
    }

    public Long getPopulation() {
        return population;
    }

    public void setPopulation(Long population) {
        this.population = population;
    }

    @Override
    public String toString() {
        return "{" +
                "\"name\":" + "\"" + name + "\"" + "," +
                "\"area\":" +  area +  "," +
                "\"population\":" + population +
                "}";
    }
}
```

The `ContinentServlet` class:

```java
import com.fasterxml.jackson.databind.ObjectMapper;

import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.stream.Collectors;

@WebServlet(urlPatterns="/continents/*")
public class ContinentServlet extends HttpServlet {

    //key is continent name, value is land area (Km^2)
    private HashMap<String, Continent> map;

    // init() is automatically called after the Servlet is instantiated
    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        // data from https://en.wikipedia.org/wiki/Continent
        map = new HashMap<>();
        map.put("asia", new Continent("asia", 44614000, 4700000000L ));
        map.put("africa", new Continent("africa", 30365000, 1400000000L));
        map.put("north_america", new Continent("north america", 24230000, 600000000L ));
        map.put("south_america", new Continent("south america", 17814000, 430000000L ));
        map.put("antarctica", new Continent("antarctica", 14200000, 0L));
        map.put("europe", new Continent("europe", 10000000, 750000000L));
        map.put("oceania", new Continent("australia/oceania", 8510900, 44000000L));
    }

    //http://localhost:8080/continents/{continent_name}
    protected void doGet(
            HttpServletRequest req,
            HttpServletResponse resp)
            throws ServletException, IOException {

        // get the {continent_name} path parameter http://localhost:8080/continents/{continent_name}
        String continent_name = req.getRequestURI().substring("/continents/".length());

        // lookup the continent from the hashmap using the continent name as the key
        Continent continent = map.get(continent_name);

        // generate the response
        if (continent == null) {
            // send error status and message as server response
            resp.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);  //status = 500
            resp.setContentType("text/plain;charset=UTF-8");
            resp.getOutputStream().print(String.format("Continent %s not found.", continent_name));
        }
        else {
            // send success status and state formatted as JSON as server response
            resp.setStatus(HttpServletResponse.SC_OK);   //status = 200
            resp.setContentType("application/json;charset=UTF-8");
            resp.getOutputStream().print(new ObjectMapper().writeValueAsString(continent));
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // Read client data from request body
        String requestData = req.getReader().lines().collect(Collectors.joining());

        // create a new Continent from JSON-formatted string contained in the request body
        Continent continent = new ObjectMapper().readValue(requestData, Continent.class);

        // add the Continent to the hashmap using the continent name as the key
        map.put(continent.getName(), continent);

        // send created status and continent formatted as JSON as server response
        resp.setStatus(HttpServletResponse.SC_CREATED);  //status=201
        resp.setContentType("application/json;charset=UTF-8");
        resp.getOutputStream().print(new ObjectMapper().writeValueAsString(continent));
    }

    // PUT http://localhost:8080/continents/{continent_name}
    protected void doPut(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        // get the continent name from path http://localhost:8080/continents/{continent_name}
        String continent_name = request.getRequestURI().substring("/continents/".length());

        // lookup the continent from the hashmap using the name as the key
        Continent continent = map.get(continent_name);

        // generate the response
        if (continent == null) {
            // send error status and message as server response
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);  //status = 500
            response.setContentType("text/plain;charset=UTF-8");
            response.getOutputStream().println(String.format("Continent %s not found", continent_name));
        }
        else {
            // Read client data from request body
            String requestData = request.getReader().lines().collect(Collectors.joining());
            // create a new continent from JSON formatted string contained in the request body
            Continent updatedContinent = new ObjectMapper().readValue(requestData, Continent.class);

            // save the updated continent in the hashmap
            map.put(continent_name, updatedContinent);

            // send success status and state formatted as JSON as server response
            response.setStatus(HttpServletResponse.SC_OK);   //status = 200
            response.setContentType("application/json;charset=UTF-8");
            response.getOutputStream().print(new ObjectMapper().writeValueAsString(updatedContinent));
        }
    }

    // DELETE http://localhost:8080/continents/{continent_name}
    protected void doDelete(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        // get the continent name from path http://localhost:8080/continents/{continent_name}
        String continent_name = request.getRequestURI().substring("/continents/".length());

        Continent continent = map.remove(continent_name);

        // generate the response
        if (continent == null) {
            // send error status and message as server response
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);  //status = 500
            response.setContentType("text/plain;charset=UTF-8");
            response.getOutputStream().println(String.format("Continent %s not found", continent_name));
        }
        else {
            // send success status and message as server response
            response.setStatus(HttpServletResponse.SC_OK);   //status = 200
            response.setContentType("application/json;charset=UTF-8");
            response.getOutputStream().print(new ObjectMapper().writeValueAsString(continent));
        }
    }

}
```

## Conclusion

We've implemented a basic RESTful API using a Java servlet:

| HTTP Request Method | Route                          | Description                  | Servlet Method   |
|---------------------|--------------------------------|------------------------------|------------------| 
| POST                | `/continents/`                 | Create a new continent.      | doPost()         |
| GET                 | `/continents/{continent_name}` | Show a specific continent.   | doGet()          |
| PUT                 | `/continents/{continent_name}` | Update a specific continent. | doPut()          |
| DELETE              | `/continents/{continent_name}` | Delete a specific continent. | doDelete()       |

The `@WebServlet` annotation lets us capture the routes:

```java
@WebServlet(urlPatterns="/continents/*")
```

The `{continent_name}` path parameter can be extracted from the URL:

```java
// get the {continent_name} path parameter http://localhost:8080/continents/{continent_name}
String continent_name = req.getRequestURI().substring("/continents/".length());
```

We can create a POJO from the request body:

```java
// Read client data from request body
String requestData = req.getReader().lines().collect(Collectors.joining());

// create a new Continent from JSON-formatted string contained in the request body
Continent continent = new ObjectMapper().readValue(requestData, Continent.class);
```

We can create a JSON response from the POJO:

```java
resp.getOutputStream().print(new ObjectMapper().writeValueAsString(continent));
```