# 展示组件 & 容器组件
原文链接:[Presentional and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
当我写React app的时候我发现一个非常有用的建艺模型. 如果你在使用React期间,也许已经发现了[这篇文章](http://facebook.github.io/react/blog/2015/03/19/building-the-facebook-news-feed-with-relay.html),但是我想补充几点.

如果你将你的组件分成两类会跟容易重用和理解其中的原理.我称这两类组件为Container组件和Presentional组件.但是我也听过 *Fat & Skinny*,*Smart & Dumb*,*Stateful & Pure*,*Screens & Components*,etc. 他们不完全相同,但是核心思想是一样的.

#### 展示组件:
1. 展示组件只关心组件的样子.(这里的thing,我认为是组件)
2. 展示组件可以包含展示和容器子组件这两者在里面.并且一般都拥有一些DOM标签和各自的样式.
3. 经常允许使用*this.props.children*来包含组件.
4. 在嵌套app都是独立的,没有依赖的.
5. 不会指定data如何被加载或者如何变化.
6. 只能经由*props*接收data和回调函数.
7. 极少拥有自己的state(如果有,也是UI state,而不是date)
8. 除非需要state,生命周期钩子或者性能优化,展示组件都应该使用函数式编程.
9. 栗子: 页面,工具栏,历史,用户信息,列表.

#### 容器组件:
1. 只关心组件如何工作.
2. 也可以包含展示和容器组件作为子组件.但是一般不会拥有DOM标签,除了包含该容器组件的`<div>`.也不会拥有自己的样式.
3. 提供data和行为给展示组件和容器组件.
4. 调用Flux的action,提供给展示组件作为回调函数.
5. 往往是有状态的,因为他们倾向于充当数据源.
6. 一般由高阶组件(如React-Redux的*connect()*)生成.
7. 栗子:用户页面,用户工具栏,历史容器,用户跟踪列表.

我把这两种组件放在不同的文件夹中,可以更清楚的区分他们.

### 这种方法的好处
1. 更好的分离关注点.你应该通过编写组件这种方式来更好的了解你的app和UI
2. 更好的重用性,你可以在完全不同的数据源中使用同样的展示组件,并且可以将展示组件变成另外的容器组件,在以后重用他们.
3. 组件的本质就是你的APP的*调色板*,你可以将他们放在一个单独的页面中,让设计师调整他们所有的变化,而不用碰触app的逻辑.You can run screenshot regression tests on that page.(这句不太懂什么叫'截屏回归测试')
4. 迫使你提取`布局组件` 例如,工具栏,页面,etc.并且使用`this.props.children`来代替重复的组件在一些容器组件中.

**记住,组件不必生成DOM,他们只需要在UI关注点提供组合和边界(Remember, components don’t have to emit DOM. They only need to provide composition boundaries between UI concerns.其实没读懂....)**
