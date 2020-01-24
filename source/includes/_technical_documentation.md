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
## Access RestAssured
TBD

## Review Guide

### Easy way to pass review
+ Good code style
+ No Wildcards imports (with asterisk *). You can use settings from the picture below
![Wildcard Imports](../images/review-wildcard-imports.png)
+ `Ctrl+Alt+O` & `Ctrl+Alt+L` in IDEA IDE before any commit
+ Right branch and label in PR

### Short checklist
1. Branch
2. Label
3. Codacy
4. *Other checks
5. Changed files

### Detailed list
1. Codacy check should be OK. If not then you need to see what's wrong
2. *When Travis would be fixed
3. Also reviewing files you can mark them with checkboxes. Should be checked:
   + Code style
   + Understandable naming
   + Unnecessary empty lines and spaces
   + Unnecessary or forgotten comments
   + Chaining should be separated by lines

### Recommended tools
1. Intellij IDEA
2. Git

### Using libraries
1. As log library: slf4j + logback-classic (TO_VERIFY)
2. As tests runner: TestNG (TO_VERIFY)
3. As reporter: Allure Framework (TO_VERIFY)
4. As test framework: JDI (TO_VERIFY)

### Intellij IDEA Preferences
1. Preferences -> Version Control -> Commit Dialog > Before commit block, setup checkboxes: Reformat Code, Rearrange code, Optimize imports
2. In app press hotkeys: `Ctrl+Shift+Alt+L`, choose checkboxes: Optimize imports, Rearrange Code
3. When you work to apply autoreformatting please use (on file, on directory or inside file) `Ctrl-Alt-L` hotkey
4. To fast open any class by its name use `Ctrl+N`/`Cmd+O`
5. Be sure that NOT checked: Settings->Editor-> Code Style -> Java -> Tabs and Indents -> Use tab character
6. Be sure that equals to 4: Settings->Editor-> Code Style -> Java -> Tabs and Indents -> Tab size field and Indent field
7. Turn off wildcard imports: set to 999 for Settings -> Editor -> Code style -> Java -> Imports -> Class count to use imports with `*` field and Names count to use static imports with `*`

### CVS section
1. Use Git
   + Name branches with useful names 
2. Merge code into any branch only through GitHub pull request (PR)
   + Name PR with useful name
3. Before add reviewers to your PR be ensure: 
   + no codacy issues
   + code is ready to review
   + you push the last changes
   + you have last changes from target branch
   + there is no merge conflicts
   + you take into account all review comments (if such were)
4. Add DarkReviewTeam as reviewers or re-request review (if there is already reviewers in PR)
5. When you get review changes request you should resolve all issues and re-request code review from the same reviewers
6. Repeat until branch will be merged by reviewers
7. Keep actual **.gitignore** file

### Code style
1. Use spaces, 1 tab size equals to 4
2. Don't use wildcard imports
3. Use Intellij's autoreformatting, imports optimization and rearrange option
4. Fix all Codacy issues
5. If Codacy issue shouldn't be resolved use warnings suppressions
6. Use semantic-value names for classes , variables, methods and so on. Use full version of words (human instead of h and so on). Common abbreviations (as id, uid and so on are applied). Use valuable names, for example instead of accountList use accounts and so on
7. Remove double empty lines, use empty lines only when it make sense (additionally to autoreformatting)
8. Use correct line breaks: (PROVIDE_SAMPLES)
9. Try to use lines with no more 100-120 symbols (including indents) 
10. Use chaining style, recommended sample:
```java 
object
    .method(1)
    .otherMethod();
```
11. Always use `{}` braces for even empty if, for, etc.
12. Follow to the sequence of keywords: public protected private abstract default static final transient volatile synchronized native strictfp
13. Don't use such conveniences as: `mValue`, `_value`, and so on. Acceptable `__` as unused argument of data provider in test methods
14. Feel free to smart refactoring
15. Feel free to use smart ?:
16. Use `UpperCamelCase` naming style for classes, `lowerCamelCase` for fields and methods, `UPPERCASE` for enums, `lowercase` for packages
17. Always use `@Override` annotation
18. Don't leave empty `catch{}`, use relevant exceptions with messages
19. Try to not use global variables
20. Use variables for integer, string and other values (for instance, `int piNumber=3.14; println(piNumber)`)
21. Write comments in English
22. It's better to write code than create task to write code
23. It's better to create task and `TODO` with this task number than to write simple `TODO` in code
24. It's better to write simple understandable `TODO` in code than to tell about problem to someone
25. It's better to tell about a problem to someone than to not take actions at all
26. Always leave newline symbol as a last symbol of file