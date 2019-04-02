# Documentation
## Framework usage

```
import static com.epam.http.requests.ServiceInit.init;
```

For effective usage of JDI Dark framework it is recommended to statically import the init() method of ServiceInit class.

This is an entry point for initialising your Service Object class.
## HTTP methods
JDI Dark supports the following HTTP methods:

 - GET
 - POST
 - PUT
 - PATCH
 - DELETE
 - OPTIONS
 - HEAD

For all of these methods there are annotations named after methods in *com.epam.http.annotations.** package.
All of these annotations take a value of URI to make request call to. Those annotations are supposed to be used in your Service Object class.

To point the base URI of your service it's convenient to use *@ServiceDomain* annotation with provided URL as a value.
Then values in your methods annotations might be just specific URL paths.
## Request Data
You might need to use specific request data in your requests. Cookies, headers, query parameters and Content-type are available in annotation form.
So you can specify them in your Service Object class with provided name and value.

There is a class *com.epam.http.requests.RequestData.** which represents the data being sent with request.

It is also possible to specify request data when making a request call.
### Request body

```java
public static RequestData requestBody(String body)
public static RequestData requestData(JAction1<RequestData> valueFunc)
 ```
 
Request body can be set when making a request call. Just pass it as argument to the *call()* method or within the RequestData object,
that contains the next fields:

 - String url
 - String body
 - ContentType contentType
 - MapArray<String, String> headers
 - MapArray<String, String> pathParams
 - MapArray<String, String> queryParams
 - MapArray<String, String> cookies

All of these fields can be set/updated from call() method as well.

You may need to statically import *com.epam.http.requests.RequestData.** package.

 ```java
public RestResponse call(RequestSpecification requestSpecification) 
 ```

There is also support to make a request call with Rest Assured request specification.
### Path parameters

```java
public static RequestData requestPathParams(String paramName, String paramValue) 
public static RequestData requestPathParams(Object[][] params)
```

A URL can have several path parameters, each denoted with curly braces, e.g. */get/{board_id}*. You can use them in your Service Object methods and 
replace it with values when making a request call.

There is method *requestPathParams()* provided for such case, where arguments are path parameters name and value.
## Tests without Service Object

```java
public static RestResponse GET(RequestData data)
public static RestResponse GET(String url)
public static RestResponse GET(String url, RequestSpecification requestSpecification) 
```

All the HTTP methods mentioned before are available not only in annotation form.
There are the same methods accessible from *com.epam.http.requests.RestMethods.** with several signatures.

You can call these methods with either of given arguments:

 - *com.epam.http.requests.RequestData*
 - String URL
 - Rest Assured request specification: *io.restassured.specification.RequestSpecification*

## Response data

```java
public ValidatableResponse isOk()
public ValidatableResponse hasErrors()
public ValidatableResponse isStatus(ResponseStatusType type)
public RestResponse assertStatus(ResponseStatus rs)
```

```java
public ValidatableResponse assertBody(MapArray<String, Matcher<?>> params)
public ValidatableResponse assertBody(Object[][] params)
```

```java
public String getFromHtml(String path)
```

```java
public Response raResponse()
public ValidatableResponse assertThat()
```
The class *com.epam.http.response.RestResponse* represents the Response data in JDI Dark.

There are methods for checking status code of response. Methods *isOk()* and *hasErrors()* verify that status code is 2** or 4**, respectively.

Response body can be verified using method *assertBody()*.

Methods *cookie()* and *headers()* return response cookies and headers values.

Method *getFromHtml()* returns html content of the page by provided path.

Methods *raResponse* and *assertThat()* returns Rest Assured Response and ValidatableResponse, so it is possible to use Rest Assured methods for validating the response body.
## Deserialization
```java
public T callAsData(Class<T> c)
public T asData(Class<T> c)
```

JDI Dark provides support for deserialization response into Java object. Just use method *callAsData()* instead of *call()* and provide the class as argument.
## Performance testing
```java
public static PerformanceResult loadService(long liveTimeSec, RestMethod... requests)
public static PerformanceResult loadService(RestMethod... requests)
public static PerformanceResult loadService(long liveTimeMSec, Map<RestMethod, Integer> weightRequests)
```

Simple performance testing is supported by JDI Dark. There is *com.epam.http.performance* package available that contains
several classes and methods for collecting requests statistics.

You can load your service and get average response time and number of fails comparing to amount of requests. Just use *loadService()* method with suitable signature. 
## Reporting
If you want to use Allure framework for reporting, JDI Dark has out-of-the-box support for generating attachments for Allure reports.
Just simply install Allure as you would always do, those attachments will be automatically added to report.

All test execution results are being saved to __base_directory/allure-results__ folder.

If you use maven, then you will need to configure Allure maven plugin to take results from appropriate folder.

```dtd
<configuration>
    <resultsDirectory>${basedir}/allure-results</resultsDirectory>
</configuration>
```

After generating allure results, the full request and response information will be attached to every test that was executed.

Here is the example of Allure report with attached request body:

![Allure Request](../images/allure_request.png)
