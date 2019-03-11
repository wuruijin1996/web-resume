<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/StyleEditor'
  import ResumeEditor from './components/ResumeEditor'
  import './assets/reset.css'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 10,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* 大家好，我是吴瑞进
* 也该做一份cool cool的简历了!
* 说做就做，我也来写一份简历!
*/

/* 给所有元素加上过渡效果 */
* {
  transition: all .2s;
}
/* 设置背景颜色 */
html {
  color: rgb(222,222,222);
  background: rgb(0,43,54);
}
/* 设置边框 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  overflow: auto;
  width: 90vw;
  margin: 2.5vh 5vw;
  height: 90vh;
}
/* 太高了 */
.styleEditor {
  height: 45vh;
}
/* 代码高亮 */
.token.selector{
  color: rgb(133,153,0);
}
.token.property{
  color: rgb(187,137,0);
}
.token.punctuation{
  color: yellow;
}
.token.function{
  color: rgb(42,161,152);
}

/* 加3D效果 */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  transform: rotateX(-10deg) translateZ(-50px) ;
}

/* 准备一个编辑器 */
.resumeEditor{
  position: fixed;
  top: 50%; 
  left: 0;
  padding: .5em;  
  margin: 2.5vh 5vw;
  width: 90vw; 
  height: 45vh;
  border: 1px solid;
  background: white; 
  color: #222;
  overflow: auto;
}
/* 开始写简历 */


`,
          `
/*将Markdown格式翻译成HTML
 *再对HTML加点样式
*/
`, `
.resumeEditor{
  padding: 1em;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
  font-size: 16px;
}
.resumeEditor * {
  font-size: 14px;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, ".") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
.resumeEditor h1 {
  font-size: 18px;
}
`],
        currentMarkdown: '',
        fullMarkdown: `吴瑞进
====

坐标: 福建厦门

前端工程师, 3年开发经验

技能
----

后端开发

技术及语言
----
  - layout           : HTML5, CSS3, LESS, Bootstrap
  - Javascript       : ES5, ES6, JQuery
  - MVVM             : Vue.js, Vue-cli
  - wechat           : 小程序
  - PHP              : ThinkPHP5(熟悉)
  - Revision control : SVN, Git

工作经历
----

3. 厦门亲优大道信息科技 (2017.11 - 至今)
2. 厦门蜕潮信息科技 (2017.5 - 2017.11)
1. 福建号码网软件开发有限公司 (2016.4 - 2017.5)

项目经验
----

* kincustom
* 面向海外的B2B and B2C商城, 给用户提供数十种商品模板(包含衣服,裤子,鞋子,日用品等), 供用户进行DIY设计, 出售
* 用户在进行商品设计时, 支持 2D, 3D预览
* 支持各种主流平台(Shopify, Etsy, Bigcommerce, Woocommerce, Amazon, Squarespace等), 支持CSV以及API一键导入
* 支持个人店铺创建, 用户可以装修店铺, 上传商品, 一键分享到(FaceBook等平台)
* 已上线: https://kincustom.com (国外服务器，访问较慢，少数内容要翻墙)

教育经历
----

* 福州职业技术学院(大专)

勾引方式
----

* 手机 : 18559997346
* 邮箱 ：wuruijin1996@163.com

感谢你看到这里
----

* 说了那么多正式的, 最后轻松一下, 省的你们觉得我是个死板的程序狗.
* 本人, 男, 不弯, 爱游戏, 爱跑步, 更爱妹子, 懒癌晚期患者, 坚信懒才是人类进步的最大动力, 代码洁癖严重, 写代码当然就要整整齐齐!
* 自带设计师技能，有UI基础；
* 一个颜值3分, 时而抽风神经, 时而一丝不苟, 立志成为业界大牛并且在努力前行着的程序狗的自述.

`
      }
    },
    created() {
      this.makeResume()
    },

    methods: {
      makeResume: async function () {
        await this.progressivelyShowStyle(0)
        await this.progressivelyShowResume()
        await this.progressivelyShowStyle(1)
        await this.showHtml()
        await this.progressivelyShowStyle(2)
      },
      showHtml: function () {
        return new Promise((resolve, reject) => {
          this.enableHtml = true
          this.$nextTick(() => {
            this.$refs.resumeEditor.goTop()
          })
          resolve()
        })
      },
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) { return }
            // 计算前 n 个 style 的字符总数
            let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
                this.$nextTick(() => {
                  this.$refs.styleEditor.goBottom()
                })
              }
              setTimeout(showStyle, interval)
            } else {
              resolve()
            }
          }).bind(this)
          showStyle()
        })
      },
      progressivelyShowResume() {
        return new Promise((resolve, reject) => {
          let length = this.fullMarkdown.length
          let interval = this.interval
          let showResume = () => {
            if (this.currentMarkdown.length < length) {
              this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
              let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
              let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
              if (prevChar === '\n' && this.$refs.resumeEditor) {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
              }
              setTimeout(showResume, interval)
            } else {
              resolve()
            }
          }
          showResume()
        })
      }
    }
  }

</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    min-height: 100vh; position: relative;
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }

</style>
