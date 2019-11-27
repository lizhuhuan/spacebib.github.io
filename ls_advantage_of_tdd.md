## 为什么要用TDD

用TDD的方法可以使代码干净，测试覆盖率高，软件做集成测试的时候一般问题会比较少。

当你修改人家的代码，看到有失败的测试用例,说明你改错东西，看到所有的测试用例都过了的话，就会有一种安全感

## 好处

+ 测试代码都是从客户需求出发的，不是从实现出发的。测试更关注于对外部的接口。
+ 软件的需求都被测试代码描叙得很清楚，可以减少很多不必要的文档（有些时候写文档时间比开发时间多多了, 需要一个专门写文档的，而且用的机会很少。）
+ 每次都是很小的步骤，这样你就很集中注意解决一个问题。葛优说的：步子迈大了容易扯着蛋，步子大想的就多，容易忽视些东西， 容易出错。小而简单就是美。
+ 可以优化设计。如果有做过测试驱动开发的会发现，为了更好的，更容易的做单元测试。它逼着你面向接口编程和使用一些设计模式，自然设计就灵活了，耦合性也低