# What happens when multiple beans of the same type exist

Say you have:
```java
public interface PaymentService { void pay(); }

@Service
public class CardPaymentService implements PaymentService { ... }

@Service
public class UpiPaymentService implements PaymentService { ... }
```
and:
```java
@Service
public class OrderService {
    public OrderService(PaymentService paymentService) { ... }
}
```

