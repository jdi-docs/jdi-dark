# Tutorial

In this tutorial we’ll take a glance at JDI Dark, a library that simplifies test automation, makes test run results stable, predictable and easy to maintain.

1. **Quick Start** - a short instruction on how to add JDI Dark to your project and perform its basic configuration.
2. **JDI Light at a glance** -  a simple example for creating test case with JDI DARK. 

## 1. Quick Start
###Maven Dependencies
 ```java
<dependency>
    <groupId>com.epam.jdi</groupId>
    <artifactId>jdi-light-dark</artifactId>
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

###First test creating

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

  - Set annotation @POST(BOARDS) with specifying endpoint

  - Set annotation @ContentType(JSON) for specifying sending content-type

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

## Running test examples
Before first running test examples execute next maven commands:  
**mvn clean**       
**mvn compile**
## 3. JDI Dark Service Objects
TBD

## 4. JDI Dark and Cucumber
TBD
## 5. JDI Dark Performance
TBD
## JDI Dark settings at a glance
TBD
## API Testing Framework structure

JDI dark consist of next packages:

 - annotations - contains annotations that can be used in project    
 - logger - contains classes which implements JDI DARK logging
 - requests - contains classes which implements JDI sending requests, error handling and initialization of service
 - response - contains classes which used for perfomance testing
 - performance - contains of classes which implements JDI DARK logging

 

