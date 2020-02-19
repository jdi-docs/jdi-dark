# Documentation
## Framework usage

```
import static com.epam.http.requests.ServiceInit.init;
```

In order to effectively use the JDI Dark framework, it is recommended to statically import the ```init()``` method of the *ServiceInit* class.

This is the entry point for initialising your Service Object class.
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
## Request Data
You might need to use specific request data in your requests. *Cookies*, *headers*, *query parameters* and *Content-type* are available in annotated form.
Therefore, you can specify them in your Service Object class providing name and value.

There is a class called *com.epam.http.requests.RequestData.** which represents the data sent along with your request.

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
 - MultiMap<String, String> pathParams
 - MultiMap<String, String> queryParams
 - MultiMap<String, String> formParams
 - Cookies cookies
 - ArrayList<MultiPartSpecification> multiPartSpecifications

All of these fields can be set/updated from the *call()* method as well.

You may need to statically import the *com.epam.http.requests.RequestData.** package.

 ```java
public RestResponse call(RequestSpecification requestSpecification) 
 ```

JDI Dark also supports making request calls with Rest Assured request specification.
### Path parameters

```java
@GET("/{firstName}/{lastName}")
public static RestMethod getUser;

@Test
public void supportsPassingPathParamsToRequestSpec(){
    RestResponse response = getUser
            .call(requestPathParams(new Object[][]{{"firstName", "John"}, {"lastName", "Doe"}}));
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
            .call(requestData(d -> d.pathParams.addAll(params)));
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

There are methods provided for passing path params to RequestData:

|Method | Description | Return Type
--- | --- | ---
**requestPathParams(String paramName, String paramValue)** | pass one parameter to a path | RequestData
**requestPathParams(Object[][] params)** | pass multiple parameters to a path | RequestData

Methods for passing path params (with/without query params) in RestMethod:

|Method | Description | Return Type
--- | --- | ---
**callWithNamedParams(String... namedParams)** | pass parameters to a path without key| RestResponse

<br>
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/PathParamTests.java" target="_blank">Test examples in Java</a>
<br>
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

### Parameters

```java
@POST("/greet")
public static RestMethod greetPost;

@Test
public void charsetIsReallyDefined() {
    RestResponse resp = greetPost.call(requestData(rd -> {
        rd.contentType = "application/x-www-form-urlencoded; charset=ISO-8859-1";
        rd.formParams.add("firstName", "Some & firstname");
        rd.formParams.add("lastName", "<lastname>");
    }));
    resp.isOk().assertThat().body("greeting", equalTo("Greetings Some & firstname <lastname>"));
}

@GET("/greet")
public static RestMethod getGreet;

@Test
public void whenLastParamInGetRequestEndsWithEqualItsTreatedAsANoValueParam() {
    JettyService.getGreet.call(requestData(d -> {
        d.queryParams.add(FIRST_NAME, FIRST_NAME_VALUE);
        d.queryParams.add(LAST_NAME, "");
    })).isOk().assertThat().body("greeting", equalTo("Greetings John "));
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

@DELETE("/greet")
public static RestMethod deleteGreet;

@Test
public void bodyHamcrestMatcherWithOutKey() {
    deleteGreet.call(requestQueryParams(
            new Object[][]{{FIRST_NAME, FIRST_NAME_VALUE},
                    {LAST_NAME, LAST_NAME_VALUE}
            })).isOk().assertThat().body(equalTo("{\"greeting\":\"Greetings John Doe\"}"));
}

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

JDI Dark supports query, form, multipart parameters addition in the request.

Method allows to send the request with invoked request data in RestMethod:

|Method | Description | Return Type
--- | --- | ---
**call(RequestData requestData)** | make request with parameters indicated by Request Data| RestResponse


Methods allow to set multipart parameters to request data:

|Method | Description | Return Type
--- | --- | ---
**setMultiPart(MultiPartSpecBuilder multiPartSpecBuilder)** | set multipart parameters | 
**setMultiPart(File file)** | set File parameter | 

Methods allow to send query params to RequestData:

|Method | Description | Return Type
--- | --- | ---
**requestQueryParams(String paramName, String paramValue)** | pass one query parameter to a path | RequestData
**requestQueryParams(Object[][] params)** | pass multiple query parameters to a path | RequestData

Method allow to send specific query parameters in url in RestMethod:

|Method | Description | Return Type
--- | --- | ---
**call(String queryParams)** | pass query parameters | RestResponse

<br>
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/ParamTest.java" target="_blank">Test examples in Java</a>
<br>

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
    @Headers({
        @Header(name = "Name", value = "Roman"),
        @Header(name = "Id", value = "Test")
    })
    static RestMethod<Info> getInfo;

    @Header(name = "Type", value = "Test")
    @POST("/post")
    RestMethod postMethod;

    @PUT("/put") RestMethod putMethod;
    @PATCH("/patch") RestMethod patch;
    @DELETE("/delete") RestMethod delete;
    @GET("/status/%s") RestMethod status;

} 
```

It's possible to describe tested web service as a Service Object class using annotations.
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

### Single header can be added with usage of @Header annotation:

```java
@GET("/header")
@Header(name = "Header_name", value = "Header_value2")
public static RestMethod getHeader;
```
Here's a simple example of adding a header to an endpoint
<br />
<br />
<br />


### Multiple headers are also supported through @Headers annotation:

```java
@GET("/multiHeaderReflect")
@Headers({@Header(name = "Header_name", value = "Header_value"),
@Header(name = "Header_name2", value = "Header_value2")})
public static RestMethod getMultiHeaderReflect;
```
<br />
<br />
<br />
<br />

### Headers with same name, with no value and with multiple values can be added as well.

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
@Headers annotation is used to add 2 headers with the same name - "Header_name"
<br />
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

### Methods to add Headers to Request Data. 
Headers can be passed as strings, header objects, maps, and even arrays of objects.
Headers without value and with multiple values can be added as well.

```java
public RequestData addHeader(String name, String value, String... additionalValues)
public RequestData addHeader(String name)
public RequestData addHeaders(String name, Object value, Object... cookieNameValuePairs)
public RequestData addHeaders(Object[][] objects)
public RequestData addHeaders(MapArray mapArray)
public RequestData addHeaders(Map map)
public RequestData addHeaders(List<Header> headers)
public RequestData addHeaders(Header... headers)

RestResponse response = MyService.getHello.call(requestData(
                requestData -> requestData.addHeader("Header_example", "Test_value")));
```

|Method | Description | Return Type
--- | --- | ---
**addHeader(String name, String value, String... additionalValues)** | pass name and value of header. If additional values are specified, then several cookies will be created with the same name (multiple-value header) | RequestData
**addHeader(String name)** | pass header name without value | RequestData
**addHeaders(Object[][] objects)** | pass array with header names and values | RequestData
**addHeaders(MapArray mapArray)** | pass MapArray with header names and values | RequestData
**addHeaders(Map map)** | pass map with header names and values | RequestData
**addHeaders(Map map)** | pass map with header names and values | RequestData
**addHeaders(List list)** | pass list of headers | RequestData
**addHeaders(Header... header)** | pass header object | RequestData

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
Cookies without value and with multiple values can be added as well.

|Method | Description | Return Type
--- | --- | ---
**addCookie(String name, String value, String... additionalValues)** | pass name and value of cookie. If additional values are specified, then several cookies will be created with the same name (multiple-value cookie) | RequestData
**addCookie(String name)** | pass cookie name to cookie without value | RequestData
**addCookies(String name, Object value, Object... cookieNameValuePairs)** | pass several pairs of cookie names and values | RequestData
**addCookies(Object[][] objects)** | pass array with cookie names and values | RequestData
**addCookies(MapArray mapArray)** | pass MapArray with cookie names and values | RequestData
**addCookies(Map map)** | pass map with cookie names and values | RequestData

```java
public RequestData addCookie(String name, String value, String... additionalValues)
public RequestData addCookie(String name)
public RequestData addCookies(String name, Object value, Object... cookieNameValuePairs)
public RequestData addCookies(Object[][] objects)
public RequestData addCookies(MapArray mapArray)
public RequestData addCookies(Map map)

RestResponse response = MyService.getHello.call(requestData(
                requestData -> requestData.addCookie("key1", "value1")));
```


## Delete

For sending a DELETE request you can use the RestMethod class with a @DELETE annotation.

```java
@DELETE("/body")
public static RestMethod deleteBody;
@DELETE("/cookie")
public static RestMethod deleteCookie;

@Test
public void deleteSupportsStringBody() {
    RestResponse response = deleteBody.call(requestBody(TEST_BODY_VALUE));
    response.assertThat().body(is(TEST_BODY_VALUE));
}

@Test
public void requestSpecificationAllowsSpecifyingCookie() {
    RestResponse response = deleteCookie.call(requestData(requestData ->
            requestData.addCookies(new MapArray<>(new Object[][]{
                    {USERNAME, FIRST_NAME_VALUE},
                    {TOKEN, TOKEN_VALUE}
            }))));
    assertEquals(response.body, "username, token");
}


```

|Method | Description | Return Type
--- | --- | ---
**call(RequestData requestData)**| make request with Request Data parameters | RestResponse

<br>
<a href="https://github.com/jdi-testing/jdi-dark/blob/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/DeleteTest.java" target="_blank">Test examples in Java</a>
<br>

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
## Access RestAssured
TBD

### Accessing RestAssured.config
Sometime, for testing purposes, some RestAssured configuration properties might be changed.
You can access RestAssured from test or define desired properties in @BeforeTest.

In the code snippet we have a test example using default headers set in RestAssured config.
In order to restore the initial RestAssured config after test execution, the ```reset()``` method is called.

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

## Review Guide

### Easy way to pass review
+ Good code style
+ No Wildcard imports (with asterisk *). You can use settings from the picture below
![Wildcard Imports](../images/review-wildcard-imports.png)
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
