## 什么是BEM

> BEM 是一种命名规范，是块（block）、元素（element）、修饰符（modifier）的简写，是 Yandex 团队提出的一种前端命名方法论。它利用不同的区块，功能以及样式来给元素命名，其目的是将用户界面分割成独立的区块，使得即使有复杂的UI，界面开发也能变得更加简单和快速；这种巧妙的命名方法让你的 CSS 类对其他开发者来说更加透明而且更有意义。

## BEM命名的好处

> BEM是定义了一种css class的命名规范，每个名称及其组成部分都是存在一定的含义，让前端代码更容易阅读和理解，更容易控制和扩展，更加健壮和明确，能够帮助开发者实现模块化、低耦合可复用、高可维护性和结构化的CSS代码。

## BEM规则写法

.block （块）

.block__element (元素)

.block__element--modifier (修饰符)

例如下面是一个卡片的html结构
```
<div class="card">
  <div class="cart__head">
    <div class="cart__title">card 标题</div>
  </div>
  <div class="cart__body">
  card内容
  </div>
  <div class="cart__foot">
    <button class="cart__button">cart footer 按钮（正常状态）</button>
    <button class="cart__button cart__button--disable">cart footer 按钮（禁止状态）</button>
  </div>
</div>
```
#### 通用型模块命名
+ wrapper 最外层容器
+ container 容器/主体内容
+ header 头部
+ footer 底部
+ nav/menu 导航了
+ aside 侧边栏
...

#### 功能型模块命名
+ search
+ login
+ title
+ tab
...

连词之间用建议用中横线（-）连接

## 哪些UI组件使用了BEM命名规范

#### 饿了么 element ui
```
.el-input 块（一个整体）
.el-input__wrapper  元素（外围包装）
.el-input__inner 元素（input框）
.el-input__clear 元素（清除元素）

.el-input—large 修饰符（input大小）
.el-input—small 修饰符（input大小）

对于less/sass可以这样写

.el-input {
 &__wrapper{}
 &__inner{}
 &__close{}
 &—large{}
 &—small{}
}
```
#### 有赞 vant ui
```
.van-popover 块（一个整体）
.van-popover__arrow 元素（小三角形）
.van-popover__content 元素（内容区域）
.van-popover__action 元素（按钮）
.van-popover__action—disabled  元素（按钮）— 禁止点击状态（修饰符）
.van-popover__action-icon  元素（按钮图标）
.van-popover__action-text 元素（按钮文字）

html结构
<div class=“van-popover”>
  <div class=“van-popover__arrow”></div>
  <div class=“van-popover__content”>
    <div class=“van-popover__action”>
       <div class=“van-popover__action-icon”></div>
       <div class=“van-popover__action-text”>选项1</div>
    </div>
    <div class=“van-popover__action van-popover__action—disabled”>
       <div class=“van-popover__action-icon”></div>
       <div class=“van-popover__action-text”>选项2</div>
    </div>
  </div>
</div>
```
