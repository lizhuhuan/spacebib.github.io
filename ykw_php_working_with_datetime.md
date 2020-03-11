# Working with Datetime

## 本地时间

> 之所以有时区的概念是因为住在地球上不同地方的人看到太阳升起的时间是不一样的。我们假设北京人民在早上 8:00 看到了太阳刚刚升起，而此刻欧洲人民还在夜里，他们还需要再过 7 个小时才能看到太阳升起，所以，此刻欧洲人民的手表上显示的是凌晨 1:00。如果你强迫他们用北京时间那他们每天看到日出的时间就是下午 3 点。
> 也就是说，东 8 区的北京人民的手表显示的 8:00 和东 1 区欧洲人民手表显示的 1:00 是相同的时刻：
> "2014-10-14 08:00 +8:00" = "2014-10-14 01:00 +1:00"

这就是本地时间的概念。

只有在显示给用户的时候, 才转换为本地时间。

## 总是保存 UTC 的时间

## 尽可能使用 DateTimeImmutable

```php
function formatNextMondayFromNow( DateTime $dt )
{
  return $dt->modify('next monday')->format( 'Y-m-d' );
}

$d = new DateTime();
echo formatNextMondayFromNow( $d ), "\n";
echo $d->format( 'Y-m-d' ), "\n";
```

 上面👆的打印结果:

```
2020-03-02
2020-03-02
```

因为 add() 方法不仅返回修改后的 DateTime 对象，而且还更改了调用它的 DateTime 对象。这完全是出乎意料的，当你利用一个 DateTime 对象计算时，它本身也被改变了。

而 DateTimeImmutable 不会修改实例本身，而仅返回修改后的对象。