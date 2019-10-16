# 测试替身

在使用 TDD 开发时，经常会遇到需要被测对象需要依赖其他子系统的情况，但是你希望将测试代码跟依赖项隔离，以保证测试代码仅仅针对当前被测对象或方法展开，这时候你需要的是测试替身。测试替身细分可以分为：

- Dummy Object
- Fake Object
- Test Stub
- Test Spy
- Mock Object

## Dummy Object

如果在测试替身上没有调用任何方法，也没有访问任何字段，则 Dummy Object 是首选。
Dummy Object 通常用来满足类型提示或者空检查。在没有 Type Hint 的情况下可以直接
用 null 或 字符串代替；在指定类型时可以用一个简单的子类或实现了接口的简单类代替。

## Test Stub

它提供了一些简单的实现，实现我们在测试中要用到的方法，指定期望的行为（如返回期望的值）。

假设我们的一个模块需要与第三方交互, 在测试需要用到这个模块时, 我们显然不太可能与第三方去交互,而是提供一个带有 一系列 方法的假对象（Stub），然后我们需要将这个假对象注入到产品代码中，从而在测试产品代码时替换真实的模块对象，执行假对象的行为。

```php
class FaksePaymentGateway
{
    public function __construct()
    {
        $this->charges = collect();
    }

    public function getValidTestToken()
    {
        return 'valid-token';
    }

    public function charge($amount, $token)
    {
        $this->charges[] = $amount;
    }

    public function totalCharges()
    {
        return $this->charges->sum();
    }
}

$paymentGateway = new FakePaymentGateWay();
// @Laravel
// 注入 $paymentGateway, 这样在测试中在需要 PaymentGateway 时就会使用 $paymentGateway
$this->app->instance(PaymentGateway::class, $paymentGateway);

```

## Test Spy，Mock Object

对于复杂的测试替身，我们可以使用一些 Mock 框架。在 PHP 的生态里，[Mockery](https://github.com/mockery/mockery) 是最主流的。Mockery 允许使用灵活的 API 设置期望的方法调用，返回值以及调用次数。
