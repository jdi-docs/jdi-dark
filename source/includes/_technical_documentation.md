# Documentation
## Framework usage

JDI Dark Framework provides great opportunities for web services testing.  
Below you will find the description of their features and the ways of using with code examples.

## HTTP methods
JDI Dark supports the following HTTP methods:

 - GET
 - POST
 - PUT
 - PATCH
 - DELETE
 - OPTIONS
 - HEAD

For all of these methods there are annotations named after them in the *com.epam.http.annotations.** package.
All of these annotations take the call request URI value. The annotations are supposed to be used in your Service Object class.

To point to the base URI of your service, it's convenient to use the *@ServiceDomain* annotation with URL provided as argument.
Then, values in your method annotations might be just specific URL paths.

### Specifying HTTP method

```java
@POST("/greet")
public static RestMethod greetPost;

@DELETE("/body")
public static RestMethod deleteBody;

@GET("/hello")
public static RestMethod getHello;

@PUT("/cookie")
public static RestMethod putCookie;

@Test
public void formParamsAcceptsIntArgumentsJDI() {
    Object[][] queryPramsArray = new Object[][]{{"firstName", 1234}, {"lastName", 5678}};

    JettyService.greetPost(queryPramsArray)
                .isOk()
                .body("greeting", equalTo("Greetings 1234 5678"));
    }

@Test
public void deleteSupportsStringBody() {
    RestResponse response = deleteBody.call(requestBody(TEST_BODY_VALUE));
    response.assertThat().body(is(TEST_BODY_VALUE));
}

@Test
public void getCanReturnResponseDataAsString() {
    RestResponse response = JettyService.getHello.call();
    final String responseInfo = response.toString();
    assertThat(responseInfo, containsString("Response status: 200 OK (OK)"));
    assertThat(responseInfo, containsString("Response body: {\"hello\":\"Hello Scalatra\"}"));
}

@Test
public void putCanReturnBodyAsString() {
    Map<String, Object> cookies = new HashMap<>();
    cookies.put("username", "John");
    cookies.put("token", "1234");
    final String body = JettyService.putCookie.call(cookies().addAll(cookies)).getBody();
    assertEquals("username, token", body);
}

```

For describing HTTP method use RestMethod class with appropriate annotation in Service object class.  
**@POST("endpoint")** - for POST method  
**@GET("endpoint")** - for GET method  
**@DELETE("endpoint")** - for DELETE method  
**@PUT("endpoint")** - for PUT method  

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/main/java/com/epam/jdi/services/JettyService.java" target="_blank">Test example in Java</a>


Available methods for sending HTTP request in JDI dark: 

|Method | Description | Return Type
--- | --- | ---
**call(RequestData requestData)**| send request with Request Data parameters | RestResponse
**call(JAction1<RequestData> action)**| send request with Request Data parameters | RestResponse
**call()**| send request | RestResponse
**call(RequestSpecification requestSpecification)**| send request with RequestSpecification | RestResponse
**call(RestAssuredConfig restAssuredConfig)**| send request with RestAssuredConfig  | RestResponse
**call(String queryParams)**| send request with query parameters | RestResponse
**callWithNamedParams(String... namedParams)**| send request with named query parameters | RestResponse
**callAsData(Class<T> c)**| send request and map response to Java object | java object
**post(Object body)**| send post/put request with body| RestResponse
**post(Object body, Class<T> c)**| send post/put request with body and parse result to object | java object

<br>
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples" target="_blank">Test examples in Java</a>
<br>


## Request Data
You might need to use specific request data in your requests. *Cookies*, *headers*, *query parameters* and *Content-type* are available in annotated form.
Therefore, you can specify them in your Service Object class providing name and value.

There is a class called *com.epam.http.requests.RequestData* which represents the data sent along with your request.

It is also possible to specify request data when making a request call.
### Request body

```java
public static RequestData requestBody(String body)
public static RequestData requestData(JAction1<RequestData> valueFunc)
 ```
 
Request body can be set when making a request call. Just pass it as argument to the *call()* method or within the RequestData object 
with the following fields:

 - String url
 - String path
 - Object body
 - String contentType
 - Headers headers
 - MultiMap<String, String> <a href="https://jdi-docs.github.io/jdi-dark/#path-parameters">pathParams</a>
 - MultiMap<String, String> <a href="https://jdi-docs.github.io/jdi-dark/#query-parameters">queryParams</a> 
 - MultiMap<String, String> <a href="https://jdi-docs.github.io/jdi-dark/#form-parameters">formParams</a> 
 - Cookies cookies
 - ArrayList<MultiPartSpecification> <a href="https://jdi-docs.github.io/jdi-dark/#multipart-parameters">multiPartSpecifications</a> 
 - ProxySpecification <a href="https://jdi-docs.github.io/jdi-dark/#proxy">proxySpecification</a> 

All of these fields can be set/updated from the *call()* method as well.

You may need to statically import the *com.epam.http.requests.RequestData.** package.

 ```java
public RestResponse call(RequestSpecification requestSpecification) 
 ```
Another way to generate request data is <a href="https://jdi-docs.github.io/jdi-dark/#using-request-data-object-as-a-parameter">using Request Data object as a parameter</a>
<br>
JDI Dark also supports making request calls with <a href="https://jdi-docs.github.io/jdi-dark/#requestspecification-configuration">Rest Assured request specification</a>
### Path parameters

```java
@GET("/{firstName}/{lastName}")
public static RestMethod getUser;

@Test
public void supportsPassingPathParamsToRequestSpec(){
    RestResponse response = getUser
            .call(pathParams().addAll(new Object[][]{{"firstName", "John"}, 
{"lastName", "Doe"}}));
    response.isOk().body("fullName", equalTo("John Doe"));
}

@Test
public void canSpecifySpacePathParamsWithoutKey(){
    RestResponse response = getUser.callWithNamedParams("John", " ");
    response.isOk().body("firstName", equalTo("John")).body("lastName", equalTo(" "));
}

@Test
public void urlEncodesPathParamsInMap(){
    final Map<String, String> params = new HashMap<>();
    params.put("firstName", "John: å");
    params.put("lastName", "Doe");
    RestResponse response = getUser
            .call(pathParams().addAll(params));
    response.isOk().body("fullName", equalTo("John: å Doe"));
}

@GET("/status/{status}?q={value}") RestMethod statusWithQuery;

@Test
public void statusTestWithQueryInPath() {
    RestResponse resp = service.statusWithQuery.callWithNamedParams("503", "some");
    assertEquals(resp.status.code, 503);
    assertEquals(resp.status.type, SERVER_ERROR);
    resp.isEmpty();
}
```

A URL can have one or several path parameters, each denoted with curly braces, e.g. */get/{board_id}*, */boards/{board_id}/cards/{short_card_id}/*. 
You can use them in your Service Object methods and replace placeholders with values when making request calls.

Methods allow to send path params to RequestData:

|Method | Description | Return Type
--- | --- | ---
**pathParams().add(paramName, paramValue)** | pass one parameter to a path | RequestData
**pathParams().addAll(Object[][] array2D)** | pass multiple parameters to a path | RequestData
**pathParams().addAll(Map map)** | pass multiple parameters to a path | RequestData

Methods for passing path params (with/without query params) in RestMethod:

|Method | Description | Return Type
--- | --- | ---
**callWithNamedParams(String... namedParams)** | pass parameters to a path without key| RestResponse

<br>
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples/requestparams/PathParamTests.java" target="_blank">Test examples in Java</a>
<br>
<br />
<br />
<br />
<br />
<br />

### Query parameters

```java
@POST("/greet")
public static RestMethod greetPost;

@Test
public void whenLastParamInGetRequestEndsWithEqualItsTreatedAsANoValueParam() {
  Map<String, String> queryParamsMap = new HashMap<>();
  queryParamsMap.put(FIRST_NAME, FIRST_NAME_VALUE);
  queryParamsMap.put(LAST_NAME, StringUtils.EMPTY);
  
  JettyService.getGreet.call(queryParams().addAll(queryParamsMap)).isOk()
.assertThat().body("greeting", equalTo("Greetings John "));
}

@DELETE("/greet")
public static RestMethod deleteGreet;

@Test
public void bodyHamcrestMatcherWithOutKey() {
    deleteGreet.call(queryParams().addAll(
            new Object[][]{{FIRST_NAME, FIRST_NAME_VALUE},
                    {LAST_NAME, LAST_NAME_VALUE}
            })).isOk().assertThat()
.body(equalTo("{\"greeting\":\"Greetings John Doe\"}"));
}
```

Methods allow to send query params to RequestData:

|Method | Description | Return Type
--- | --- | ---
**queryParams().add(paramName, paramValue)** | pass one parameter to a path | RequestData
**queryParams().addAll(Object[][] array2D)** | pass multiple parameters to a path | RequestData
**queryParams().addAll(Map map)** | pass multiple parameters to a path | RequestData
<br />
<br />
<br />
<br />
<br />
<br />

### Simplified Query parameters
```java
@GET("/noValueParam")
public static RestMethod getNoValueParam;

@Test
public void singleNoValueQueryParamWhenUsingQueryParamInUrlForGetRequest() {
    JettyService.getNoValueParam.call("some")
            .isOk().assertThat().body(is("Params: some="));
}

@Test
public void mixingStartingNoValueQueryParamWhenUsingQueryParamInUrlForGetRequest() {
    JettyService.getNoValueParam.call("some1&some2=one")
            .isOk().assertThat().body(is("Params: some1=some2=one"));
}
```
The method allows to send specific query parameters in URL in RestMethod:

|Method | Description | Return Type
--- | --- | ---
**call(String queryParams)** | pass query parameters | RestResponse
<br>
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples/requestparams/ParamTests.java" target="_blank">Test examples in Java</a>
<br>  
<br />
<br />
<br />
<br />

### Form parameters

```java
@POST("/greet")
public static RestMethod greetPost;

@Test
public void formParamsAcceptsIntArgumentsJDI() {
    RestResponse response = greetPost
            .call(formParams()
.addAll(new Object[][]{{"firstName", 1234}, {"lastName", 5678}}));
    response.isOk().body("greeting", equalTo("Greetings 1234 5678"));
}
```
Methods allow to send form params to RequestData:

|Method | Description | Return Type
--- | --- | ---
**formParams().add(paramName, paramValue)** | pass one form parameter to a path | RequestData
**formParams().addAll(Object[][] array2D)** | pass multiple parameters to a path | RequestData
**formParams().addAll(Map map)** | pass multiple parameters to a path | RequestData

### Multipart parameters

```java
@POST("/multipart/file")
@MultiPart(controlName = "file", fileName = "myFile")
public static RestMethod postMultiPartFile;

@Test
public void multiPartByteArrays() throws Exception {
    final byte[] bytes = IOUtils.toByteArray(getClass()
.getResourceAsStream("/car-records.xsd"));
    JettyService.postMultiPartFile.withMultiPartContent(bytes).call().assertThat()
                .statusCode(200).body(is(new String(bytes)));
    }

@POST("multipart/multiple")
public static RestMethod postMultipartMultiple;

@Test
public void multiPartUploadingWorksForFormParamsAndByteArray() {
    JettyService.postMultipartMultiple.call(requestData(rd -> {
        rd.formParams.add("formParam1", "");
        rd.formParams.add("formParam2", "formParamValue");
        rd.setMultiPart(new MultiPartSpecBuilder("juX").controlName("file"));
        rd.setMultiPart(new MultiPartSpecBuilder("body").controlName("string"));
    })).assertThat()
            .statusCode(200)
            .body(containsString("formParam1 -> WrappedArray()"));
}
```
Methods allow to set multipart parameters to request data:

|Method | Description | Return Type
--- | --- | ---
**setMultiPart(MultiPartSpecBuilder multiPartSpecBuilder)** | set multipart parameters | RequestData

*MultiPartSpecBuilder* - class for creating advanced multi-part requests
<br>
<br>
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples/requestparams/MultiPartUploadTests.java" target="_blank">Test examples in Java</a>  
<br>  
<br>  
<br>  
<br>  
<br>  
<br>  

### Using Request Data object as a parameter

```java
@POST("/greet")
public static RestMethod greetPost;

@Test
public void charsetIsReallyDefined() {
    Map<String, String> formParamsMap = new HashMap<>();
    formParamsMap.put("firstName", "Some & firstname");
    formParamsMap.put("lastName", "<lastname>");

    RestResponse resp = greetPost.call(requestData(rd -> {
        rd.contentType = "application/x-www-form-urlencoded; charset=ISO-8859-1";
        rd.formParams.addAll(formParamsMap);
    }));
    resp.isOk().assertThat()
       .body("greeting", equalTo("Greetings Some & firstname <lastname>"));
}
```

The method allows to send the request with invoked request data in RestMethod:

|Method | Description | Return Type
--- | --- | ---
**call(RequestData requestData)** | make request with parameters indicated by Request Data| RestResponse

<br>
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples/requestparams/ParamTests.java" target="_blank">Test examples in Java</a>
<br>  
<br />
<br />
<br />
<br />

###RequestSpecification configuration

```java
@GET("/jsonStore")
public static RestMethod getJsonStore;

@Test
public void supportsConfiguringJsonConfigProperties() {
    RequestSpecification rs = getJsonStore.getInitSpec()
            .config(RestAssuredConfig.newConfig()
            .jsonConfig(JsonConfig.jsonConfig()
            .numberReturnType(JsonPathConfig.NumberReturnType.BIG_DECIMAL)));
    RestResponse resp = getJsonStore.call(rs);
    resp.isOk()
            .rootPath("store.book")
            .body("price.min()", is(new BigDecimal("8.95")))
            .body("price.max()", is(new BigDecimal("22.99")));
}
```

You can set RequestSpecification for your request. Get RestAssured RequestSpecification from RestMethod object, set and use it.

|Method | Description | Return Type
--- | --- | ---
**getInitSpec()** | get RestAssured RequestSpecification | RequestSpecification 
**call(RequestSpecification spec)** | make request with RequestSpecification | RestResponse 

<br>
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples/customsettings/ConfigITests.java" target="_blank">Test examples in Java</a>
<br>
 
*For general setting RestAssured config see [Accessing RestAssured](https://jdi-docs.github.io/jdi-dark/#access-restassured)*
## Tests without Service Object

```java
public static RestResponse GET(RequestData data)
public static RestResponse GET(String url)
public static RestResponse GET(String url, RequestSpecification requestSpecification) 
```

All the HTTP methods mentioned before are also available in non-annotated form.
They can be made accessible by importing *com.epam.http.requests.RestMethods.** with several signatures.

You can call these methods with either of given arguments:

 - *com.epam.http.requests.RequestData*
 - String URL
 - Rest Assured request specification: *io.restassured.specification.RequestSpecification*

## Tests with Service Object
### Create service
```java
@ServiceDomain("https://httpbin.org/")
public class ServiceExample {
    @ContentType(JSON) @GET("/get")
    @Header(name = "Name", value = "Roman")
    @Header(name = "Id", value = "Test")
    static RestMethod<Info> getInfo;

    @Header(name = "Type", value = "Test")
    @POST("/post")
    RestMethod postMethod;

    @PUT("/put") RestMethod putMethod;
    @PATCH("/patch") RestMethod patch;
    @DELETE("/delete") RestMethod delete;
    @GET("/status/{status}") RestMethod status;

    @Cookie(name = "session_id", value = "1234")
    @Cookie(name = "hello", value = "world")
    @GET("/cookies")
    public RestMethod getCookies;
} 

@ServiceDomain("http://localhost:8080")
public class JettyService {
    @POST("/noValueParam")
    @FormParameter(name = "some1", value = "one")
    public static RestMethod postNoValueParamWithPreDefinedFormParam;
    
    @QueryParameter(name = "firstName", value = "John")
    @QueryParameter(name = "lastName", value = "Doe")
    @GET("/greetXML")
    public static RestMethod getGreetXml;

    @POST("/multipart/file")
    @MultiPart(controlName = "file", fileName = "myFile")
    public static RestMethod postMultiPartFile;

    @GET("/greetJSON")
    @Proxy(host = "127.0.0.1", port = 8888, scheme = "http")
    public static RestMethod getGreenJSONWithProxyParams;

    @URL("http://www.google.se")
    @GET("/search?q={query}&hl=en")
    public static RestMethod searchGoogle;
}

@RetryOnFailure
@ServiceDomain(value = "http://localhost:8080/")
public class RetryingService {
    @GET(value = "503")
    @RetryOnFailure(numberOfRetryAttempts = 2, delay = 1, unit = TimeUnit.SECONDS)
    public static RestMethod get503;

    @GET(value = "502")
    @IgnoreRetry
    public static RestMethod ignoreRetrying;
}

@ServiceDomain("https://localhost:8443")
public class JettyServiceHttps {
    @GET("/jsonStore")
    @TrustStore(pathToJks = "src/test/resources/truststore_mjvmobile.jks", 
password = "test4321")
    public static RestMethod getJsonStore;
}

@ServiceDomain("http://www.dneonline.com/calculator.asmx")
@SOAPNamespace("http://tempuri.org/")
public class DneOnlineCalculator {
    @POST()
    @SOAPAction("http://tempuri.org/Multiply")
    @SOAP12
    public static SoapMethod<Multiply, MultiplyResponse> multiply;
}
```

It's possible to describe tested web service as a Service Object class using annotations:

- @ContentType - represents a Content-Type. There can be used any values in <a href="https://static.javadoc.io/io.rest-assured/rest-assured/3.1.0/io/restassured/http/ContentType.html">Rest Assured enumeration</a>
<br />
- @Header - represents an HTTP Header as:
    - name
    - value
<br />
- @Headers - represents the collection of HTTP headers
<br />
- @Cookie - represents a Cookie as:
    - name
    - value, default = "[unassigned]"
    - additionalValues, default = "[unassigned]"
<br />
- @Cookies - represents the collection of Cookies
<br />
- @FormParameter - represents form parameter as:
    - name
    - value
<br />
- @FormParameters - represents the collection of form parameters
<br />
- @QueryParameter - represents query parameter as:
    - name
    - value
<br />
- @QueryParameters - represents the collection of query parameters
<br />
- @MultiPart - represents MultiPart parameters:
    - fileName, default = ""
    - controlName, default = ""
    - filePath, default = ""
    - mimeType, default = ""
<br />
- @HEAD - represents HTTP head method
    - value, default = ""
<br />
- @GET - represents HTTP get method
    - value, default = ""
<br />
- @DELETE - represents HTTP delete method
    - value, default = ""
<br />
- @PATCH - represents HTTP patch method
    - value, default = ""
<br />
- @POST - represents HTTP post method
    - value, default = ""
<br />
- @PUT - represents HTTP put method
    - value, default = ""
<br />
- @OPTIONS - represents HTTP options method
    - value, default = ""
<br />
- @IgnoreRetry - represents ignore settings for failed tests
<br />
See example <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/main/java/com/epam/jdi/services/RetryingService.java" target="_blank">here</a>.
<br />
- @Proxy - represents Proxy parameters:
    - host
    - port
    - scheme
<br />
- @RetryOnFailure - represents retry settings for failed tests:
    - numberOfRetryAttempts, default = 3
    - errorCodes, default = 502, 503
    - delay, default = 10
    - unit, default = microseconds
<br />
See example <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/main/java/com/epam/jdi/services/RetryingService.java" target="_blank">here</a>.
<br />
- @ServiceDomain - represents the domain name
    - value
<br />
- @TrustStore - represents a TrustStore located on the file-system:
    - pathToJks
    - password
<br />
See example <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/main/java/com/epam/jdi/services/JettyServiceHttps.java" target="_blank">here</a>.
<br />
- @URL - represents HTTP get method, where value is uri
    - value
<br />
<br />
- @SOAP12 - represents SOAP 1.2
<br />
- @SOAPAction - represents the intention of the SOAP HTTP request
    - value
<br />
- @SOAPNamespace - represents the namespace
    - value
<br />
<br />
See annotations <a href="https://github.com/jdi-testing/jdi-dark/tree/master/jdi-dark/src/main/java/com/epam/http/annotations" target="_blank">here</a>.
<br />
See service example <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/main/java/com/epam/jdi/services/JettyService.java" target="_blank">here</a>.

### Create tests for service

```java
public class ServiceTest {

    private ServiceExample service;

    @BeforeClass
    public void before() {
        service = init(ServiceExample.class);
    }

    @Test
    public void simpleRestTest() {
        RestResponse resp = ServiceExample.getInfo.call();
        resp.isOk().
                body("url", equalTo("https://httpbin.org/get")).
                body("headers.Host", equalTo("httpbin.org")).
                body("headers.Id", equalTo("Test"));
        resp.assertThat().header("Connection", "keep-alive");
    }

    @Test
    public void serviceInitTest() {
        RestResponse resp = service.postMethod.call();
        resp.isOk().assertThat().
                body("url", equalTo("https://httpbin.org/post")).
                body("headers.Host", equalTo("httpbin.org"));
    }
}
```

This class can be initialized in tests.
Fields of initialized object can be used to send requests from tests.
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

### Use predefined RestSpecification

```java
public class ServiceTest {

    private RequestSpecification requestSpecification;
    private ServiceExample service;

    @BeforeClass
    public void before() {
        requestSpecification = given().filter(new AllureRestAssured());
        requestSpecification.auth().basic("user", "password");
        service = init(ServiceExample.class, requestSpecification);
    }
}
```

It's possible to setup already used RestSpecification. 
Predefined settings will be used for all endpoints of that service.
In this example basic auth credentials will be passed to all endpoints.  
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

## Working with objects
There are key ability for any Rest Client is working with objects:

 - Send object as request body
 - Get response body as object

### Setup Object Mapper

JDI allows user to setup Object Mapper for serialization/deserialization objects within endpoints.
After Object Mapper is set it will be used in all requests implicitly.

```java
public class BaseTest {
    
    //Set global object mapper for whole project
    @BeforeSuite
    public void preconditions() {
        RestAssured.config = RestAssuredConfig.config().objectMapperConfig(new ObjectMapperConfig().jackson2ObjectMapperFactory(
                new Jackson2ObjectMapperFactory() {
                    @Override
                    public ObjectMapper create(Type type, String s) {
                        ObjectMapper objectMapper = new ObjectMapper();
                        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
                        return objectMapper;
                    }
                }
        ));
    }

    //Set object mapper for service. In this case global mapper will be overridden for SimpleService's endpoints
    @BeforeClass
    public void before() {
        ObjectMapper objectMapper = new Jackson2Mapper(new Jackson2ObjectMapperFactory() {
            @Override
            public com.fasterxml.jackson.databind.ObjectMapper create(Type type, String s) {
                com.fasterxml.jackson.databind.ObjectMapper objectMapper = new com.fasterxml.jackson.databind.ObjectMapper();
                objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
                return objectMapper;
            }
        });
        init(SimpleService.class, objectMapper);
    }
}
```

There are 2 ways to setup Object Mapper:

 - For whole project
 - For specific service

If Object Mapper isn't set default RestAssured mapper will be used.
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

### Using objects in tests
 
  ```java
 public class FlowTests {
    
    @Test
    public void assignBoardToOrganization() {

        //Create organization
        Organization organization = TrelloDataGenerator.generateOrganization();
        //Send object to POST request and get response body as object
        Organization createOrg = TrelloService.createOrganization(organization);

        //Create board
        Board board = TrelloDataGenerator.generateBoard();
        board.setIdOrganization(createOrg.getId());
        //Send object to POST request
        TrelloService.createBoard(board);

        //Check that organization contains created board
        //Send GET request and get List of objects from response
        List<Board> boards = TrelloService.getOrganizationBoards(createOrg);
        Assert.assertTrue(boards.stream().map(Board::getName).collect(Collectors.toList()).contains(board.getName()), "Board wasn't added to organization");
    }
  }
 ```

JDI allows create tests on business language using Service classes and working with objects.
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

  ```java
 public class ServiceExample {
    
    @ContentType(JSON)
    @POST("/organizations")
    public static RestMethod<Organization> createOrganization;
    
    //wrapper method for working with objects
    //send Organization object as request body and return response as Organization object
    public static Organization createOrganization(Organization organization) {
        return createOrganization.post(organization, Organization.class);
    }

    @ContentType(JSON)
    @GET("/organizations/{id}/boards")
    public static RestMethod<Board[]> getOrganizationBoards;

    //send GET request and return response as List of objects
    public static List<Board> getOrganizationBoards(Organization organization) {
        return Arrays.asList(getOrganizationBoards.call(requestPathParams("id", organization.getId())).getRaResponse().as(Board[].class));
    }
}
 ```

For convenient working with objects user can add additional wrapper methods to service classes.
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />
<br />

## Error handling

```java
public interface ErrorHandler {
    boolean hasError(RestResponse restResponse) throws IOException;
    void handleError(RestResponse restResponse) throws IOException;
}

//example of implementation and using ErrorHandler
public class ErrorHandlerTests {
    private ServiceSettings serviceSettings;

    @BeforeClass
    public void initServiceSettings() {
        ErrorHandler errorHandler = new ErrorHandler() {
            @Override
            public boolean hasError(RestResponse restResponse) throws IOException {
                //only Client errors will be caught
                return ResponseStatusType.getStatusTypeFromCode(restResponse.getStatus().code) == ERROR;
            }
            
            @Override
            public void handleError(RestResponse restResponse) throws IOException {
                Assert.fail("Exception is caught: " + restResponse.toString());
            }
        };
        serviceSettings = ServiceSettings.builder().errorHandler(errorHandler).build();
    }

    @BeforeClass(dependsOnMethods = {"initServiceSettings"})
    public void initService() {
        init(TrelloService.class, serviceSettings);
    }
}
```

JDI allows to setup custom behaviour for catching and processing any unexpected responses.

For this need inject implementation of ErrorHandler interface into RestMethods through call Service Object init() method.

By default JDI uses DefaultErrorHandler class for server (5XX) and client(4XX) errors.

## Logging

```java
private static JFunc2<RestMethod, List<RequestData>, String> LOG_REQUEST_TEMP;
private static JAction2<RestResponse, String> LOG_RESPONSE_TEMP;

@BeforeClass
public void initService() {
    init(JettyService.class);
    LOG_REQUEST_TEMP = LOG_REQUEST;
    LOG_RESPONSE_TEMP = LOG_RESPONSE;
    LOG_REQUEST = this::logRequest;
    LOG_RESPONSE = this::logResponse;
}

private String logRequest(RestMethod restMethod, List<RequestData> requestData) {
    MultiMap<String, String> queryparams = new MultiMap<>();
    for (RequestData rd : requestData) {
        queryparams.addAll(rd.queryParams);
    }
    String message = String.format("Do %s %s", restMethod.getType(), restMethod.getUrl());
    logger.info(message);
    //Change request logging for allure
    startStep(message,
            String.format("%s %s %s", restMethod.getType(), restMethod.getUrl(), queryparams));
    return message;
}

private void logResponse(RestResponse response, String uuid) {
    String message = String.format("Received response with %s and body: %s", response.getStatus(), response.getBody());
    logger.info(message);
    //Change response logging for allure
    AllureLogger.passStep(message, uuid);
}
@AfterClass
public void clearLogger() {
    LOG_REQUEST = LOG_REQUEST_TEMP;
    LOG_RESPONSE = LOG_RESPONSE_TEMP;
}

```

By default JDI logging is defined in the next variables:
 
**LOG_REQUEST = RestMethod::logRequest** located in package *com.epam.http.requests*, where logRequest method uses:
    
+ ILogger object method info(String msg, Object... args) - for console logging  
+ AllureLogger.startStep(String message, String requestData)  - for allure logging

**LOG_RESPONSE = RestResponse::logResponse** located in package *com.epam.http.response*, where logResponse method uses:
 
+ ILogger object method info(String msg, Object... args) - for console logging  
+ AllureLogger.passStep(String responseData, String uuid) - for allure logging   

**Customizing logging** 

For customizing logging you should redefine these static variables

+ For customizing logger just for one or several tests -it's necessary to return logging variable in original state.   
+ If it is used allure logging - don't forget to include allure logging in the redefining variable.  
+ For customizing allure logging -it's necessary to redefine it in both variables: LOG_REQUEST and LOG_RESPONSE    

See full example with redefining variables for console logging and allure.
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples/customsettings/LoggingCustomizeTests.java" target="_blank">Test example in Java</a>


## Response data

```java
public ValidatableResponse isOk()
public ValidatableResponse hasErrors()
public ValidatableResponse isStatus(ResponseStatusType type)
public RestResponse assertStatus(ResponseStatus rs)
public ValidatableResponse isEmpty()
```

```java
public ValidatableResponse assertBody(MapArray<String, Matcher<?>> params)
public ValidatableResponse assertBody(Object[][] params)
```

```java
public String getFromHtml(String path)
```

```java
public Response getRaResponse()
public ValidatableResponse assertThat()

public Map<String, String> cookies()
public String cookie(String name)
public Headers headers()
public String header(String name)

public String getBody()
public ResponseStatus getStatus()
public String getContentType() 

```

```java
RestResponse response = JettyService.getHello.call();
assertThat(response.body, containsString("{\"hello\":\"Hello Scalatra\"}"));
String body = response.getBody();

String hello = JettyService.getHello.call().getRaResponse().jsonPath().getString("hello");
```

The class **com.epam.http.response.RestResponse** represents Response data in JDI Dark.

Below are methods that allow to work with response data:

**isOk()** and **hasErrors()** -  verify that status code is 2** or 4**, respectively.

**isStatus(ResponseStatusType type)** and **assertStatus(ResponseStatus rs)** - verify response status type and response 
status respectively against expected result. 

**assertBody(Object[][] params)** and **assertBody(MapArray<String, Matcher<?>> params)** - allow to verify response body

**cookies()** and **headers()** - return response cookies and header values

**cookie(String name)** and **header(String name)** - return cookie and header values respectively corresponding to specified names

**getFromHtml()** -  returns html content of the page by provided path

**getRaResponse()** and **assertThat()** returns Rest Assured Response and ValidatableResponse, 
so it is possible to use Rest Assured methods for validating response body.

**getBody()**, **getStatus()**, **getContentType()** - allows to get response body, status and content type. 
Response body and status can be also retrieved using *body* and *status* fields of the RestResponse class.

**isEmpty()** - verifies that response body is empty.


## Headers

JDI Dark supports header addition to Service endpoints using annotations.

**Single header can be added with usage of @Header annotation:**  

```java
@GET("/header")
@Header(name = "Header_name", value = "Header_value2")
public static RestMethod getHeader;
```
Here's a simple example of adding a header to an endpoint
<br />
<br />
<br />
<br />
<br />


```java
@GET("/multiHeaderReflect")
@Headers({@Header(name = "Header_name", value = "Header_value"),
@Header(name = "Header_name2", value = "Header_value2")})
public static RestMethod getMultiHeaderReflect;
```
**Multiple headers are also supported through @Headers annotation:**  
<br />
<br />
<br />
<br />
<br />

```java
@GET("/header")
@Headers({@Header(name = "Header_name", value = "Header_value"),
@Header(name = "Header_name", value = "Header_value2")})
public static RestMethod getHeader;


@GET("/header")
@Header(name = "some_header")
public static RestMethod getHeaderWithNoValue;


@GET("/multiHeaderReflect")
@Header(name = "MultiValueHeader", value = "Header_value_1", additionalValues = "Header_value_2")
public static RestMethod getMultiValueHeader;
```

**Headers with same name, with no value and with multiple values can be added as well.**  

@Headers annotation is used to add 2 headers with the same name - "Header_name"
<br />
<br />
<br />
<br />
@Header is used in this example to pass a header with name only
<br />
<br />
<br />
<br />
@Header is used here to pass a multiple-value header
<br />
<br />
<br />
<br />
**Methods to add Headers to Request Data.**  
 
Headers can be passed as strings, header objects, maps, and even arrays of objects.
Headers without value and with multiple values can be added as well.

```java

@Test
public void requestDataAllowsSpecifyingHeaderWithoutServiceObjectMethods() {
    RestResponse response = getMultiHeaderReflect
            .call(headers().add("MyHeader", "TestValue"));
    response.isOk().assertThat().header("MyHeader", equalTo("TestValue"));
}

@Test
public void requestSpecificationAllowsSpecifyingMultiValueHeadersWithoutServiceObjectMethod() {
    RestResponse response = getMultiHeaderReflect
            .call(headers().addAll(new Object[][]{
                    {"MyHeader", "Something"},
                    {"MyHeader", "SomethingElse"}}));
    response.isOk();
    assertThat(response.headers().getValues("MyHeader").size(), is(2));
    assertThat(response.headers().getValues("MyHeader"), hasItems("Something", "SomethingElse"));
}

```

|Method | Description | Return Type
--- | --- | ---
**headers().add(paramName, paramValue)** | pass name and value of header | RequestData
**headers().addAll(Object[][] array2D)** | pass array with header names and values | RequestData
**headers().addAll(Map map)** | pass map with header names and values | RequestData
**headers().addAll(Header... header)** | pass header objects | RequestData
**headers().add(String name)** | pass header name without value | RequestData

## Cookies

JDI Dark supports addition of Cookies to Service endpoints.

Cookies with no value and with multiple values can be added as well.

```java
@GET("/cookie")
@Cookies({@Cookie(name = "username", value = "John"),
@Cookie(name = "token", value = "1234")})
public static RestMethod getCookie;

@GET("/cookie_with_no_value")
@Cookie(name = "some_cookie")
public static RestMethod getCookieWithNoValueWithCookies;

@GET("/multiCookieRequest")
@Cookie(name = "key1", value = "value1", additionalValues = "value2")
public static RestMethod getMultiCookieWithCookies;
```

There are methods to add cookies to Request Data. Cookies can be passed as name and value, name and value pairs, maps, and arrays of objects.

|Method | Description | Return Type
--- | --- | ---
**cookies().add(paramName, paramValue)** | pass name and value of cookie | RequestData
**cookies().addAll(Object[][] array2D)** | pass array with cookie names and values | RequestData
**cookies().addAll(Map map)** | pass map with cookie names and values | RequestData
**cookies().add(String name)** | pass cookie name without cookie | RequestData
**cookies().add(String name, String... values)** | pass header name without value | RequestData



```java
@GET("/cookie_with_no_value")
public static RestMethod getCookieWithNoValue;
public static RestResponse getCookieWithOnlyName(String name) {
    return getCookieWithNoValue.call(cookies().add(name));
}

@GET("/cookie")
public static RestMethod getCookie;
public static RestResponse getCookieSpecifiedUsingMap(Map<String, String> cookieMap) {
    return getCookie.call(cookies().addAll(cookieMap));
}

@Test
public void requestSpecificationAllowsSpecifyingCookieWithNoValue() {
    RestResponse response = JettyService.getCookieWithOnlyName("some_cookie");
    assertThat(response.getBody(), equalTo("some_cookie"));
}

@Test
public void requestSpecificationAllowsSpecifyingCookieUsingMap() {
    Map<String, String> cookies = new LinkedHashMap<>();
    cookies.put("username", "John");
    cookies.put("token", "1234");

    RestResponse response = JettyService.getCookieSpecifiedUsingMap(cookies);
    assertThat(response.getBody(), equalTo("username, token"));
}
```

## Proxy

```java
@GET("/greetJSON")
@Proxy(host = "127.0.0.1", port = 8888, scheme = "http")
public static RestMethod getGreenJSONWithProxyParams;

@Test
public void usingProxyAnnotationOnServiceLayer() {
    final Map<String, String> params = new HashMap<>();
    params.put("firstName", "John");
    params.put("lastName", "Doe");
    JettyService.getGreenJSONWithProxyParams.call(RequestData.requestData(queryParams().addAll(params))
            .isOk().assertThat().
            body("greeting.firstName", equalTo("John")).
            body("greeting.lastName", equalTo("Doe"));
}
```

JDI Dark supports addition of Proxy parameters (host, port and scheme) to Service endpoints.

You can use the RestMethod class with a @Proxy annotation

There is method to add proxy parameters to Request Data. JDI Dark also supports making request calls with Rest Assured ProxySpecification.

|Method | Description | Return Type
--- | --- | ---
**setProxySpecification(String scheme, String host, int port)** | set proxy parameters to request data | 

```java
@GET("/greetJSON")
public static RestMethod getGreenJSON;

@Test
public void usingProxyWithSetProxySpecification() {
    final Map<String, String> params = new HashMap<>();
    params.put("firstName", "John");
    params.put("lastName", "Doe");
    JettyService.getGreenJSON.call(RequestData.requestData(rd -> {
        rd.setProxySpecification("http", "localhost", 8888);
        rd.queryParams.addAll(params);
    })).isOk().assertThat().
            body("greeting.firstName", equalTo("John")).
            body("greeting.lastName", equalTo("Doe"));
}

@Test
public void usingProxySpecification() {
    final Map<String, String> params = new HashMap<>();
    params.put("firstName", "John");
    params.put("lastName", "Doe");
    JettyService.getGreenJSON.call(RequestData.requestData(rd -> {
        rd.queryParams.addAll(params);
        rd.proxySpecification = ProxySpecification.host("localhost");
    })).isOk().assertThat().
            body("greeting.firstName", equalTo("John")).
            body("greeting.lastName", equalTo("Doe"));
}
```

## Authentication

```java
public void basicAuthTest() {
        BasicAuthScheme basicAuth = new BasicAuthScheme();
        basicAuth.setUserName("postman");
        basicAuth.setPassword("password");
        RestResponse resp = postmanAuthBasic.call(auth(basicAuth));
        assertEquals(resp.status.code, HttpStatus.SC_OK);
    }
```

Authentication is performed using AuthenticationScheme interface. Many authentication methods are supported in 
RestAssured, so the following methods are already implemented: Basic, Digest, NTLM, OAuth1, OAuth2, Form, Certificate
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

```java
public class customAuthScheme implements AuthenticationScheme {

    private String customAuthKey;

    public void setCustomAuthKey(String customAuthKey) {
        this.customAuthKey = customAuthKey;
    }

    @Override
    public void authenticate(HTTPBuilder httpBuilder) {
        httpBuilder.getClient().addRequestInterceptor(
                (request, context) ->
                        request.addHeader("Authorization", this.customAuthKey));
    }
}
```
In order to create a custom authentication scheme one have to implement AuthenticationScheme interface, realising authenticate() method.
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

```java
@BeforeClass
public void before() {
    BasicAuthScheme authScheme = new BasicAuthScheme();
    authScheme.setUserName("postman");
    authScheme.setPassword("password");
    init(AuthorizationPostman.class, ServiceSettings.builder().authenticationScheme(authScheme).build());
    }
```
Authentication can be instantiated on service level. To do so authentication scheme should be passed to service settings at service init.
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

If new authentication is also passed on test level, service level authentication will be overrode.


## Parallel running
 ```java
 <?xml version="1.0" encoding="WINDOWS-1251"?>
 <!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
 <suite name="Http tests" data-provider-thread-count="3">
     <test name="parallel classes running" parallel="classes" thread-count="4">
         <classes>
             <class name="com.epam.jdi.httptests.examples.custom.ResponseTests"/>
             <class name="com.epam.jdi.httptests.examples.custom.AdvancedValidationTests"/>
         </classes>
     </test>
     <test name="parallel methods running" parallel="methods" thread-count="4">
         <classes>
             <class name="com.epam.jdi.httptests.examples.custom.JSONPostTests"/>
         </classes>
     </test>
 </suite>
 ``` 
 ```java
@DataProvider(name = "createNewBoards", parallel = true)
public static Object[][] createNewBoards() {
    return new Object[][] {
            {generateBoard()},
            {generateBoard()},
            {generateBoard()}
    };
}

@Test(dataProvider = "createNewBoards", threadPoolSize = 3)
public void createCardInBoard(Board board) {
    //Crеate board
    Board createdBoard = service.createBoard(board);
    Board gotBoard = service.getBoard(createdBoard.id);
    assertEquals(gotBoard.name, createdBoard.name, "Name of created board is incorrect");   
}
 ``` 
 Here is an example of the parallel tests running based on the TestNG approach. 
 <br />
 <br />
**Setting TestNG xml suite file for the parallel tests running**  

  1. Specify the *parallel* attribute in the tag *suite* or *test*  
  Set *parallel="methods"* to run test **methods** in separate threads 
  <br />
  Set *parallel="classes"* to run test **classes** in separate threads 
  2. Specify the number of threads allocated for the execution in the attribute *thread-count* 
  3. For Data Providers parallel running specify the count of threads in the *data-provider-thread-count* attribute of the tag *suite*
  
**Setting Data Provider for the parallel running directly in the test**  

 1. Add the attribute *parallel* = true to the @DataProvider annotation. (By default the count of threads = 10)  
 2. Specify the count of threads in the attribute *threadPoolSize* in @DataProvider annotation 

<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/resources/parallelRunningExamples.xml" target="_blank">Parallel suite example</a> 

Read more about <a href="https://testng.org/doc/documentation-main.html#parallel-running" target="_blank">TestNG parallel running</a> 




    
## Performance testing
```java
public static PerformanceResult loadService(long liveTimeSec, RestMethod... requests)
public static PerformanceResult loadService(RestMethod... requests)
public static PerformanceResult loadService(long liveTimeMSec, Map<RestMethod, Integer> weightRequests)
```

Simple performance testing is supported by JDI Dark. There is a *com.epam.http.performance* package available that contains
several classes and methods for collecting request statistics.

You can load your service and get average response time and number of fails compared to amount of requests. Just use the *loadService()* method with suitable signatures.
## Reporting
If you want to use Allure framework for reporting, JDI Dark has out-of-the-box support for generating attachments for Allure reports.
Simply install Allure as you would normally do, and those attachments will be automatically added to your reports.

All test execution results are saved to the __base_directory/allure-results__ folder.

If you use ```maven```, then you will need to configure the ```Allure maven plugin``` to fetch results from a custom folder.

```dtd
<configuration>
    <resultsDirectory>${basedir}/allure-results</resultsDirectory>
</configuration>
```

After generating allure results, the full request and response information will be attached to every test executed.

Here is an example of an Allure report with request body attached:

![Allure Request](../images/allure_request.png)
## Retry Request Option
```java
@GET(value = "503")
@RetryOnFailure(numberOfRetryAttempts = 2, delay = 1, unit = TimeUnit.SECONDS)
public static RestMethod get503;
```
Annotation @RetryOnFailure created for resending request when the service responds with one of the statuses you define.
<br><br>
For example, if status codes 502 - more likely it's temporary server-side issue and retrying request will give successful result.
<br>

```java
@GET(value = "451")
@RetryOnFailure(numberOfRetryAttempts = 6, errorCodes = 451,delay = 15, unit = TimeUnit.NANOSECONDS)
public static RestMethod get451;
```

**Annotation has parameters to specify:** 
 
1. numberOfRetryAttempts - how many times retry request after failing one.<br>
2. errorCodes - on which status codes retry request.<br>
3. delay - after what time to repeat the request.<br>
4. unit - time unit of delay time<br>

```java
@RetryOnFailure
@ServiceDomain(value = "http://localhost:8080/")
public class RetryingService {

    @GET
    public static RestMethod get502;

    @GET(value = "501")
    public static RestMethod get501;
}
```

**@RetryOnFailure can be applied to class and to field:**  

JDI Dark will merge annotations data if it's 
were placed in both places - so you need to specify annotation data only if you want to change some parameters to specific 
endpoint.<br>

**Default annotation param values are:**  
 
1. numberOfRetryAttempts - 3<br>
2. errorCodes - {502,503}<br>
3. delay - 10<br>
4. unit - TimeUnit.MICROSECONDS
<br>

```java
@GET(value = "502")
@IgnoreRetry
public static RestMethod ignoreRetrying;
```

**@RetryOnFailure placed on class can be ignored by adding @IgnoreRetry to field:**  

In that case even if status code will be in specified list of errorCodes no retry requests will be send.

## Access RestAssured

### Accessing RestAssured.config
Sometime, for testing purposes, some RestAssured configuration properties might be changed.
You can access RestAssured from test or define desired properties in @BeforeTest.

In the code snippet we have a test example using default headers set in RestAssured config.
In order to restore the initial RestAssured config after test execution, the ```reset()``` method is called.

*For configuring one request settings see [Request data - RequestSpecification config](https://jdi-docs.github.io/jdi-dark/#request-data)*

```java
    public void followsRedirectsWhileKeepingHeadersSpecifiedIfRestAssuredConfig() throws Exception {
        final List<Header> httpClientHeaders = new ArrayList<Header>();
        httpClientHeaders.add(new BasicHeader("header1", "value1"));
        httpClientHeaders.add(new BasicHeader("header2", "value2"));
        RestAssured.config = RestAssuredConfig.newConfig().httpClient(HttpClientConfig
                .httpClientConfig().setParam(DEFAULT_HEADERS, httpClientHeaders));
        RestResponse response = getRedirect.call(requestQueryParams("url", "multiHeaderReflect"));
        response.isOk();
        response.assertThat().header("header1", equalTo("value1"))
                .header("header2", equalTo("value2"));
        RestAssured.reset();
    }
```

## JDI Dark BDD Steps

 JDI Dark supports writing tests in BDD style. Create your tests using next steps:

```gherkin
Feature: Request headers check

  Scenario: Pass headers and check response
    Given init service example
    When set request headers
      | Name | Katarina |
      | Id   | 1        |
    And perform 'get' request
    And print response
    Then response status type is OK
    And response parameter 'headers.Name' is 'Katarina'
    And response parameter 'headers.Id' is '1'

```

**Actions**
 
*When* perform "\<METHOD\>" request  
*When* set request content type to "\<CONTENT-TYPE\>"  
*When* perform "\<METHOD NAME\>" request with named path parameters "\<PATH PARAMETERS\>"  
*When* perform "\<METHOD NAME\>" request with query parameters "\<QUERY PARAMETERS\>"  
*When* set request headers:  
    |\<GHERKIN DATA TABLE\>|  
*When* print response   
*When* load service for "\<SECONDS\>" seconds with "\<METHOD NAME\>" request  
*When* print number of performance results requests    

**Validations**

*Then* "\<METHOD NAME\>" method is alive 
*Then* response status code is "\<STATUS CODE\>"  
*Then* response body is empty  
*Then* response status type is "\<TYPE\>"  
*Then* response parameter "\<PARAMETER\>" is "\<VALUE\>"  
*Then* response parameter "\<PARAMETER\>" contains "\<VALUE\>"  
*Then* response body has values:  
   |\<GHERKIN DATA TABLE\>|  
*Then* the average response time is less than "\<SECONDS\>" seconds  
*Then* response header "\<HEADER\>" is "\<VALUE\>"  
*Then* performance result doesn't have any fails

See more information in the <a href="https://jdi-docs.github.io/jdi-dark/#4-jdi-dark-and-cucumber" target="_blank">Tutorial</a>.
<br>
See Cucumber examples <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-bdd-tests/src/test/resources/features" target="_blank">here</a>.
<br>

##WebSockets

Use JDI Dark module *jdi-dark-ws* for working with sockets.

###JDI Dark methods for WebSockets 

For working with sockets create the WebSocket class extending the JdiWSServer class. Mark it with the annotation @ServerEndpoint("")

There are available methods for working with sockets:  

```java
@ServerEndpoint("")
public class TrelloSocket extends JdiWSServer {

}

public class TrelloTest {
    private TrelloSocket trelloSocket = new TrelloSocket();

    @Test
    public void checkMessages() throws IOException, InterruptedException, DeploymentException, URISyntaxException {
        trelloSocket.connect("wss://trello.com/1/Session/socket?token=" + token);
        trelloSocket.sendMessage("{\"type\":\"subscribe\",\"modelType\":\"Member\",\"idModel\":\"5e8ef65b384f806fbb911f5d\",\"tags\":[\"messages\",\"updates\"],\"invitationTokens\":[],\"reqid\":5}");
        trelloSocket.waitNewMessage(30);     
    }
}
```
|Method | Description | Return Type
--- | --- | ---
**connect(URI path)** | connect to the URI | void
**connect(String path)** | connect to specified path | void
**closeSession()** | close the session | void
**sendMessage(String text)** | send the web-socket message as a String| void
**sendMessage(Object object)** | send the web-socket message as an Object| void
**sendBinary(ByteBuffer data)** | send the web-socket message as a binary data | void
**waitNewMessage(int sec)** | wait for the message | void
**waitAndGetNewMessage(int sec)** | wait for the message and return it | JsonElement
**waitNewMessages(int count, int sec)** | wait for the messages | void
**waitNewMessageMatches(String regex, int maxMsgCount, int sec)** | wait the message which matches regexp | void
**waitNewMessageContainsText(String text, int maxMsgCount, int sec)** | wait the message which contains the specified text | void
**waitNewMessageContainsKey(String key, int maxMsgCount, int sec)** |wait the message which contains the specified JSON key| void
**getNewMessageAsJsonObject()** | get the message as Object | JsonObject
**clearMessages()** | clear the received messages| void

See test examples <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/websockettest/TrelloTest.java" target="_blank">here</a>

## SOAP
###Creating Service Object

```java
@ServiceDomain("https://geoservices.tamu.edu/Services/Geocode/WebService/GeocoderService_V04_01.asmx")
@SOAPNamespace("https://geoservices.tamu.edu/")
public class GeoServices {

    @POST()
    @SOAPAction("https://geoservices.tamu.edu/GeocodeAddressNonParsed")
    public static SoapMethod<GeocodeAddressNonParsed, GeocodeAddressNonParsedResponse> geocodeAddressNonParsed;

    @POST()
    @SOAP12
    public static SoapMethod<GeocodeAddressNonParsed, GeocodeAddressNonParsedResponse> geocodeAddressNonParsed12;
}

```
Available annotations for Service Object:

|Annotation | Description | 
--- | --- | ---
**@ServiceDomain("value")** | represents the domain name |
**@POST()** |  represents HTTP post method | 
**@URL("value")** | represents URL for SOAP request| 
**@SOAPNamespace("value")** | represents SOAP namespace | 
**@SOAP12** | represents SOAP 1.2 | 
**@SOAPAction("value")** | add SOAPAction header to the SOAP request |
   
See service examples <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/main/java/com/epam/jdi/soap" target="_blank">here</a>.

###Creating Objects

Generate objects from your WSDL scheme using <a href="https://jdi-docs.github.io/jdi-dark/#soap-objects-generation-from-wsdl" target="_blank">JDI Dark generator</a>.


###JDI Dark Methods for SOAP
For working with SOAP use **SoapMethod \<T, S\> class** in package *com.epam.http.requests*, where T - class describing the body of sending message, S - class describing body of the received message.

Available methods for SoapMethod:

|Method | Description | Return Type  
--- | --- | ---  
**callSoap(T object)** | makes SOAP request and get SOAP answer | S Object |
**withSoapHeader(Object header)** | adds header to the SOAP message| SoapMethod \<T, S\> object |
 
###Creating tests
```java
public class GeoServicesTests {

    @BeforeTest
    public void before() {
        init(GeoServices.class);
    }

    @Test
    public void negativeGeocodeAddressNonParsed() {
        GeocodeAddressNonParsedResponse response = GeoServices.geocodeAddressNonParsed.callSoap(new GeocodeAddressNonParsed()
                .withStreetAddress("9355 Burton Way")
                .withCity("Beverly Hills")
                .withState("ca")
                .withZip("90210")
                .withApiKey("wrong")
                .withVersion(4.01)
                .withShouldCalculateCensus(true)
                .withCensusYear(CensusYear.ALL_AVAILABLE)
                .withShouldNotStoreTransactionDetails(false)
        );
        Assertions.assertThat(response.getGeocodeAddressNonParsedResult().getQueryStatusCodes()).isEqualTo(QueryStatusCodes.API_KEY_INVALID);
    }

}
```
Service object class can be initialized in tests. Fields of initialized object can be used to send requests from tests.

See test examples <a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples/soap" target="_blank">here</a>.

## Review Guide

### Easy way to pass review
+ Good code style
+ `Ctrl+Alt+O` & `Ctrl+Alt+L` (replace ```Ctrl``` with ```Cmd``` on MAC) in IDEA IDE before any commit
+ Right branch and label in PR

### Short checklist
1. Branch
2. Label
3. Codacy
4. *Other checks
5. Modified files

### Detailed list
1. Codacy check should pass. If it doesn't, you need to check what's wrong.
2. *When Travis will be fixed
3. Also, while reviewing files, you can mark them with checkboxes. The following should be checked:
   + Code style
   + Understandable naming
   + Unnecessary empty lines and spaces
   + Unnecessary or obsolete comments
   + Chaining should be separated by lines

### Recommended tools
1. Intellij IDEA
2. Git

### Using libraries
1. Log library: slf4j + logback-classic (TO_VERIFY)
2. Test runner: TestNG (TO_VERIFY)
3. Reporter: Allure Framework (TO_VERIFY)
4. Test framework: JDI (TO_VERIFY)

### Intellij IDEA Preferences
1. Preferences -> Version Control -> Commit Dialog > Before commit block, check boxes: Reformat Code, Rearrange code, Optimize imports
2. Inside the app press hotkeys: `Ctrl+Shift+Alt+L` (`Cmd+Shift+Alt+L` for MAC), check boxes: Optimize imports, Rearrange Code
3. When you work on applying autoreformatting, use (for file / directory) `Ctrl-Alt-L` combination
4. To open any class instantly by its name, use `Ctrl+N` (`Cmd+O` for MAC)
5. Be sure that the following are NOT checked: Settings -> Editor -> Code Style -> Java -> Tabs and Indents -> Use tab character
6. Be sure that the following equals to 4: Settings -> Editor -> Code Style -> Java -> Tabs and Indents -> Tab size field and Indent field
7. Turn off wildcard imports: set  the following to 999: Settings -> Editor -> Code style -> Java -> Imports -> Class count to use imports with `*` field and Names count to use static imports with `*`

### CVS section
1. Use Git
   + Name branches with useful names 
2. Merge code into any branch only through GitHub pull request (PR)
   + Name PR with useful name
3. Before adding reviewers to your PR, make sure: 
   + there are no codacy issues
   + your code is ready for review
   + you have pushed latest changes
   + you have merged latest changes from your target branch
   + there are no merge conflicts
   + you have addressed all review comments (if applicable)
4. Add DarkReviewTeam as reviewers or re-request review (if there are already reviewers in PR)
5. When you get a request for review changes you should resolve all issues and re-request code review from the same reviewers
6. Repeat until branch is merged by reviewers
7. Keep your **.gitignore** file up-to-date

### Code style
1. Use spaces, 1 tab size equals to 4
2. Don't use wildcard imports
3. Use Intellij's autoreformatting, import optimization and rearrange option
4. Fix all Codacy issues
5. If Codacy issue shouldn't be resolved, use warning suppression
6. Use semantic names for classes , variables, methods and so on. Use full versions of words (e.g. *human* instead of *h* and so on). 
Common abbreviations (such as id, uid and so on are allowed). 
Use meaningful names, for example instead of *accountList* use *accounts* and so on.
7. Remove double empty lines, use empty lines only when it makes sense (in addition to to autoreformatting)
8. Use line breaks correctly: (PROVIDE_SAMPLES)
9. Limit line width to 100-120 symbols (including indents) 
10. Use chaining style, recommended sample:
```java 
object
    .method(1)
    .otherMethod();
```
11. Always use `{}` braces even for empty ```if```, ```for```, etc.
12. Follow the sequence of keywords: public protected private abstract default static final transient volatile synchronized native strictfp
13. Don't use conveniences such as `mValue`, `_value` and alike. `__` is acceptable as an unused argument placeholder of test method data provider.
14. Feel free to smart refactor your code.
15. Feel free to use ```?:```
16. Use `UpperCamelCase` naming style for classes, `lowerCamelCase` for fields and methods, `UPPERCASE` for enums, `lowercase` for packages
17. Always use `@Override` annotation
18. Don't leave `catch{}` blocks empty, instead use relevant exceptions with messages
19. Try to not use global variables
20. Use variables for integer, string and other values (for instance, `int piNumber=3.14; println(piNumber)`)
21. Write comments in English
22. It's better to write code than create a task to write code
23. It's better to create a task and a `TODO` with this task number than to simply write `TODO` in code
24. It's better to write a simple understandable `TODO` in code than to tell someone about a problem
25. It's better to tell someone about a problem than to not take action at all
26. Always end your files with a newline
