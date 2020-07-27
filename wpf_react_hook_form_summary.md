# React-Hook-Form的使用总结

> Performant, flexible and extensible forms with easy-to-use validation.  --react-hook-form

### Form 的使用

为什么需要使用 form，或者说使用 form 哪些收益？

以 input 为例，你可以使用多个 input 再配合一些 state 或者 redux 可以完成对用户输入的收集，也能完成对 API 的调用。但是当你处理一些数据的联动（不同选择对应不同选项），对验证(validation)，对错误(errors)，对触摸(touched)这些数据或者状态进行管理时，你会发现用 input 会一遍遍的做重复的工作，而且相当繁琐，所以这时候你需要 form 统一对这些 input 产生的各种状态进行管理，但是原生 html5 form 并不是为了管理这些状态。

> Web forms — Working with user data     --MDN

于是，产生了很多 form 库来帮助我们去管理状态和验证，例如 [Formik](https://formik.org/) 、[React-Hook-Form](https://react-hook-form.com/) 等等就是其中最受欢迎的一些库。

### Form 的对比

在 react 框架中，常见的 fom 库对比情况如下：

|                | Formik | React-Hook-Form | Redux-Form | React-Jsonschema-Form | React-Final -Form |
| :------------: | ------ | --------------- | ---------- | --------------------- | ----------------- |
|      性能      | 低     | 高              | 低         | 低                    | 高                |
|   教程友好度   | 高     | 高              | 一般       | 一般                  | 一般              |
| 开发者活跃程度 | 高     | 高              | 一般       | 一般                  | 高                |
|    接入成本    | 低     | 一般            | 低         | 高（与其他框架耦合）  | 低                |
|      备注      |        | 需要引入ref     |            | 需要维护json schema   | 低                |

注：教程友好度指文档完善程度，相关问题是否容易被搜索到，issue 处理。

那为什么选择 react-hook-form，首先可以看看相关文章的对比，[react-hook-form vs formik-form](https://blog.bitsrc.io/react-hook-form-vs-formik-form-builder-library-for-react-23ed559fdae#:~:text=Just%20like%20Formik%2C%20React%20Hook,rendering%20caused%20by%20user%20inputs.) . formik 的多年积累是非常成熟的，而且在 react form 库的各个数据都是排名第一，但是性能问题非常严重，性能问题作者也知道，在 GitHub issue上依旧可以见，但是未能解决。主要问题如下：

```html

    <form>
      <input type="text" name="name" />
      <input type="password" name="password" />
    </form>

```

这是一个常见的简单的 form ， 当你现在这么使用，formik 和 react-hook-form是没有太大区别的，但是当你需要验证 name 是否重复的时候，你会加上一个 nameValidation 这时候性能就出现了，你希望能在输入 name 去验证 name 是否唯一，你肯定不希望在输入 password 还去验证 name。由于formik是整体刷新，所以每次输入都可能在验证name，这浪费了很多性能，特别是输入多的form，会异常卡顿。所以全局form刷新的基本不考虑，那还有react-final-form，react-final-form的设计确实不错，但是学习起来有点困难，作者对文档的要求不高，不太方便探索。在当时的情况，我就选择了 react-hook-form 。

react-hook-form 也有许多缺点，例如和其他组件（例如 react-select）支持很不方便，需要不断的处理 ref ，API设计的不够成熟好用。但还是瑕不掩瑜，作者非常活跃，改进速度非常快，issues处理及时。教程也十分完善，这些优点可以弥补那些缺点了。

### React-Hook-Form 基本使用

form 的最重要的就是 user data, validation, form status。

- 首先是[基础教程](https://react-hook-form.com/get-started/) 需要全部过一遍，
- 然后codesanbox 也有[十分多的例子](https://github.com/react-hook-form/react-hook-form/tree/master/examples)

这样就差不多了，react-hook-form 入门是轻松愉快的。

需要注意的地方：

- react >= 16.8，因为是hook方式

- 必须为每个输入注册 ref
- validation 必须返回 string 的 message
- 假如发生表单联动，例如小于18岁，仍然会发生整体刷新

其他还有一些数据上的问题，比较难以描述，遇到时搜索就能解决。总体来说这个库在作者高频率的维护下，bug并不多。

## 最后

form 类似富文本，虽然没有那么多坑，但是也不少，经过对比没有最好的form library，只有相对还行的，目前的form library都做的还不太够，离像react这样正在相对完善成熟的性能高，使用友好的库目前仍然是没有的。选择react-hook-form属于目前来看发展的非常不错的库，代表现在的进步，但是未来还需要更好的form library出现来拯救 react form。
