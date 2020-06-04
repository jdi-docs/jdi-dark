# Tutorial

In this tutorial we’ll take a glance at JDI Dark, a library that simplifies test automation, makes test run results stable, predictable and easy to maintain.

1. **Quick Start** - a short instruction on how to add JDI Dark to your project and perform its basic configuration.
2. **JDI Dark at a glance** -  a simple example of creating test cases for REST services with JDI Dark. 
3. **JDI Dark Service Objects** - a Service Object class description
4. **JDI Dark and Cucumber** -  a short instruction on how to create cucumber feature files with JDI Dark.
5. **JDI Dark and SOAP** -  a short instruction on how to create test cases for SOAP with JDI Dark.

## 1. Quick Start
###Maven Dependencies
 ```java
<dependency>
    <groupId>com.epam.jdi</groupId>
    <artifactId>jdi-dark</artifactId>
    <version>{RELEASE}</version>
</dependency>
 ```
First, you need to add JDI Dark to the dependency section of your pom.xml file.
The latest version can be found in <a href="https://search.maven.org/classic/#search%7Cga%7C1%7Cjdi-dark" target="_blank">the Maven Central Repository</a>.
###Configuration
Create **test.properties file** on path **src/test/resources** to configure the project.

Add domain and log.level into the properties file.

 ```java
domain=local=http://localhost:8080, trello=https://api.trello.com/1
log.level=WARNING
 ```
 + **domain** 
 
 There are tree ways to specify your domain:
 
 **-** If there is only one domain, specify *domain=URI* in the properties file. 
 Then you can use the service domain annotation this way:  *@ServiceDomain("${domain}")*
 
 **-** In case of a list of domains, specify *domain=nameDomain=URI, nameDomain2=URI*. 
 Then use these variables in the service domain annotation:  
 *@ServiceDomain("${nameDomain}")*
  
 *@ServiceDomain("${nameDomain2}")*     
       
 **-** It's possible to specify the domain directly in the service domain annotation *@ServiceDomain("https://api.trello.com/1")*  
    
 + **log.level** 
  
  Available options: INFO, WARNING, OFF, FATAL, ERROR, STEP, DEBUG, TRACE, ALL

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
@QueryParameter(name = "key", value = "3445103a21ddca2619eaceb0e833d0db")
@QueryParameter(name = "token", value = "a9b951262e529821308e7ecbc3e4b7cfb14a24fef5ea500a68c69d374009fcc0")
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

  - Specify BaseURI service by using @ServiceDomain("${trello}") annotation before class 

  - Specify credentials via @QueryParameters - set  @QueryParameter key and token

  - Describe <a href="https://developers.trello.com/reference/#boardsid" target="_blank">trello endpoint for creating board</a>:

  - Add RestMethod<Board> boardsPos method
  
  - Create method createBoard for transfer and getting Board object in RestMethod  

  - User annotation @POST(BOARDS) for specific post endpoint

  - Use annotation @ContentType(JSON) for specific sending content-type

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/main/java/com/epam/jdi/httptests/TrelloService.java" target="_blank">See example class for TrelloService </a>.   
<br/>  
**3. Create class helper for generating data if needed** 

In our example we created TrelloDataGenerator class which contains  method for generating Trello board

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/utils/TrelloDataGenerator.java" target="_blank">See example class for generating Trello data</a>.

<br/>
**4. Create test class**  
  - Initialize your service class in @BeforeClass
<br/>  
  - Add test methods
<br/>  
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples/entities/TrelloTests.java" target="_blank">See test case example</a>.


###Running test examples 

You can find a lot of test examples <a href="https://github.com/jdi-testing/jdi-dark/tree/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests" target="_blank">JDI DARK project </a>.
Before first running, execute maven commands:

**mvn clean**<br/>
**mvn compile**


## 3. JDI Dark Service Objects
  JDI Dark provides Service Object class to describe web services. Service Object class allows to create well-structured code.
  <br/>
  <a href="https://jdi-docs.github.io/jdi-dark/#tests-with-service-object">See the examples here</a>
  

  

## 4. JDI Dark and Cucumber

```java
//STEP 1 Service Object Model class creation
@ServiceDomain("http://httpbin.org/")
public class ServiceExample {
    @ContentType(JSON)
    @GET("/get")
    @Header(name = "Name", value = "Roman")
    @Header(name = "Id", value = "Test")
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
```

```gherkin
#STEP 2, 4 Feature file creation (pre-created steps from JDI Dark BDD are used here)
Feature: Json response check

  Scenario: Check json response
    Given init service example
    And set request content type to 'JSON'
    When perform 'getMethod' request
    Then response status type is OK
    And response body has values
      | url          | http://httpbin.org/get |
      | headers.Host | httpbin.org            |
    And response header "Connection" is "keep-alive"
```
```java
//STEP 3 TestRunner creation (TestNG example)
@CucumberOptions(features = "src/test/resources/features/",
        glue = {"com/epam/jdi/httptests/steps", "com/epam/jdi/http/stepdefs/en"},
        tags = {"@smoke"})
public class HttpTestsRunner extends AbstractTestNGCucumberTests {
}

```

In this example we will create tests for simple HTTP Request & Response Service. You can get acquainted with API <a href="http://httpbin.org/" target="_blank">here</a>.

<br/>
**1. Create  service class with Rest methods** 

-Specify BaseURI service by using @ServiceDomain("http://httpbin.org/") annotation before class 

-Describe endpoints using annotations:

  - Add necessary RestMethod methods (@POST, @GET, @PUT, @PATCH and @DELETE)

  - Use annotation @Headers, @Header for specific headers

  - Use annotation @ContentType to specify the content-type
  
  - Use other useful annotation described <a href="https://jdi-docs.github.io/jdi-dark/#tests-with-service-object" target="_blank">here</a>.

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-bdd-tests/src/main/java/com/epam/jdi/services/ServiceExample.java" target="_blank">See example of the Service Object class</a>.   
<br/>

**2. Create feature files**

- Create a folder to have all *.feature files in place. Recommended to store in src/test/resources. But can be changed via @CucumberOptions -> ‘feature’ attribute

<a href="https://github.com/jdi-testing/jdi-dark/tree/master/jdi-dark-bdd-tests/src/test/resources/features" target="_blank">See examples of the feature files</a>.   
<br/>

**3. Create TestRunner class (JDI Dark supports both TestNG and Junit)**

Specify additional options/custom properties via @CucumberOptions (‘strict’, tags’, ‘plugin’, ‘features’, ‘glue’, ‘format’).
For more details on @CucumberOptions and configs check cucumber documentation <a href="https://cucumber.io/docs/cucumber/api/" target="_blank">here</a>.   
<br/>
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-bdd-tests/src/test/java/com/epam/jdi/httptests/CucumberTestRunner.java" target="_blank">Junit TestRunner example</a>.
<br/>

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-bdd-tests/src/test/java/com/epam/jdi/httptests/HttpTestsRunner.java" target="_blank">TestNG TestRunner example</a>.  
<br/>

**4. Create step definitions in Java**

- Use pre-created <a href="https://jdi-docs.github.io/jdi-dark/#jdi-dark-bdd-steps" target="_blank">JDI Dark BDD Steps</a>.

Check all pre-created JDI Dark BDD steps <a href="https://github.com/jdi-testing/jdi-dark/tree/master/jdi-dark-bdd/src/main/java/com/epam/jdi/http/stepdefs/en" target="_blank">here</a>.

- Also, you can add your own steps. Create a package for step definitions in Java.

Check the example <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-bdd-tests/src/test/java/com/epam/jdi/httptests/steps/InitService.java" target="_blank">here</a>.

**5. Install Cucumber plugin for IDEA**

Go File->Settings-->Plugins. Then add Cucumber plugin. This plugin redirects you to the step definition when clicking the step.

**6. Create scenario description in Gherkin in feature file.**

You can see the examples of feature files <a href="https://github.com/jdi-testing/jdi-dark/tree/master/jdi-dark-bdd-tests/src/test/resources/features" target="_blank">here</a>.

**7. Run the tests via any available option**

- CLI

- maven plugin

- jUnit or TestNG runner

- IDE plugin for Cucumber

## 5. JDI Dark and SOAP
In this example we will create a tests for SOAP service yandex-speller with WSDL <a href="https://speller.yandex.net/services/spellservice?WSDL" target="_blank">https://speller.yandex.net/services/spellservice?WSDL</a>

**1. Generate objects from WSDL** 

Generate objects classes from WSDL using <a href="https://jdi-docs.github.io/jdi-dark/#soap-objects-generation-from-wsdl" target="_blank">SOAP objects generation from WSDL</a>



**2. Create the Service object class** 

```java
@ServiceDomain("http://speller.yandex.net/services/spellservice")
public class YandexSpeller {

    @POST()
    public static SoapMethod<CheckTextRequest, CheckTextResponse> checkText;

    @POST()
    public static SoapMethod<CheckTextsRequest, CheckTextsResponse> checkTexts;
}
```
1. Specify BaseURI service by using @ServiceDomain("https://speller.yandex.net/services/spellservice") annotation before class 

2. Describe SOAP methods using SoapMethod<Request object, Response object> class.
For example, for checking text - ``public static SoapMethod<CheckTextRequest, CheckTextResponse>``

3. Add annotation @POST() for your method
  
You can use other necessary annotations as described <a href="https://jdi-docs.github.io/jdi-dark/#creating-service-object" target="_blank">here</a>.

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/main/java/com/epam/jdi/soap/YandexSpeller.java" target="_blank">See an example for Service Object class</a>.   
<br/>


**3. Create the test class**  

```java
public class YandexSpellerSOAPTests {
    @BeforeTest
    public void before() {
        init(YandexSpeller.class);
    }

    @Test
    public void checkTestResponse() {
        CheckTextResponse response = YandexSpeller.checkText.callSoap(new CheckTextRequest().withText("soap").withLang("en"));
        Assertions.assertThat(response.getSpellResult().getError().size()).isZero();
    }
}
```
1.Initialize your service class in @BeforeClass

2.Add test methods
    
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples/soap/YandexSpellerSOAPTests.java" target="_blank">See a test case example</a>.
## API Testing Framework structure

JDI Dark includes the packages:

 - annotations - annotations that can be used in project to simplify your test development    
 - logger - classes implementing JDI Dark logging
 - requests - classes implementing JDI Dark sending requests, error handling and initialization of the services
 - response - classes for working with Rest response
 - performance - classes for the performance testing

 

