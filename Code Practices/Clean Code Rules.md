**Summary of "Clean Code" by Robert C. Martin**
Rules listed below are extension of rules available under[https://gist.github.com/wojteklu/73c6914cc446146b8b533c0988cf8d29]

**Small Methods, Business oriented naming**
**Description -**

1. methods should be small<br>
2. methods should have meaningful names<br>
3. methods should have business names instead of technical names (if possible, technical names are allowed for purely technical code, but not in business code)<br>
4. name should clearly explain intention

**Avoid**
```
 private void updateTokenizedCardId(PaymentFunction.Builder paymentFunction,PaymentProcessingPaymentCardType paymentProcessingPaymentCardType) {
    String tokenizedCardId =
        Optional.ofNullable(paymentProcessingPaymentCardType.getTokenizedCardID())
            .orElse(StringUtils.EMPTY);
    if (StringUtils.isEmpty(tokenizedCardId)) {
      String cardNumber =
          Optional.ofNullable(paymentProcessingPaymentCardType.getCardNumber())
              .orElse(StringUtils.EMPTY);
      TokenizeRequest tokenizeRequest =
          TokenizeRequest.newBuilder()
              .setCardId(cardNumber)
              .setFormOfPayment(FORM_OF_PAYMENT)
              .build();
      TokenizeResponse response = paymentWrapperClient.tokenize(tokenizeRequest);
      tokenizedCardId = response.getTokenizedCardId();
    }
    final String tokenizedCardIdFinal = tokenizedCardId;
    Optional.of(paymentFunction)
        .map(PaymentFunction.Builder::getPaymentProcessingSummaryBuilder)
        .map(PaymentProcessingSummary.Builder::getPaymentMethodDetailsBuilder)
        .map(PaymentMethodDetails.Builder::getPaymentCardBuilder)
        .ifPresent(paymentCard -> paymentCard.setTokenizedCardId(tokenizedCardIdFinal));
  }
```
**Use**
```
  private void updateTokenizedCardId(PaymentFunction.Builder paymentFunction,PaymentProcessingPaymentCardType paymentProcessingPaymentCardType) {
    String tokenizedCardId = extractTokenizedCardId(paymentProcessingPaymentCardType);
    if (cardRequiresTokenization(tokenizedCardId)) {
      tokenizedCardId = tokenizeCreditCard(paymentProcessingPaymentCardType);
    }
    updatePaymentFunctionTokenizedCardId(paymentFunction, tokenizedCardId);
  }
```
