# Tutorial

In this tutorial we’ll take a glance at JDI Dark, a library that simplifies test automation, makes test run results stable, predictable and easy to maintain.

1. **Quick Start** - a short instruction on how to add JDI Dark to your project and perform its basic configuration.
2. **JDI Dark at a glance** -  a simple example for creating test case with JDI Dark. 
3. **JDI Dark Service Objects** - a Service Object class description
4. **JDI Dark and Cucumber** -  a short instruction on how to create cucumber feature files with JDI Dark.

## 1. Quick Start
###Maven Dependencies
 ```java
<dependency>
    <groupId>com.epam.jdi</groupId>
    <artifactId>jdi-dark</artifactId>
    <version>{RELEASE}</version>
</dependency>
 ```
First, we need to add JDI Dark to the dependency section of our pom.xml file.
The latest version can be found in <a href="https://search.maven.org/classic/#search%7Cga%7C1%7Cjdi-dark" target="_blank">the Maven Central Repository</a>.
###Configuration
For configuration project create **test.properties file** on path **src/test/resources**.

Let’s look at them:

 ```java
domain=local=http://localhost:8080, trello=https://api.trello.com/1
log.level=WARNING
 ```
 + **domain** - there is several options how specify domain:
 
 **-** Specify one domain - *domain=URI*. Than you can use this setting in service domain annotation in this way:  *@ServiceDomain("${domain}")*, *@ServiceDomain("${nameDomain2}")*
 
 **-** Specify list of domains - *domain=nameDomain=URI, nameDomain2=URI*. Than you can use this variables in service domain annotation in this way:  *@ServiceDomain("${nameDomain}")*, *@ServiceDomain("${nameDomain2}")*     
       
 **-** If you don't specify domain here, you must directly specify your base URI in this way *@ServiceDomain("https://api.trello.com/1")*  
    


 + **log.level** - you can set one of options - INFO, WARNING, OFF, FATAL, ERROR, STEP, DEBUG, TRACE, ALL

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/resources/test.properties" target="_blank">See example on GitHub</a>.

## 2. JDI Dark at a glance

```java
//STEP 1 Creating class describing Board object with needed fields   
public class Board {
    private String id;
    private String name;
    
    public String getId() {return id;}
    public void setId(String id) {this.id = id;}
    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
}


//STEP 2 Creating class Service Object Model
@ServiceDomain("${trello}")
@QueryParameters({
        @QueryParameter(name = "key", value = "3445103a21ddca2619eaceb0e833d0db"),
        @QueryParameter(name = "token", value = "a9b951262e529821308e7ecbc3e4b7cfb14a24fef5ea500a68c69d374009fcc0")
})
public class TrelloService {

    public static final String BOARDS = "/boards";
    
    @ContentType(JSON)
    @POST(BOARDS)
    public static RestMethod<Board> boardsPost;

    public static Board createBoard(Board board) {
        return boardsPost.post(board, Board.class);
    }
}

//STEP 3 Creating class for generating needed data
public class TrelloDataGenerator {

    public static Board generateBoard() {
        Board board = new Board();
        board.setName("JDI Test Board number " + new Random().nextInt(Integer.MAX_VALUE));
        return board;
    }
}


//STEP 4 Creating  test class
public class TrelloTests {

    @BeforeClass
    public void initService() {
        init(TrelloService.class);
    }
    
    @Test
    public void createCardInBoard() {    
        Board board = TrelloDataGenerator.generateBoard();
        Board createdBoard = TrelloService.createBoard(board);
        Board gotBoard = TrelloService.getBoard(createdBoard.getId());
        Assert.assertEquals(gotBoard.getName(), createdBoard.getName(), "Name of created board is incorrect");
    }
}
```

###First test creation

In this example we create test for Trello */boards/* endpoint. You can familiarize with  Trello API  <a href="https://developers.trello.com/reference/#boardsid" target="_blank">here</a>.

<br/>
**1. Create class describing REST object** 

Create class for <a href="https://developers.trello.com/reference/#board-object" target="_blank">board object</a> with needed fields. Create get and set methods for these fields.  
 
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/main/java/com/epam/jdi/dto/Board.java" target="_blank">See example class for Card Object</a>.

<br/>
**2. Create  service class with Rest methods** 

-Specify BaseURI service by using @ServiceDomain("${trello}") annotation before class 

-Specify credentials via @QueryParameters - set  @QueryParameter key and token

-Describe <a href="https://developers.trello.com/reference/#boardsid" target="_blank">trello endpoint for creating board</a>:

  - Add RestMethod<Board> boardsPos method
  
  - Create method createBoard for transfer and getting Board object in RestMethod  

  - User annotation @POST(BOARDS) for specific post endpoint

  - Use annotation @ContentType(JSON) for specific sending content-type

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/main/java/com/epam/jdi/httptests/TrelloService.java" target="_blank">See example class for TrelloService </a>.   
<br/>  
**3.If you need - create class helper for generating data** 

In our example we created TrelloDataGenerator class which contains  method for generating Trello board

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/utils/TrelloDataGenerator.java" target="_blank">See example class for generating Trello data</a>.

<br/>
**4. Create test class**  
1.Initialize your service class in @BeforeClass

2.Add test methods
    
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/TrelloTests.java" target="_blank">See test case example</a>.


###Running test examples 

You can see and run various examples test cases in <a href="https://github.com/jdi-testing/jdi-dark/tree/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests" target="_blank">JDI DARK project </a>.
Before first run test cases, execute maven commands:

**mvn clean**<br/>
**mvn compile**


## 3. JDI Dark Service Objects
  JDI Dark provides describing web service as a Service Object class, what allows to make well-structure clear code.
  So it is easy to create maintained and well-readable tests. See Tests with Service Object documentation <a href="https://jdi-docs.github.io/jdi-dark/#tests-with-service-object">here</a>.
  

  

## 4. JDI Dark and Cucumber

```java
//STEP 1 Service Object Model class creation
@ServiceDomain("http://httpbin.org/")
public class ServiceExample {
    @ContentType(JSON)
    @GET("/get")
    @Headers({
            @Header(name = "Name", value = "Roman"),
            @Header(name = "Id", value = "Test")
    })
    RestMethod getMethod;

    @ContentType(JSON)
    @GET("/get")
    RestMethod get;

    @Header(name = "Type", value = "Test")
    @POST("/post")
    RestMethod postMethod;

    @PUT("/put")
    RestMethod putMethod;
    @PATCH("/patch")
    RestMethod patch;
    @DELETE("/delete")
    RestMethod delete;
    @GET("/status/{status}")
    RestMethod status;
}



//STEP 2, 4 Feature file creation (pre-created steps from JDI Dark BDD are used here)
Feature: Json response check

  Scenario: Check json response
    Given I init service
    And I set JSON request content type
    When I do getMethod request
    Then Response status type is OK
    And Response body has values
      | url          | http://httpbin.org/get |
      | headers.Host | httpbin.org            |
    And Response header "Connection" is "keep-alive"



//STEP 3 TestRunner creation (TestNG example)
@CucumberOptions(features = "src/test/resources/features/",
        glue = {"com/epam/jdi/httptests/steps", "com/epam/jdi/http/stepdefs/en"},
        tags = {"@smoke"})
public class HttpTestsRunner extends AbstractTestNGCucumberTests {
}

```

In this example we create tests for simple HTTP Request & Response Service. You can familiarize with API <a href="http://httpbin.org/" target="_blank">here</a>.

<br/>
**1. Create  service class with Rest methods** 

-Specify BaseURI service by using @ServiceDomain("http://httpbin.org/") annotation before class 

-Describe endpoints using annotations:

  - Add necessary RestMethod methods (@POST, @GET, @PUT, @PATCH and @DELETE)

  - Use annotation @Headers, @Header for specific headers

  - Use annotation @ContentType for specific content-type
  
  - Use other necessary annotation as described <a href="https://jdi-docs.github.io/jdi-dark/#tests-with-service-object" target="_blank">here</a>.

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-bdd-tests/src/main/java/com/epam/jdi/httptests/ServiceExample.java" target="_blank">See example for Service Object class</a>.   
<br/>

**2. Create feature files**

- Create folder to have all *.feature files in place. Recommended to store in src/test/resources. But can be changed via @CucumberOptions -> ‘feature’ attribute

<a href="https://github.com/jdi-testing/jdi-dark/tree/master/jdi-dark-bdd-tests/src/test/resources/features" target="_blank">See examples for feature files</a>.   
<br/>

**3. Create TestRunner class (JDI Dark support both TestNG and Junit)**

Specify additional options/custom properties via @CucumberOptions (‘strict’, tags’, ‘plugin’, ‘features’, ‘glue’, ‘format’ are mostly used).
For more details on @CucumberOptions and configs check cucumber documentation <a href="https://cucumber.io/docs/cucumber/api/" target="_blank">here</a>.   
<br/>
Junit TestRunner example <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-bdd-tests/src/test/java/CucumberTestRunner.java" target="_blank">here</a>.
<br/>

TestNG TestRunner example <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-bdd-tests/src/test/java/com/epam/jdi/httptests/HttpTestsRunner.java" target="_blank">here</a>.  
<br/>

**4. Create step definitions in java**

- Use pre-created <a href="https://jdi-docs.github.io/jdi-dark/#jdi-dark-bdd-steps" target="_blank">JDI Dark BDD Steps</a>.

Check all pre-created JDI Dark BDD steps <a href="https://github.com/jdi-testing/jdi-dark/tree/master/jdi-dark-bdd/src/main/java/com/epam/jdi/http/stepdefs/en" target="_blank">here</a>.

- Also, you can add your own steps. Create a package for step definitions in java.

Check the example <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-bdd-tests/src/test/java/com/epam/jdi/httptests/steps/InitService.java" target="_blank">here</a>.

**5. Install plugin for relevant IDEA**

Go File->Settings-->Plugins and double check the plugin installation. Then you can manage cucumber plugin as shown below This should redirect you to step definition when you click your test.

**6. Create scenario description in Gherkin in feature file.**

You can see the various examples of feature files <a href="https://github.com/jdi-testing/jdi-dark/tree/master/jdi-dark-bdd-tests/src/test/resources/features" target="_blank">here</a>.

**7. Run the tests via any available option**

- CLI

- maven plugin

- jUnit or TestNG runner

- IDE plugin for Cucumber

## API Testing Framework structure

JDI dark consists of next packages:

 - annotations - contain annotations that can be used in project    
 - logger - contain classes which implement JDI DARK logging
 - requests - contain classes which implement JDI sending requests, error handling and initialization of service
 - response - contain classes which used for working with Rest response
 - performance - contain classes which used for perfomance testing

 

