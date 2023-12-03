**Usage of experimental lombok features**

**Description** - Do not use experimental features. Experimental features has their limitations and might cause hard to found root cause compilation issues.

For experimental features there might be many limitations which may end up with compilation issues without meaningful error message.
Time lost on identifying such issues is significantly higher then potential benefits.

IntelliJ already provides simple way to generate repetitive code: [Generate Code]https://www.jetbrains.com/help/idea/generating-code.html

Examples:
Usage of [Utility Class Example]https://projectlombok.org/features/experimental/UtilityClass

Code that works fine:
  
  ``` 
  return MapperUtil.getPOSActionCode(posActionCode);
   ```

Code that ends up in compilation issue:<br>
  ```
   import static com.sabre.wolverine.payment.wrapper.mapper.MapperUtil.getPOSActionCode;
   return getPOSActionCode(posActionCode);
  ```

Using static import might cause lombok plugin to fail on unrelated classes:

```
08:57:48    required: no arguments
08:57:48    found:    String
08:57:48    reason: actual and formal argument lists differ in length
08:57:48  [mvn-builder-payment-wrapper] [ERROR] /home/sabre/.cache/bazel/_bazel_sabre/12c7f27ee0056f4585fb0f099d3e104b/execroot/sabre2/wolverine/order/payment_wrapper/service/src/main/java/com/sabre/wolverine/payment/wrapper/utils/ActionType.java:[10,22] error: constructor ActionType in enum ActionType cannot be applied to given types;
08:57:48    required: no arguments
08:57:48    found:    String
08:57:48    reason: actual and formal argument lists differ in length
08:57:48  [mvn-builder-payment-wrapper] [ERROR] /home/sabre/.cache/bazel/_bazel_sabre/12c7f27ee0056f4585fb0f099d3e104b/execroot/sabre
```
