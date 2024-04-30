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
   
