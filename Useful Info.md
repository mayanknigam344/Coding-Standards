**1. Sonar uses jacoco for taking coverage report, but regarding exclusions it has separate property i.e sonar.coverage.exclusions**

```
<properties>
    <sonar.coverage.exclusions>
      **/com/sabre/wolverine/order/revenueaccount/configuration/**/*,
      **/com/sabre/wolverine/order/revenueaccount/GenericSpringBootApplication.java
    </sonar.coverage.exclusions>
  </properties>
```

"**/*" - will exclude child packages as well.
For more info - link[https://stackoverflow.com/questions/12135939/how-to-make-sonar-ignore-some-classes-for-codecoverage-metric/27133765#27133765]<br>
<br>
**2. verify(mock,times(1)) can be replaced with verify(mock) as it is default behaviour**<br>

    **Code Snippet from Mockito.java class**
    ```
    public static <T> T verify(T mock) {
        return MOCKITO_CORE.verify(mock, times(1));
    }
    ```
<br>

**3. For Integration tests we need to give the following properties** <br>

```
@SpringBootTest(
    properties = {
        "grpc.server.inProcessName=under-test-process",
        "grpc.server.port=-1",
        "grpc.client.revenue-account.address=in-process:under-test-process"
    }
)
```
<br>
These properties are used to configure the gRPC server and client for the integration test.

**"grpc.server.inProcessName=under-test-process":** This property sets the name of the in-process server. An in-process server is a gRPC server that is created for the purpose of the test and runs within the same process as the test.  

**"grpc.server.port=-1":** This property sets the port for the gRPC server. The value -1 is used to indicate that the server should not listen on any TCP port. This is useful in testing scenarios where you don't want the server to open any actual network ports.  

**"grpc.client.revenue-account.address=in-process:under-test-process":** This property sets the address of the gRPC client. The value in-process:under-test-process indicates that the client should connect to an in-process server with the name under-test-process. This is useful in testing scenarios where you want the client and server to run in the same process.
<br>
<br>
*what is SpringBootTest annotation*-<br>
<br>
From java documentation:<br>
Annotation that can be specified on a test class that runs Spring Boot based tests. Provides the following features over and above the regular Spring TestContext Framework:<br>
<br>
    1. Uses SpringBootContextLoader as the default ContextLoader when no specific @ContextConfiguration(loader=...) is defined.<br>
    2. Automatically searches for a @SpringBootConfiguration when nested @Configuration is not used, and no explicit classes are specified.<br>
    3. Allows custom Environment properties to be defined using the properties attribute.<br>
    4. Allows application arguments to be defined using the args attribute.<br>
    5. Provides support for different webEnvironment modes, including the ability to start a fully running web server listening on a defined or random port.<br><br>
**4. Jacoco not generating the report for Unit Tests(i.e Sonar coverage showing 0 coverage)** <br>

   1. Jacoco uses argline to add a **javaagent** which collects the coverage data.
   2. The JaCoCo Maven plugin uses the argLine configuration to add a Java agent for collecting code coverage data during test execution. This is done by setting 
      the argLine property with the path to the JaCoCo agent.<br>
  
      In below case jacoco was setting up property with name **surefire.jacoco.args** <br>
        ```
        <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <executions>
            <execution>
                <id>prepare-agent</id>
                <goals>
                    <goal>prepare-agent</goal>
                </goals>
                <configuration>
                    <destFile>${project.build.directory}/coverage-reports/jacoco-unit.exec</destFile>
                    <propertyName>surefire.jacoco.args</propertyName>
                </configuration>
            </execution>
        </executions>
        </plugin>
        <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <configuration>
            <argLine>@{surefire.jacoco.args}</argLine>
        </configuration>
        </plugin>
        ```
   3. It adds some line passed to java executing tests.
   4. The argLine configuration should include the -javaagent option followed by the path to the JaCoCo agent JAR file and the destfile option specifying the             location of the output file for the code coverage data.
       ```
        -javaagent:<...>org.jacoco.agent-0.8.5-runtime.jar=destfile=<...>target\\jacoco.exec
       ```
   5. But **maven-surefire-plugin which executes unit tests** were not picking it:
      ```
       <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                  <argLine>@{argLine}</argLine>  --- Issue here
                </configuration>
       </plugin>
      ```
      Because of that jacoco agent was not attached during unit test coverage resulting in no coverage report for unit test execution. So until now coverage was         taken only based on **integration tests which executes with different plugin (failsafe)**

   6. Fix will be <br>
       ```
        <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                  <argLine>@{surefire.jacoco.args}</argLine>  --- Fix here
                </configuration>
        </plugin>
       ```
