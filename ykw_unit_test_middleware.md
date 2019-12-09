# 测试 laravel 的中间件

中间件的逻辑一般都是简单却特别重要的。我们可以利用 unit test, 在
没有构建完整的功能时就确保中间件能按预期工作。

## 测试修改 request

假设我们有一个允许用户创建帖子的应用程序。帖子有标题和正文。在创建标题时，某些用户可能选择大写标题，而其他用户可能希望以小写形式写标题。为了统一起见，我们希望在将帖子保存到数据库之前对所有标题进行大小写区分。

```php
class TitlecaseMiddlewareTest extends TestCase
{
    /** @test */
    public function titles_are_set_to_titlecase()
    {
        $request = new Request;

        $request->merge([
            'title' => 'Title is in mixed CASE'
        ]);

        $middleware = new TitlecaseMiddleware;

        $middleware->handle($request, function ($req) {
            $this->assertEquals('Title is in Mixed Case', $req->title);
        });
    }
}
```

## 测试重定向不合法请求

Promoter 必须绑定 stripe 账户来收款，若没有绑定就重定向到绑定 stripe 的页面。

```php
class ForceStripAccountTest extends TestCase
{
    /** @test */
    public function users_without_a_stripe_account_are_forced_to_connect_with_stripe()
    {
        $this->be(facotry(User:class)->create([
            'stripe_account_id' => null,
        ]));
        $middleware = new ForeceStripeAccount;

        $response = $middleware->handle(new Request, function ($request) {
            $this->fail('Next middleware was called when it should not have been.');
        });

        $this->assertInstanceOf(RedirectResponse::class, $response);
        $this->assertEuqals(route('backstage.stripe-connect.authorize'), $response->getTargetUrl())
    }
}
```
