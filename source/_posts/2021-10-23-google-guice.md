---
title: Google Guice
date: 2021-10-23 11:53:19
tags: [Java, Guice]
categories: tech
---

除了Spring，你还知道其它依赖注入框架吗？
今天我看到了 Google Guice。其实早在2009年它就在Google I/O大会上被介绍过，我现在才知道。
是时候放下Spring了，了解一些其它"主流"框架了。原因是：Spring 是强大的，但你真的需要那么强大的它吗？

先翻译一下[Motivation](https://github.com/google/guice/wiki/Motivation)这个文章，来感受一下Google Guice

<!-- more -->

## 动机
将所有内容连接在一起是应用程序开发的乏味部分。 有几种方法可以将数据、服务和表示类相互连接起来。 为了对比这些方法，我们将为披萨订购网站编写计费代码：
```java
public interface BillingService {

  /**
   * Attempts to charge the order to the credit card. Both successful and
   * failed transactions will be recorded.
   *
   * @return a receipt of the transaction. If the charge was successful, the
   *      receipt will be successful. Otherwise, the receipt will contain a
   *      decline note describing why the charge failed.
   */
  Receipt chargeOrder(PizzaOrder order, CreditCard creditCard);
}
```
随着实现，我们将为我们的代码编写单元测试。 在测试中，我们需要一个 FakeCreditCardProcessor 来避免向真正的信用卡收费！

## 直接构造函数调用
当我们刚刚更新信用卡处理器和交易记录器时，代码如下所示：
```java
public class RealBillingService implements BillingService {
  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    CreditCardProcessor processor = new PaypalCreditCardProcessor();
    TransactionLog transactionLog = new DatabaseTransactionLog();

    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```
这段代码给模块化和可测试性带来了问题。 对真实信用卡处理器的直接编译时依赖意味着测试代码将收取信用卡费用！ 测试拒绝收费或服务不可用时会发生什么也很尴尬。

## 工厂模式
工厂类将客户端和实现类解耦。 一个简单的工厂使用静态方法来获取和设置接口的模拟实现。 工厂是用一些样板代码实现的：
```java
public class CreditCardProcessorFactory {

  private static CreditCardProcessor instance;

  public static void setInstance(CreditCardProcessor processor) {
    instance = processor;
  }

  public static CreditCardProcessor getInstance() {
    if (instance == null) {
      return new SquareCreditCardProcessor();
    }

    return instance;
  }
}
```

在我们的客户端代码中，我们用工厂查找替换`new`调用：

```java
public class RealBillingService implements BillingService {
  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    CreditCardProcessor processor = CreditCardProcessorFactory.getInstance();
    TransactionLog transactionLog = TransactionLogFactory.getInstance();

    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

工厂模式可以编写正确的单元测试：

```java
public class RealBillingServiceTest extends TestCase {

  private final PizzaOrder order = new PizzaOrder(100);
  private final CreditCard creditCard = new CreditCard("1234", 11, 2010);

  private final InMemoryTransactionLog transactionLog = new InMemoryTransactionLog();
  private final FakeCreditCardProcessor processor = new FakeCreditCardProcessor();

  @Override public void setUp() {
    TransactionLogFactory.setInstance(transactionLog);
    CreditCardProcessorFactory.setInstance(processor);
  }

  @Override public void tearDown() {
    TransactionLogFactory.setInstance(null);
    CreditCardProcessorFactory.setInstance(null);
  }

  public void testSuccessfulCharge() {
    RealBillingService billingService = new RealBillingService();
    Receipt receipt = billingService.chargeOrder(order, creditCard);

    assertTrue(receipt.hasSuccessfulCharge());
    assertEquals(100, receipt.getAmountOfCharge());
    assertEquals(creditCard, processor.getCardOfOnlyCharge());
    assertEquals(100, processor.getAmountOfOnlyCharge());
    assertTrue(transactionLog.wasSuccessLogged());
  }
}
```

这段代码很笨拙。 一个全局变量保存模拟实现，所以我们需要小心设置和拆除它。 如果tearDown 失败，全局变量将继续指向我们的测试实例。 这可能会导致其他测试出现问题。 它还阻止我们并行运行多个测试。

但最大的问题是依赖项隐藏在代码中。 如果我们添加对 CreditCardFraudTracker 的依赖，我们必须重新运行测试以找出哪些会中断。 如果我们忘记为生产服务初始化工厂，我们在尝试收费之前不会发现。 随着应用程序的增长，保姆工厂对生产力的消耗越来越大。

质量问题将被 QA 或验收测试发现。 这可能就足够了，但我们当然可以做得更好。

## 依赖注入
和工厂一样，依赖注入只是一种设计模式。 核心原则是将行为与依赖解析分开。 在我们的示例中，RealBillingService 不负责查找 TransactionLog 和 CreditCardProcessor。 相反，它们作为构造函数参数传入：

```java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  public RealBillingService(CreditCardProcessor processor,
      TransactionLog transactionLog) {
    this.processor = processor;
    this.transactionLog = transactionLog;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

我们不需要任何工厂，我们可以通过删除 setUp 和 tearDown 样板来简化测试用例：

```java
public class RealBillingServiceTest extends TestCase {

  private final PizzaOrder order = new PizzaOrder(100);
  private final CreditCard creditCard = new CreditCard("1234", 11, 2010);

  private final InMemoryTransactionLog transactionLog = new InMemoryTransactionLog();
  private final FakeCreditCardProcessor processor = new FakeCreditCardProcessor();

  public void testSuccessfulCharge() {
    RealBillingService billingService
        = new RealBillingService(processor, transactionLog);
    Receipt receipt = billingService.chargeOrder(order, creditCard);

    assertTrue(receipt.hasSuccessfulCharge());
    assertEquals(100, receipt.getAmountOfCharge());
    assertEquals(creditCard, processor.getCardOfOnlyCharge());
    assertEquals(100, processor.getAmountOfOnlyCharge());
    assertTrue(transactionLog.wasSuccessLogged());
  }
}
```

现在，每当我们添加或删除依赖项时，编译器都会提醒我们需要修复哪些测试。 依赖项在 API 签名中公开。

不幸的是，现在 BillingService 的客户端需要查找其依赖项。 我们可以通过再次应用该模式来修复其中的一些问题！ 依赖它的类可以在其构造函数中接受 BillingService。 对于顶级类，有一个框架很有用。 否则，当您需要使用服务时，您将需要递归构建依赖项：

```java
public static void main(String[] args) {
    CreditCardProcessor processor = new PaypalCreditCardProcessor();
    TransactionLog transactionLog = new DatabaseTransactionLog();
    BillingService billingService
        = new RealBillingService(processor, transactionLog);
    ...
  }
```

## Guice 依赖注入
依赖注入模式导致代码模块化和可测试，Guice 使其易于编写。 要在我们的计费示例中使用 Guice，我们首先需要告诉它如何将我们的接口映射到它们的实现。 此配置在 Guice 模块中完成，该模块是任何实现 Module 接口的 Java 类：

```java
public class BillingModule extends AbstractModule {
  @Override
  protected void configure() {
    bind(TransactionLog.class).to(DatabaseTransactionLog.class);
    bind(CreditCardProcessor.class).to(PaypalCreditCardProcessor.class);
    bind(BillingService.class).to(RealBillingService.class);
  }
}
```

我们将@Inject 添加到 RealBillingService 的构造函数中，指示 Guice 使用它。 Guice 将检查带注释的构造函数，并查找每个参数的值。

```java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  @Inject
  public RealBillingService(CreditCardProcessor processor,
      TransactionLog transactionLog) {
    this.processor = processor;
    this.transactionLog = transactionLog;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

最后，我们可以把它们放在一起。 Injector 可用于获取任何绑定类的实例。

```java
public static void main(String[] args) {
    Injector injector = Guice.createInjector(new BillingModule());
    BillingService billingService = injector.getInstance(BillingService.class);
    ...
  }
```

## 总结
* Guice 是一个轻量级框架，实现依赖注入。（稍后深入其它特性）
* AbstractModule configure中的绑定，相当于Spring @component 等相关注解，Spring scan时即建立绑定关系。至于Guice何时对象初始化，我需要再看看。
* @Inject 相当于 Spring 中的@Autowired，自动注入对象实例
* 最后一个UnitTest代码没有再提出，但可以想像就是之前的UniTest即可，在具体测试单元通过构造方法传参即可。或将整个UnitTest的AbstractModule配置修改，绑定对应的mock class。

初步感觉这个框架还是容易理解的，稍后深入后续内容。