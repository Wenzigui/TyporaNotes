---

---

[toc]

# 概述

Vue：渐进式JavaScript框架

Vue使用进阶：

声明式渲染 -> 组件系统 -> 客户端路由 -> 集中式状态管理 -> 项目构建

# 基本使用 

## 传统开发模式对比

![](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/20200420211405.png)



> JQ对JS封装，提高了易用性



![](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/20200420212454.png)



- el表示元素的挂载位置（值可以是CSS选择器或者DOM元素）
- data表示模型数据
- {{xxx}}表示插值表达式，可以将数据填充到HTML标签，同时支持基本的计算操作

![](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/20200420212401.png)

> 编译过程

## 模板语法

![](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/20200420212935.png)

前端渲染的方式：

- 原生JS拼接字符串（代码可读性差，维护困难）
![](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/20200420212730.png)

- 使用前端模板引擎（代码可读性好，但是没有专门提供事件机制）
![](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/20200420212959.png)

- 使用Vue的模板语法

### 指令

- 指令的本质就是自定义属性
- 指令格式：v-xxx

#### v-cloak

解决插值表达式的“闪动”问题

> “闪动”原因：先展示插值表达式，然后填充数据，所以频繁刷新页面的时候，可以看到插值表达式

![](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/20200420214059.png)

> 原理：通过样式隐藏内容，然后再内存中进行值得替换，替换之后再显示最终结果

#### 数据绑定指令

- v-text：填充纯文本
- v-html：填充HTML片段（存在安全问题）
- v-pre：填充原始信息（跳过编译过程）

![](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519213720565.png)

#### 数据响应式

HTML5中的响应式：屏幕尺寸的变化导致样式变化

数据的响应式：数据变化导致页面内容的变化



![image-20200519212131839](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212131839.png)



> 数据驱动内容发现变化

v-once指令：只编译一次，显示内容之后不再具有响应式功能

### v-model 双向数据绑定

![image-20200519212205353](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212205353.png)

v-model指令：绑定属性值

MVVM设计思想

- Model
- View
- View-Model

![image-20200519212255537](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212255537.png)

> - 从视图到模型通过DOM监听器
> - 从模型到视图通过数据绑定

### 事件绑定

#### v-on指令

![image-20200519212327323](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212327323.png)

> this表示当前的vue实例

#### 事件修饰符

![image-20200519212346729](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212346729.png)

#### 按键修饰符

![image-20200519212359402](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212359402.png)

### v-bind 属性绑定

![image-20200519212416585](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212416585.png)

![image-20200519212429367](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212429367.png)

**v-model双向绑定的原理**

![image-20200519212443851](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212443851.png)

![image-20200519212454214](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212454214.png)

> 改变数据模型，影响视图；改变视图，影响数据模型

v-model = v-bind + v-on

### 样式绑定

![image-20200519212527618](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212545081.png)

- 对象绑定和数组绑定可以结合使用

### 分支循环结构

#### 分支结构

- v-if
- v-else
- v-else-if
- v-show

> - v-if控制元素是否渲染到页面
> - v-show控制元素是否显示（实际上已经渲染到页面上了）

#### 循环结构

- v-for

![image-20200519212545081](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212545081.png)

使用key提高性能

![image-20200519212606498](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212606498.png)

# 常用特性

## 表单操作

![image-20200519212616146](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212616146.png)

### 表单域修饰符

- number：转化为数值
- trim：去掉开始和结尾的空格
- lazy：将input事件切换为change事件（失去焦点的时候才触发更新）

![image-20200519212635642](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212635642.png)

![image-20200519212647338](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212647338.png)

## 自定义指令

> 在内置指令不满足需求的时候，可以自定义指令

### 语法规则

![image-20200519212708814](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212708814.png)

> 获取元素的焦点
> - inserted：钩子函数，被绑定元素插入父节点时调用（仅保证父节点存在，但不一定已被插入文档中）
> - el：指令所绑定的元素，可以用来直接操作DOM
>

### 使用自定义指令

![image-20200519212721003](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212721003.png)

![image-20200519212735614](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212735614.png)

### 带参数的自定义指令

![image-20200519212851759](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212851759.png)

![image-20200519212910338](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212910338.png)

### 局部指令

![image-20200519212931077](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519212931077.png)

> 局部指令只能在组件内使用

## 计算属性

### 为什么需要计算属性？

表达式的计算逻辑可能会比较复杂，使用计算属性可以使模板内容更加简洁

![image-20200519213028879](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519213028879.png)

### 用法

 ![image-20200519213039552](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519213116021.png)

### 计算属性和方法的区别

 - 计算属性是基于它们的依赖进行缓存
 - 方法不存在缓存

![image-20200519213116021](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519213116021.png)

> - reverseString只执行一次，msg不更新，则计算属性不会更新

## 侦听器

![image-20200519213410905](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519213410905.png)

> 适用于当数据变化时，执行异步操作或者开销较大的操作

![image-20200519213609618](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519213609618.png)

## 过滤器

### 作用

格式化数据，比如将字符串格式化为首字母大写等

![image-20200519213634399](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519213634399.png)



### 自定义过滤器

![image-20200519213720565](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/20200420214926.png)

### 使用过滤器

![image-20200519213740820](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519213752427.png)

> 支持级联操作

### 局部过滤器

![image-20200519213752427](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519213740820.png)

### 带参数的过滤器

![image-20200519213812204](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519213959966.png)

## 生命周期

### 主要阶段

1. 挂载（初始化相关属性
    1. beforeCreate
    2. created
    3. beforeMount
    4. mounted
2. 更新（元素或组件的变更操作）
    1. beforeUpdate
    2. updated
3. 销毁（销毁相关属性）
    1. beforeDestroy
    2. destroyed

![image-20200519213921886](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519213812204.png)

# 组件化开发

## 组件注册

### 全局组件注册

![image-20200519213959966](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519213921886.png)

> 组件支持重用，数据互相独立

注意事项：

- 组件中的data必须是个函数
- 组件模板的根元素必须只有一个
- 组件模板中的内容可以是模板字符串
- 组件命名方式
    - 驼峰式（ComponentName）
    - 短横线（component-name，推荐使用）

![image-20200519214047511](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214047511.png)

> 用反引号括住的就是模板字符串

![image-20200519214119309](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214119309.png)

> 通过驼峰式的命名去注册组件，在字符串模板可以依旧使用驼峰的方式使用组件，但是在普通的标签模板中，必须使用短横线的方式使用组件

### 局部组件注册

![image-20200519214131837](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214131837.png)

> - 在Vue实例中使用components属性
> - 全局组件的模板中不允许使用局部组件

## 组件间数据交互

### 父组件向子组件传值

![image-20200519214146125](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214146125.png)

> 组件内部通过props接受传递过来的值

![image-20200519214240929](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214240929.png)

> 第一种方式传递静态的值
> 第二种方式传递动态的值

注意事项：

- 在props中使用驼峰形式，模板中需要使用短横线的形式
- 字符串形式的模板中没有这个限制

![image-20200519214251479](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214251479.png)

### 子组件向父组件传值

> props传递数据原则，单向数据流：只允许父组件通过props给子组件传值

子组件通过自定义事件向父组件传递消息

![image-20200519214320528](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214320528.png)

> @enlarge-text是v-on指令简写

![image-20200519214401104](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214401104.png)

> $event是固定的，代表子组件传过来的值

### 非父子组件间传值

> 通过事件中心管理组件进行通信

![image-20200519214419038](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214419038.png)

1. 单独new一个Vue实例，由该实例扮演事件中心的角色

![image-20200519214802452](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214802452.png)

2. 监听、销毁事件

![image-20200519214813267](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214828178.png)

3. 触发事件

![image-20200519214828178](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519214813267.png)

```javascript
// 事件中心
var hub = new Vue();

Vue.component('test-tom', {
  data: function(){
    return {
      num: 0
    }
  },
  template: `
    <div>
      <div>TOM:{{num}}</div>
      <div>
        <button @click='handle'>点击</button>
      </div>
    </div>
  `,
  methods: {
    handle: function(){
      // 触发兄弟组件的事件
      hub.$emit('jerry-event', 2);
    }
  },
  mounted: function() {
    // 监听事件
    hub.$on('tom-event', (val) => {
      this.num += val;
    });
  }
});
Vue.component('test-jerry', {
  data: function(){
    return {
      num: 0
    }
  },
  template: `
    <div>
      <div>JERRY:{{num}}</div>
      <div>
        <button @click='handle'>点击</button>
      </div>
    </div>
  `,
  methods: {
    handle: function(){
      // 触发兄弟组件的事件
      hub.$emit('tom-event', 1);
    }
  },
  mounted: function() {
    // 监听事件
    hub.$on('jerry-event', (val) => {
      this.num += val;
    });
  }
});
```

## 组件插槽

### 作用

父组件向子组件传递模板内容

![image-20200519215050324](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519215050324.png)

> slot中可以设置默认内容，如果父组件没有传递模板内容，则显示默认内容

### 具名插槽

> 就是带名字的插槽

![image-20200519215106174](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519215106174.png)

### 作用域插槽

> 应用场景，父组件对子组件的内容进行加工处理

```javascript
Vue.component('fruit-list', {
  props: ['list'],
  template: `
    <div>
      <li :key='item.id' v-for='item in list'>
      // 通过插槽绑定属性，向父组件传递内容
        <slot :info='item'>{{item.name}}</slot>
      </li>
    </div>
  `
});

<fruit-list :list='list'>
  <template slot-scope='slotProps'>
  // 父组件通过作用域插槽访问子组件传递的内容，并根据内容进行加工
    <strong v-if='slotProps.info.id==3' class="current">{{slotProps.info.name}}</strong>
    <span v-else>{{slotProps.info.name}}</span>
  </template>
</fruit-list>
```

# 前后端交互

## axios

axios是一个基于Promise用于浏览器和node.js的http客户端



### 基本用法

![image-20200519215122483](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519215122483.png)



### 常用API



**Get请求**

![image-20200519223957530](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519223957530.png)



**Delete请求**

![image-20200519224116344](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519224116344.png)

>   跟get方式类似



**Post请求**

![image-20200519224312212](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519224605206.png)

>   默认传递json参数



![image-20200519224605206](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200519224312212.png)

>   通过URLSearchParams传递表单参数application/x-www-form-urlencoded



**Put请求**

>   跟Post请求类似



### 结果响应

![image-20200521155036483](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200521155036483.png)



### 全局配置

![image-20200521155556937](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200521155556937.png)



### 拦截器



#### 请求拦截器

![image-20200521155752228](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200521155752228.png)



![image-20200521160027303](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200521160027303.png)



#### 响应拦截器

![image-20200521160132758](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200521160132758.png)



![image-20200521160327012](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200521160327012.png)



# Vue Router

>   Vue的官方路由管理器



包含功能：

-   支持html5历史模式或者hash模式
-   支持嵌套路由
-   支持路由传参
-   支持编程式路由
-   支持命名路由



## 基本概念

![image-20200522213655694](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200522213655694.png)



## 基本使用



1.  引入相关的库文件

![image-20200522215144744](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200522215144744.png)

>   先引入Vue，再引入Vue Router



2.  添加路由链接

![image-20200522215131516](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200522215131516.png)



3.  添加路由填充位

![image-20200522215253687](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200522215253687.png)



4.  定义路由组件

![image-20200522215454049](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200522215454049.png)



5.  配置路由规则

![image-20200522215901460](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200522215901460.png)



6.  把路由挂载到Vue根实例

![image-20200522215943638](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200522215943638.png)



## 路由重定向

>   在用户访问地址A的时候，强制用户跳转到地址C，从而展示特定的组件页面



![image-20200524100109303](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524100109303.png)



## 嵌套路由



1.  在父组件的模板中配置子路由链接和路由占位符

![image-20200524100813255](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524100813255.png)



2.  创建路由组件

![image-20200524101218632](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524101218632.png)



3.  配置路由规则

![image-20200524101252767](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524101252767.png)



## 动态路由匹配

>   应用场景，通过路由参数进行路由匹配



1.  设置路由参数

![image-20200524102207561](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524102207561.png)



2.  获取路由参数

![image-20200524102229295](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524102229295.png)



## 路由组件传递参数

>   使用`$route`传参会导致与路由耦合度高，推荐使用props传参，可以跟路由解耦



1.  在路由规则中设置开启props传参

![image-20200524102758951](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524102758951.png)

>   如果是要传递对象类型的数据，可以如下图所示

![image-20200524103033633](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524103033633.png)

>   上图中的`id`属性无法在路由组件中获取
>   如果需要获取`id`属性，可以传递函数类型的props

![image-20200524103545891](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524103545891.png)



2.  在路由组件中使用props接受路由参数

![image-20200524103112710](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524103112710.png)



## 命名路由

>   就是给路由规则起了个别名



![image-20200524104556308](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524104556308.png)



![image-20200524104701463](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524104701463.png)



## 编程式导航



页面导航的方式：

-   声明式导航
    通过点击链接实现导航方式
    比如点击网页中的`<a>`标签，或者使用Vue中的`<router-link>`
-   编程式导航
    通过调用JS形式的API是心啊导航
    比如`location.href`



![image-20200524105533836](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524105533836.png)

>   跳转到注册页面



![image-20200524105651835](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524105651835.png)

>   退回到上一个页面



还有一些其他的用法：

![image-20200524105833410](https://images-1255831004.cos.ap-guangzhou.myqcloud.com/online/image-20200524105833410.png)