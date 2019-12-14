## webpack结合svg-sprite-loader开发svg-icon

继[svg Sprite介绍](./svg_Sprite介绍.md)

### 一、安装 svg-sprite-loader
```js
npm install svg-sprite-loader --save
```

### 二、在公共组件目录下创建SvgIcon.vue和index.js
在SvgIcon.vue中：
```html
<template>
  <svg :class="svgClass">
    <use :xlink:href="iconName" />
  </svg>
</template>
<script>
export default {
  name: 'SvgIcon',
  props: {
    iconClass: {
      type: String,
      required: true
    },
    className: {
      type: String,
      default: ''
    }
  },
  computed: {
    iconName () {
      return `#icon-${this.iconClass}`
    },
    svgClass () {
      if (this.className) {
        return 'svg-icon ' + this.className
      } else {
        return 'svg-icon'
      }
    }
  }
}
</script>
<style scoped>
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>
```

在index.js中：
```js
import Vue from 'vue'
import SvgIcon from '@/components/SvgIcon' // svg组件
// 注册为全局组件
Vue.component('svg-icon', SvgIcon)
const req = require.context('@/assets/icons', false, /\.svg$/)
const requireAll = requireContext => requireContext.keys().map(requireContext)
requireAll(req)
```
然后在main.js中引入该文件index.js
> 在引入svg文件时，使用webpack提供的require.context Api来动态批量引入svg icon。它接收三个参数，第一个是文件夹路径，第二个是是否遍历子文件夹，第三个是匹配文件的正则表达式。

### 三、配置webpage
在vue.config.js中加入以下代码:
```js
function resolve(dir){
    return path.join(__dirname, dir)
}

module.exports = {
    chainWebpack(config) {
        // 解析所有svg文件
        config.module
            .rule('svg')
            .exclude.add(resolve('src/icons'))
            .end()
        // 合并所有指定的svg到一个svg文件中
        config.module
            .rule('icons')
            .test(/\.svg$/)
            .include.add(resolve('src/icons'))
            .end()
            .use('svg-sprite-loader')
            .loader('svg-sprite-loader')
            .options({
                symbolId: 'icon-[name]'
            })
            .end()
    }
}
```
完成以上操作，运行项目后，页面中会自动插入节点，类似：
```html
<svg>
    <symbol id='icon1'>
        <!--第一个svg图标的path代码-->
    </symbol>
    <symbol id='icon2'>
        <!--第二个svg图标的path代码-->
    </symbol>
    <symbol id='icon3'>
        <!--第三个svg图标的path代码-->
    </symbol>
<svg>
```

### 四、调用组件

```html
<template>
    <section>
        <svg-icon class="testIcon" icon-class='icon1'/>
    </section>
</template>
<script>
export default {
  name: 'TestIcon',
}
</script>
<style scoped>
.testIcon {
  font-size: 24px;
  fill: blue;
}
</style>
```


以上就完成了svgicon组件开发。svg文件需要提前移除fill属性和stroke属性
