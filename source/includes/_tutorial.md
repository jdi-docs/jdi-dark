# Simple JDI Dark examples
## 1. Service Object Model
JDI Dark provides capability to describe your API as a simple, as we call it, Service Object.
That gives you an ability to unify and combine your REST methods and endpoints in one class
(similar to UI Page Object pattern).

Let's say we have a website for REST testing located by the following address: http://httpbin.org/ and we need to test some of its endpoints. We can create the Java class and describe methods we want to test in a following way:

```java
@ServiceDomain("http://httpbin.org/")
public class ServiceExample { 
    @GET("/get") RestMethod get;
    @POST("/post") RestMethod post;
    @PUT("/put") RestMethod put;
    @PATCH("/patch") RestMethod patch;
    @DELETE("/delete") RestMethod delete;
}
```
So simple, isn't that right?

## 2. Create simple test using Service Object
Some text here...

```java
@Test
public void getTest() {
  test 
}
```

... and here

link to tests [jdi dark tests](https://github.com/jdi-testing/jdi-dark/tree/master/jdi-httpTests)

## 3. Create tests without using Service Object
Again some text here...
```java
code here
```
and a bit of text ...

## 4. Query Parameters, Headers, Cookies support
Some explanation...
```java
java code here
```
tons of text

## 5. Performance testing support
text
```java
code examples
```

text
