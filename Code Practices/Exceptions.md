**Log exception or rethrow it - NEVER do both**

**Description -** In try catch statement exceptions might be handled(with optional logging) or rethrown.
When exception is rethrown it will be handled in other place when error message would be logged.

**Use**
```
try {
  return Event.OrderSyncEvent.parseFrom(payload);
} catch (InvalidProtocolBufferException e) {
  throw new OrderSyncProcessingException("Filed to parse binary for OrderSyncEvent", e);
}
```
Avoid

```
try {
  return Event.OrderSyncEvent.parseFrom(payload);
} catch (InvalidProtocolBufferException e) {
  log.error("Filed to parse binary for OrderSyncEvent", e);
  throw new RuntimeException(e);
}
```
See : [More Details] https://howtodoinjava.com/best-practices/java-exception-handling-best-practices/#5
