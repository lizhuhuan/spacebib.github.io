# 用整数生成绝对不重复的唯一 ID 类库——Hashids

在项目中，暴露给用户真实的项目 ID，很有可能被恶意采集，很有可能被猜到目前有多少量。或者说我们想要生成较短的，易记的唯一标识符都可以是使用 Hashids 解决这个问题。

Hashids 是一个能利用整数生成出短小、唯一、非连续标识符的类库。但是该库并不是一个加密库，所以不建议用来加密敏感数据。
Hashids 支持通过生成出来的标识符进行解码为原数字，还支持加盐加密，不会因为大家都用这个类库就被猜到真实 ID。

## 使用

```php
$hashids = new Hashids\Hashids('this is my salt');
$id = $hashids->encode(1);
$numbers = $hashids->decode($id);
```

### 指定生成标识符的长度

```php
$hashids = new Hashids\Hashids('this is my salt'，6);
$id = $hashids->encode(1); // VolejR
```

### 使用自定义字母

```php
use Hashids\Hashids;

$hashids = new Hashids('', 0, 'abcdefghijklmnopqrstuvwxyz'); // all lowercase
$hashids->encode(1, 2, 3); // mdfphx
```
