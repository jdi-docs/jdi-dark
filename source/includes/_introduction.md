# Simple JDI Dark examples
## 1. Service Object Model

```java
@ServiceDomain("http://httpbin.org/")
public class ServiceExample { 
    @GET("/get") RestMethod getInfo;
    @POST("/post") RestMethod post;
    @PUT("/put") RestMethod put;
    @PATCH("/patch") RestMethod patch;
    @DELETE("/delete") RestMethod delete;
}
```

JDI Dark provides capability to describe your API as a simple Service Object.
That allows you to unify and combine your REST methods and endpoints in one class
(similar to UI Page Object pattern).

Let's say we have a website for REST testing located at the following address: [http://httpbin.org](http://httpbin.org/) and we need to test some of its endpoints.
We can create a Java class and describe the methods we want to test.

*Isn't that simple?*

## 2. Create simple test using Service Object

```java
@Test
public void simpleRestTest() {
    RestResponse resp = ServiceExample.getInfo.call();
    resp.isOk().
      body("url", equalTo("http://httpbin.org/get")).
      body("headers.Host", equalTo("httpbin.org")).
      body("headers.Id", equalTo("Test"));
    resp.assertThat().header("Connection", "keep-alive");
}
```

Based on the Service Object class we created before, we can create a simple test to execute the HTTP GET method and validate the response body.

1. Make a request call to the HTTP method described in your Service Object class.
2. Validate the response body.

The HTTP GET method described in the service class is being called here. Next, the response is being validated.

You can check other tests by following the link:
 
 [JDI Dark test examples](https://github.com/jdi-testing/jdi-dark/tree/master/jdi-dark-tests/src/test/java/com/epam/jdi/httptests/examples)

## 3. Create tests without using Service Object

```java
@Test
public void noServiceObjectTest(){
    RestResponse resp = GET(requestData(
      rd -> {rd.url="http://httpbin.org/get";
           rd.headers=new MapArray<>(new Object[][]{
              {"Name","Roman"},
              {"Id","TestTest"}
           });}
      ));
}
```

As it has been mentioned, you are still able to write tests without using Service Object model.
It is possible to call HTTP methods directly from your tests. You just need to provide required information within method arguments.

In this test we are making an HTTP GET request with invoked request data containing URL and headers.

## 4. Query Parameters, Headers, Cookies support

```java
@QueryParameter(name = "test", value = "test")
@GET("/get") RestMethod getInfo;
```

```java
@QueryParameter(name = "param1", value = "test")
@QueryParameter(name = "param2", value = "test")
@GET("/get") RestMethod getInfo;
```

```java
@Header(name = "Name", value = "Roman")
@Header(name = "Id", value = "Test")
@Cookie(name = "session_id", value = "1234")
@GET("/get") RestMethod getInfo;
```

JDI Dark has support for query parameters, you can specify them in your service class.
You just need to use the *@QueryParameter* annotation in your method or service class description.

If you need to specify several query parameters, do it with the *@QueryParameters* annotation.

The same way you can specify headers and cookies. JDI Dark has a special annotation for them as well.

## 5. Performance testing support

```java
    @Test
    public void concurrentTest() throws InterruptedException, ExecutionException {
        PerformanceResult pr = RestLoad.loadService(5, 10, ServiceExample.getInfo);
        Assertions.assertThat(pr.getNumberOfFails()).describedAs("Fails found").isEqualTo(0);
        Assertions.assertThat(pr.getAverageResponseTime()).describedAs("The average response time is greater than expected.").isLessThan(1000);
        Assertions.assertThat(pr.getMaxResponseTime()).describedAs("The maximum response time is greater than expected.").isLessThan(3000);
    }
```
JDI Dark supports simple performance testing. There is *com.epam.http.performance* package available that contains
several classes and methods for collecting response statistics.

You can load your service and get the response time and the number of fails compared to the amount of requests. Just use the *loadService()* method with the parameters you need.
