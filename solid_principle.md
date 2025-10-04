These are five core principles of object-oriented design that help make software more maintainable, flexible, and scalable. Each letter in SOLID stands for a principle:

## S – Single Responsibility Principle (SRP)

**A class should have only one reason to change, meaning it should do one thing only.**

- Why it matters: If a class does too many things, changing one feature may break another.
- Examples:

```java
class Account {
    private double balance;
    void deposit(double amount) { balance += amount; }
    void withdraw(double amount) { balance -= amount; }
    double getBalance() { return balance; }
}

class TransactionLogger {
    void logTransaction(String message) {
        System.out.println("Transaction log: " + message);
    }
}
```
- Explanation: Account handles account operations, TransactionLogger handles logging.
- Bad practice: Don’t put logging inside Account.

## O – Open/Closed Principle (OCP)

**Software should be open for extension but closed for modification.**

- Example:

```java
interface NotificationService {
    void notifyCustomer(String message);
}

class EmailNotification implements NotificationService {
    public void notifyCustomer(String message) {
        System.out.println("Email: " + message);
    }
}

class SMSNotification implements NotificationService {
    public void notifyCustomer(String message) {
        System.out.println("SMS: " + message);
    }
}

class AccountService {
    private NotificationService notificationService;

    AccountService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    void deposit(Account account, double amount) {
        account.deposit(amount);
        notificationService.notifyCustomer("Deposited: " + amount);
    }
}
```
- Explanation: If you want to add a new notification type (like Push Notification), you don’t need to modify AccountService. Just extend NotificationService.

```java
class PushNotification implements NotificationService {
    @Override
    public void notifyCustomer(String message) {
        System.out.println("Push Notification: " + message);
    }
}
```
- Usage Example:

```java
public class Main {
    public static void main(String[] args) {
        Account account = new Account();
        
        // Use Push Notification without changing AccountService
        NotificationService pushService = new PushNotification();
        AccountService accountService = new AccountService(pushService);

        accountService.deposit(account, 500);
    }
}

```

```yml
Push Notification: Deposited: 500.0
```

## L – Liskov Substitution Principle (LSP)

**Subtypes should be replaceable for their base types.**

- Example:

```java
abstract class Account {
    abstract void withdraw(double amount);
}

class SavingsAccount extends Account {
    void withdraw(double amount) { /* allow withdrawal */ }
}

class FixedDepositAccount extends Account {
    void withdraw(double amount) {
        throw new UnsupportedOperationException("Cannot withdraw from Fixed Deposit before maturity");
    }
}
```

- Problem: FixedDepositAccount violates LSP because it cannot behave like Account.
- Solution: Introduce separate hierarchy or interface:

```java
interface Withdrawable { void withdraw(double amount); }
class SavingsAccount implements Withdrawable { ... }
```
### I – Interface Segregation Principle (ISP)

**Don’t force classes to implement methods they don’t need.**

```java
interface AccountOperations {
    void deposit(double amount);
    void withdraw(double amount);
    void calculateInterest();
}

class SavingsAccount implements AccountOperations {
    public void deposit(double amount) { ... }
    public void withdraw(double amount) { ... }
    public void calculateInterest() { ... }
}

class FixedDepositAccount implements AccountOperations {
    public void deposit(double amount) { ... }
    public void withdraw(double amount) { throw new UnsupportedOperationException(); }
    public void calculateInterest() { ... }
}
```
- Problem: FixedDepositAccount doesn’t need withdraw.
- Better: Split interfaces:

```java
interface Depositable { void deposit(double amount); }
interface Withdrawable { void withdraw(double amount); }
interface InterestCalculable { void calculateInterest(); }

class SavingsAccount implements Depositable, Withdrawable, InterestCalculable { ... }
class FixedDepositAccount implements Depositable, InterestCalculable { ... }
```
## D – Dependency Inversion Principle (DIP)

**High-level modules should depend on abstractions, not concrete classes.**

- Example:

```java
interface PaymentGateway {
    void pay(double amount);
}

class StripePayment implements PaymentGateway {
    public void pay(double amount) { System.out.println("Paid via Stripe: " + amount); }
}

class BankService {
    private PaymentGateway gateway;

    BankService(PaymentGateway gateway) { this.gateway = gateway; }

    void processPayment(double amount) {
        gateway.pay(amount);
    }
}
```

- Explanation: BankService doesn’t depend on StripePayment directly. You can inject any payment gateway (PayPal, Razorpay, etc.) without changing BankService.

## Summary Table in Banking Context:

| Principle | Banking Example |
|-----------|----------------|
| SRP | `Account` handles balance, `TransactionLogger` handles logging |
| OCP | `NotificationService` allows adding new notification types without changing `AccountService` |
| LSP | Separate `Withdrawable` interface instead of forcing `FixedDepositAccount` to support withdrawal |
| ISP | Split `Depositable`, `Withdrawable`, `InterestCalculable` instead of one big interface |
| DIP | `BankService` depends on `PaymentGateway` interface, not a concrete payment class |
