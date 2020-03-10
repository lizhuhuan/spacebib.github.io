
## 正确逻辑

- 当前时间与event结束时间进行比较,做倒计时显示.倒计时未结束,可以添加result
- 当前时间在event的开始与结束时间之间,则可以添加result,否则不能添加


## 问题

倒计时还未结束,result不能添加


## 解决问题思路

1.服务器返回时间为NSString, 已转换为UTC
2.通过 NSDateFormatter, 把NSString转为NSDate
3. 把当前时间与服务器返回的时间,统一到服务器返回的时区
4.通过 NSComparisonResult result = [date1 compare:date2]比较两个日期的大小

## 涉及知识点

1.NSDate中存的是 UTC(时间标准时间)
2. NSDateFormatter
3.[NSTimeZone timeZoneForSecondsFromGMT:0]
4.[NSTimeZone timeZoneWithName:serverTimeZone]
