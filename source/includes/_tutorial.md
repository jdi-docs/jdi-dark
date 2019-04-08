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
That gives you an ability to unify and combine your REST methods and endpoints in one class
(similar to UI Page Object pattern).

Let's say we have a website for REST testing located by the following address: [http://httpbin.org](http://httpbin.org/) and we need to test some of its endpoints.
We can create the Java class and describe methods we want to test.

*So simple, isn't that right?*

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

Based on the Service Object class we created before, we can create a simple test to exercise HTTP GET method and validate the response body.

1. Make a request call to HTTP method described in your Service Object class.
2. Validate the response body.

The HTTP GET method described in service class is being called here. Next, the response is being validated.

You can check other tests by the following link:
 
 [JDI Dark test examples](https://github.com/jdi-testing/jdi-dark/tree/master/jdi-httpTests)

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
It is possible to call HTTP methods directly from your tests. You just need to provide required information within the method arguments.

In this test we make an HTTP GET request with invoked request data, which contains URL and headers.

## 4. Query Parameters, Headers, Cookies support
```java
@QueryParameter(name = "test", value = "test")
@GET("/get") RestMethod getInfo;
```

```java
@QueryParameters({
    @QueryParameter(name = "param1", value = "test"),
    @QueryParameter(name = "param2", value = "test")
})
@GET("/get") RestMethod getInfo;
```

```java
@Headers({
    @Header(name = "Name", value = "Roman"),
    @Header(name = "Id", value = "Test")
})
@Cookie(name = "session_id", value = "1234")
@GET("/get") RestMethod getInfo;
```

JDI Dark has support for query parameters, you can specify them in your service class.
You just need to use *@QueryParameter* annotation in your method or service class description.

If you need to specify several query parameters, then use *@QueryParameters* annotation, where you can put all of your request parameters.

Just the same way you can specify headers and cookies. JDI Dark has special annotation for them as well.

## 5. Performance testing support
```java
 @Test
public void simplePerformanceTest() {
    PerformanceResult pr = loadService(3600, getInfo);
    System.out.println("Average time: " + pr.AverageResponseTime + "ms");
    System.out.println("Requests amount: " + pr.NumberOfRequests);
    Assert.assertTrue(pr.NoFails(), "Number of fails: " + pr.NumberOfFails); 
}
```

There is the support for measuring time your requests take.

It is possible to call your service several times and get the descriptive results.

You can load your service and analyse the response time.
