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

