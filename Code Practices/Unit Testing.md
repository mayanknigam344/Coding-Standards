**1. Assertions should fail with clear, readable message**

**Description** - When writing assertions, make sure that assertion is written in a way, that will produce clear message when failing on Jenkins.

For example, let's say that we want to test if given field value matches regexp.

**Avoid**
```
@Test
void shouldMaskPCIData() {
  //given
  //...
   
  //when
  //...
   
  //then
  assertTrue(fieldValue.matches(MASKED_FIELD_VALUE));
}
```
The problem with the above code is that when it will fail on Jenkins, it will produce message like below:
```
org.opentest4j.AssertionFailedError: 
Expected :true
Actual   :false
```
The challenge is that when trying to understand test failure, we need to now debug the code locally, run the test.

Following approach is much better, as it will actually show what was produced, and what is expected:

**Use**
```
@Test
void shouldMaskPCIData() {
  //given
  //...
 
  //when
  //...
 
  //then
  assertThat(fieldValue, matchesPattern(MASKED_FIELD_VALUE));
}
```
When above test fails, it will produce meaningful message and will show actual data produced, and expected data:
```
Expected: a string matching the pattern '^\**$' but: was "4485 0653 3119 6823"
Expected :a string matching the pattern '^\**$'
Actual   :"4485 0653 3119 6823"
```
Ex- <br>
```
assertThat(passenger.getProfileConsentInd())
.as("Check ProfileConsentIndicator value for Passenger with id %s", passenger.getId())
.isTrue();
```
Useful Links <br>
[1]https://joel-costigliola.github.io/assertj/assertj-core-features-highlight.html<br>
[2]https://phauer.com/2019/modern-best-practices-testing-java/#avoid-asserttrue-and-assertfalse<br>
[3]https://phauer.com/2019/modern-best-practices-testing-java/#use-assertj
