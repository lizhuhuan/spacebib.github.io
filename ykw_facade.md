# Facade 使用和测试

## 自定义 Facade

首先创建一个需要绑定到服务容器的 OrderConfirmationNumberGenerator 类:

```php
class OrderConfirmationNumberGenerator
{
    public function generate()
    {
        //
    }
}

```

然后创建一个静态指向 OrderConfirmationNumberGenerator 类的门面类 OrderConfirmationNumber:

```php
namespace App\Facades;

use Illuminate\Support\Facades\Facade;
use App\OrderConfirmationNumberGenerator;

class OrderConfirmationNumber extends Facade
{
    protected static function getFacadeAccessor()
    {
        return OrderConfirmationNumberGenerator::class;
    }
}
```

接下来我们要在服务提供者中绑定 OrderConfirmationNumberGenerator 类到服务容器:

```php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\OrderConfirmationNumberGenerator;

class AppServiceProvider extends ServiceProvider
{

    /**
     * Register the application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->bind(OrderConfirmationNumberGenerator::class, function () {
            return new OrderConfirmationNumberGenerator();
        });
    }
}
```

## Facade 测试

与传统静态方法调用不同的是，facades 也可以被模拟。相较传统的静态方法而言，它具有很大的优势，即便你使用依赖注入，可测试性不逊半分。在测试中，你可能想在控制器中模拟对 Laravel Facade 的调用。我们可以通过 shouldReceive 方法来模拟 Cache Facade，此函数会返回一个 Mockery 实例。由于 Facade 的调用实际是由 Laravel 的 服务容器 管理的，所以 Facade 能比传统的静态类表现出更好的可测试性。下面，让我们模拟一下 OrderConfirmationNumber Facade 的 generate 方法:

```php
class OrdersControllerTest extends TestCase
{
    public function testConfirmeOrder()
    {
        //...
        OrderConfirmationNumber::shouldReceive('generate')
                    ->once()
                    ->andReturn('TESTCODE');
    }
}
```
