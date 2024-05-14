1. Sonar uses jacoco for taking coverage report, but regarding exclusions it has separate property i.e sonar.coverage.exclusions

```
<properties>
    <sonar.coverage.exclusions>
      **/com/sabre/wolverine/order/revenueaccount/configuration/**/*,
      **/com/sabre/wolverine/order/revenueaccount/GenericSpringBootApplication.java
    </sonar.coverage.exclusions>
  </properties>
```

"**/*" - will exclude child packages as well.
For more info - link[https://stackoverflow.com/questions/12135939/how-to-make-sonar-ignore-some-classes-for-codecoverage-metric/27133765#27133765]

2. verify(mock,times(1)) can be replaced with verify(mock) as it is default behaviour.<br>

    **Code Snippet from Mockito.java class**
    ```
    public static <T> T verify(T mock) {
        return MOCKITO_CORE.verify(mock, times(1));
    }
    ```
    
3. For Integration tests we need to give the following properties - <br>
```
@SpringBootTest(
    properties = {
        "grpc.server.inProcessName=under-test-process",
        "grpc.server.port=-1",
        "grpc.client.revenue-account.address=in-process:under-test-process"
    }
)
```

These properties are used to configure the gRPC server and client for the integration test.

**"grpc.server.inProcessName=under-test-process":** This property sets the name of the in-process server. An in-process server is a gRPC server that is created for the purpose of the test and runs within the same process as the test.  

**"grpc.server.port=-1":** This property sets the port for the gRPC server. The value -1 is used to indicate that the server should not listen on any TCP port. This is useful in testing scenarios where you don't want the server to open any actual network ports.  

**"grpc.client.revenue-account.address=in-process:under-test-process":** This property sets the address of the gRPC client. The value in-process:under-test-process indicates that the client should connect to an in-process server with the name under-test-process. This is useful in testing scenarios where you want the client and server to run in the same process.
<br>
what is SpringBootTest annotation-<br>
<br>
From java documentation:<br>
Annotation that can be specified on a test class that runs Spring Boot based tests. Provides the following features over and above the regular Spring TestContext Framework:<br>
<br>
    1. Uses SpringBootContextLoader as the default ContextLoader when no specific @ContextConfiguration(loader=...) is defined.<br>
    2. Automatically searches for a @SpringBootConfiguration when nested @Configuration is not used, and no explicit classes are specified.<br>
    3. Allows custom Environment properties to be defined using the properties attribute.<br>
    4. Allows application arguments to be defined using the args attribute.<br>
    5. Provides support for different webEnvironment modes, including the ability to start a fully running web server listening on a defined or random port.<br>



