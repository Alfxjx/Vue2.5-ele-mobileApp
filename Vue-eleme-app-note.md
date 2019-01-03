### Vue.js 2.5 + cube-ui 重构饿了么 App笔记

**注意** 本文主要记录的是课程中需要改动的部分，其他的还是看视频吧。

### icon

SVG格式转换 [icomoon](https://icomoon.io/app/#/select)

```
import icons
generate fonts
preference
download
```
主要使用的是`fonts`以及`style.css`

将`fonts`放在`common/fonts`,`style.css->icon.styl`,并把css里面的{};都删了

### Mock数据

`data.json`拷贝到根目录下

Vue 2.5 /prod.server.js

```javascript
// Mock数据
const express = require('express')
const app = express()
const appData = require('../data.json')
const seller = appData.seller
const goods = appData.goods
const ratings = appData.ratings


apiRoutes.get('/seller', function (req, res) {
	res.json({
		errno: 0,
		data: seller
	});
});
//注意最后登录的网址localhost:8080/api/seller
```

### Webstorm默认模板

setting-搜索 templates

```vue
<script>
export default {
name: "${COMPONENT_NAME}"
}
</script type="text/ecmascript-6">

<style lang="stylus" rel="stylesheet/stylus">

</style>
```

### reset.css

覆盖原始元素的样式

```
Eric Meyer's Reset CSS v2.0 (http://meyerweb.com/eric/tools/css/reset/)
```

### 代码格式化

`ctrl+alt+L`

### index.html

移动端适配

```html
<meta name="viewport"
        content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=no">
  <link rel="stylesheet" type="text/css" href="static/css/reset.css">
```

### Vue组件&Router

`app.vue`中导入（`import`）组件，再`export` 到自己的`<template>`里面。

路由需要在`router/index.js`中引入路由，定义对应的路径，接着在`App.vue`中：

```
<div class="tab-item">
        <router-link to="/seller">商家</router-link>
</div>
<router-view></router-view>
```

组织

### a 标签 按钮

```
& > a
        display: block
```

**疑问** 现在给`linkActiveClass` 添加别名是在哪里



### 1px问题

```css
border-1px($color)
  position relative
  &:after
    display block
    position absolute
    left 0
    bottom 0
    width 100%
    border-top 1px solid $color
    content ''

```

按此先设置，接着对不同的屏幕尺寸进行检查，设置一个`.border-1px`，进行缩放：

```
@media (-webkit-min-device-pixel-ratio: 1.5),(min-device-pixel-ratio: 1.5)
  .border-1px
    &::after
      -webkit-transform scaleY(0.7)
      transform scaleY(0.7)
@media (-webkit-min-device-pixel-ratio: 2),(min-device-pixel-ratio: 2)
  .border-1px
    &::after
      -webkit-transform scaleY(0.5)
      transform scaleY(0.5)

```

### vue-resource

`main.js`引入：

```
import VueResource from 'vue-resource'
Vue.use(VueResource)
```

`App.vue`使用created函数：

```javascript
created () {
      this.$http.get('/api/seller').then((res) => {
        res = res.body
        if (res.errno === ERR_OK) {
          this.seller = res.data
          console.log(this.seller)
        }
      })
    },
```

改变的是：

```html
<v-header :seller="seller"></v-header>
```

于是`header.vue`中：

```javascript
export default {
    props: {
      seller: {
        type: Object
      }
    }
  }
```

使用`props`接受这个信息



### 自动折行的样式

```
white-space: nowrap;
overflow: hidden;
text-overflow: ellipsis;
```

### v-show & @click="show()"

```vue
<div class="detail" v-show="detailShow">
<div class="bulletin-wrapper" @click="showDetail"></div>

<script>
data () {
      return {
        detailShow: false
      }
    },
methods: {
      showDetail () {
        this.detailShow = true
      }
    }
</script>
```

### Sticky Footer样式

```
<div class="detail-wrapper clearfix">
        <div class="detail-main">
    	</div>
</div>
<div class="detail-close">
        <i class="icon-close"></i>
</div>
```

```stylus
.clearfix
  display inline-block
  &:after
    content "."
    display block
    height 0
    line-height 0
    clear both
    visibility hidden

.detail-wrapper
        min-height 100%
        .detail-main
          margin-top 64px
          //important
          padding-bottom 64px
      .detail-close
        position relative
        width 32px
        height 32px
        //important
        margin -64px auto 0 auto
        clear both
        font-size 32px    
    
```

### 向下去整

```javascript
Math.floor(this.score * 2) / 2
```

### 组件抽象

见`star/star.vue`

**我在抽象`line-title`时遇见的一个问题**

就是`:title="bonus"`，然后父组件里面添加数据`bonus:'优惠信息'`，接着子组件的`{{title}}`才会显示，要是不添加`bonus:'优惠信息'`，而是直接`:title="优惠信息"`，就显示不出来了。

```vue
//不显示
<lineTitle :title="优惠信息"></lineTitle>
//正确
<lineTitle :title="'商家公告'"></lineTitle>
```

这是因为绑定的子组件中的title是一个String类型的，所以`:title=""`里面也得是一个String类型的。

### 文件别名`alias`

位置在：`webpack.base.conf.js/reslove{'alias'}`

### `v-for`

for之后的 a in b, 需要加上空格

```vue
//right
<li class="support-item" v-for="(item,index) in seller.supports" :key="index">
//wrong
<li class="support-item" v-for="(item,index)in seller.supports" :key="index">
```

### api调用

```javascript
computed(){
    this.$http.get('/api/goods').then((res) => {
        res = res.body
        if (res.errno === ERR_OK) {
          this.goods = res.data
        }
      })
}
```

### `margin`重叠

外边距会重叠起来，因此不会变成想象中两倍的距离。

### ref 绑定DOM

v-ref、v-el 弃用 统一使用ref属性为元素或组件添加标记，然后通过this.$refs获取。 
开始我用:
`<div class="menu-wrapper" ref="menu-wrapper"> `
来绑定dom对象，然而浏览器报错vue.esm.js?65d7:434 [Vue warn]: Error in nextTick: “TypeError: Cannot read property ‘children’ of undefined”，在经过多次修改后，发现
`<div class="menu-wrapper" ref="menuWrapper"> `
将ref里“-”去掉，就不会报错了，应该是修改了命名规则吧。

### 圆形图标

`border-radius 50%`

### border-box模型

看看这个[链接](https://www.jianshu.com/p/006a422afb8e)

### props

`props`接受来自父组件的值，也就是`props`写在子组件的`<script>`里面，接受父组件`v-bind`过来的信息。



### 函数柯里化

[link](https://www.cnblogs.com/pigtail/p/3447660.html)

在计算机科学中，柯里化（英语：Currying），又译为卡瑞化或加里化，是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。这个技术由 Christopher Strachey 以逻辑学家哈斯凯尔·加里命名的，尽管它是 Moses Schönfinkel 和 Gottlob Frege 发明的。

这是来自维基百科的名词解释。顾名思义，柯里化其实本身是固定一个可以预期的参数，并返回一个特定的函数，处理批特定的需求。这增加了函数的适用性，但同时也降低了函数的适用范围

### 反引号

可以比较方便的拼接字符串。

```es6
`还差￥${diff}元起送`
```

### 计算属性判断对应的样式

```vue
<div class="pay" :class="payClass">{{payDesc}}</div>

<script>
      payClass () {
        if (this.totalPrice < this.minPrice) {
          return 'not-enough'
        } else {
          return 'enough'
        }
      }
</script>

<style>
  &.not-enough
    background: #2b333b
  &.enough
    background: #00b43c
    color: #fff
</style>
```
