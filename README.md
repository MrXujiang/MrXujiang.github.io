## 前言
最近几年**微前端**一直是前端界的热门议题, 它类似于**微服务**架构, 主要面向于浏览器端，能将一个复杂而庞大的单体应用拆分为多个功能模块清晰且独立的子应用,且共同服于务同一个主应用。各个子应用可以独立运行、独立开发和独立部署。

**微前端架构**概念的诞生及应用对于提供**复杂应用服务**的企业来说显然是一种机遇, 同样也是一种挑战.本文主要就**微前端架构**的概念和实现方案做一个总结和复盘,并且通过一个实际案例来实践**微前端架构**,希望能对同样有此需求的朋友们提供一些帮助和思路.
## 你将收获
* 什么是微服务以及微服务能给企业带来什么
* 微前端架构概念及方案
* umi下的微前端架构方案实战
* 一个程序员的技术复盘与展望

## 正文
在总结**微前端架构**之前,让我们来先看看**微服务**是什么.
### 1.什么是微服务以及微服务能给企业带来什么
> 微服务是一种用于构建应用的架构方案。微服务架构有别于更为传统的单体式方案，可将应用拆分成多个核心功能。每个功能都被称为一项服务，可以单独构建和部署，这意味着各项服务在工作（和出现故障）时不会相互影响。

传统的web软件开发架构往往如下图所示:
![](https://user-gold-cdn.xitu.io/2020/4/2/1713a39c48cce5fc?w=1404&h=1044&f=png&s=289878)
虽然我们在传统应用中可以采用**模块化**来拆分业务逻辑和开发方式,但最终它们会打包并部署为单体式应用。这种架构往往更适合中小型项目, 开发简单直接,更适合集中化管理应用.但往往也会存在很多缺点,比如可扩展性不足,相同或者相似业务复用困难,部署时间长, 业务复杂之后很难维护等问题.

对于复杂系统和业务来说,我们一般会采用微服务架构。其思路是将一个完整的应用分解为小的、互相连接的微服务,每个服务完成特定的功能, 并且某些特定的服务还能为其他服务提供API接口.
![](https://user-gold-cdn.xitu.io/2020/4/2/1713a60b87e67bbd?w=1546&h=900&f=png&s=284706)
由上图可以发现微服务给我们带来的好处:
* 将一个庞大的单体拆解成多个子服务,大大降低了开发复杂度
* 任务边界划分明确, 每个子服务之间单独开发, 不同服务之间可并行由不同的开发人员开发,提高开发效率
* 更细粒度的加强了模块化进程, 可维护性和可读性更高
* 团队之间只要制定好API约定, 那么不同成员或者团队可以采用不同的技术开发服务
* 可用共享服务, 使得不同子服务可组合实现更复杂的功能
* 每个微服务可独立部署发布,使得自动化**CI**(持续集成)/**CD**(持续交付)成为可能

但微服务并不是任何场景下都是合适的, 微服务的目标是充分分解应用程序，以促进敏捷开发和持续集成部署。在部署微服务时我们需要做好适当的边界划分,并处理不同微服务之间的并发问题,这些都是对整个项目带来的挑战,需要更加专业的技术成员来把控.目前市面上也有很多开源的微服务框架比如**Dubbo**, **Spring Cloud**等,笔者之前公司采用的**Spring Cloud**就是一个很好的微服务架构方案.

### 2.微前端架构概念及方案
#### 2.1 理解微前端架构
上面简单介绍了一下**微服务**架构,接下来我们进入主题,来聊聊**微前端**. **微前端**和**微服务**实现的目的类似,都是将应用由单一的单体应用转变为多个小型子应用,差别就在于:
1. **微前端**应用于浏览器端,主要是对web应用进行拆解,最后将不同子系统(模块)聚合成一个完整的应用.
2. **微前端**主要目的是**聚合**,即将不同子系统聚合成一个大系统,而**微服务**架构目前更多是解耦,即解耦不同服务间的依赖.

我们先来看看微前端的一些思考者.
![](https://user-gold-cdn.xitu.io/2020/4/2/1713aaca25100e8b?w=1950&h=1102&f=png&s=980991)
Michael Geers 大佬发表了一些对微前端的一些思考,内容大致总结一下就是:
> “ 微前端 ”是将微服务的概念扩展到前端领域。为了构建一个功能强大的浏览器应用程序（也称为单页应用程序），当前普遍的趋势是将其建立在微服务架构之上。但是随着时间的流逝，通常由独立团队开发的前端层会不断增长，并且变得更加难以维护。Micro Frontends背后的想法是将Web应用程序视为由独立团队拥有的功能的组合。每个团队都有自己关心和专长的不同业务或任务领域。每个团队可以跨职能，并且从数据库到用户界面，端到端地开发其功能。

正如下面的例子所示:
![](https://user-gold-cdn.xitu.io/2020/4/2/1713a9b02e4dc81a?w=1846&h=1044&f=png&s=90195)
当我们的系统中有多个不同的子模块,并且子模块之间有相对独立且完整的功能体系时, 一旦子模块变得越来越多, 那么整个应用将变得非常庞大且臃肿,开发和维护成本大大提高.如果在这种场景下我们采用SPA模式开发,那么项目后期将变得不可想象,页面首次加载将变得非常慢(笔者曾经就经历过,开发一个复杂系统页面首次加载花了20多秒,所以不得不采用MPA来处理); 但是我们采用MPA(多页应用)模式,虽然解决了应用臃肿的问题, 但仍然存在很多有待处理问题,比如模块切换需要重新刷新页面, 公共组件无法共享,子模块直接,父子模块之间的通信问题,开发部署繁琐等.这写都是传统开发模式会遇到的问题.

试想一下,如果面对以上问题, 如果有一种架构模式, 可以让我们在主应用中共享公共组件和状态(但是要保证子应用运行时内部的状态隔离), 并且不同子模块之间可以单独开发部署, 模块间切换不刷新页面, 并且模块之间,父子应用之间可以通过某种简单的方式实现通信,这样是不是就完美了?不错, 这也许就是微前端要解决的问题.
![](https://user-gold-cdn.xitu.io/2020/4/2/1713ae33837e7b4c?w=1610&h=806&f=png&s=183173)
往往企业的中后台系统更加适合使用微前端架构,因为B端大部分都是业务驱动,而往往这些业务之间会非常复杂, 比如Saas, Paas等系统.像类似于云平台,CRM,ERP系统往往是微前端架构的拯救对象.

笔者曾经接手的ERP系统,由于开发时间比较早,往往有很多遗留的历史包袱,比如新老技术栈的糅合, 前端业务代码庞大而毫无违和感,为了对其继续迭代开发, 重构是必经之路,微前端另一个重要的作用笔者认为就是**解放**.解放不可挽回的痛😭.

笔者之前在写[从0到1教你搭建前端团队的组件系统（高级进阶必备）](https://juejin.im/post/5e4d3a8de51d45270a709954)这篇文章时简单提了一下微前端的一些知识,这里先回放一张笔者之前做的简陋的图,方便大家理解微前端架构.
![](https://user-gold-cdn.xitu.io/2020/4/2/1713af859e3a1aea?w=1248&h=960&f=png&s=347757)
但我们所看到的不是事实的全部,在文章后面笔者会总结一张更全面的图,来整理微前端的一些实践应用.
#### 2.2 微前端架构实现的方案
上面内容我们大致了解了一下微前端的概念和作用,接下来笔者总结一下曾经经历过的微前端实现方案.
 
**1. 基于MPA + iframe + requirejs实现的微前端架构**
这是笔者接手最早的一个项目,主要是服务于企业内部的ERP系统,当时采用的技术栈主要是jquery+layui+requirejs,记得还是笔者刚毕业时参与的项目.主要是利用AMD模块化机制来复用代码,当时项目代码及其庞大复杂,大致架构如下:
![](https://user-gold-cdn.xitu.io/2020/4/2/1713b134b02105a2?w=1060&h=1188&f=png&s=93038)
传统实现方式一般是通多多页面的方式来对应用解耦,并采用模块化加载机制来导入可复用组件.系统间通型采用storage+window.opener+iframe.比如我们一个大系统的首页可能会有来自不同子系统的页面,已iframe的方式嵌入.不同子系统间由不同的人维护,虽然做到了微前端的模式,但是还是有很多缺点.
![](https://user-gold-cdn.xitu.io/2020/4/2/1713b19ed8a9c20b?w=1614&h=1210&f=png&s=111273)
![](https://user-gold-cdn.xitu.io/2020/4/2/1713b4db5aabca53?w=1386&h=996&f=png&s=61532)
改架构的一般分工是架构组主要负责制定架构标准和规范，维护公共模块（类似于现在的组件，当时由于采用jquery生态，可以简单的说成公共UI插件的维护）；业务组主要负责编写业务代码，实现某个系统的具体交互和功能。

**2. 基于MPA + iframe + Webpack实现的微前端架构**
基于MPA + iframe + Webpack实现的微前端架构实现和上面介绍的传统架构实现类似，只不过采用了更新的技术栈，以及真正的模块化，组件化技术。笔者之前做的Saas系统就是一个很典型的该方案的例子：
![](https://user-gold-cdn.xitu.io/2020/4/2/1713b5cb91350bf2?w=1620&h=1436&f=png&s=210332)
上图可知不同子系统之间可以各自维护，单独打包部署，最后通过node或者nginx做路由分发。我们采用公共的ui组件库和js类库来抽离公共组件，但是前提是不同组件库和技术栈强相关，如果没有历史遗留的新项目，建议采用一致的技术栈。

以上两个方案的缺点就是组件库只能复用而无法真正共享，并且切换路由会导致页面重新渲染刷新。父子系统通信困难，仍然需要iframe最为容器来通信。（有关iframe父子页面通信的各种方式笔者在[记一次老项目中的跨页面通信问题和前端实现文件下载功能](https://juejin.im/post/5d8399e8f265da03a9506fc6)）

**3. 基umi + qiankun实现的微前端架构**
目前市面上也有非常成熟的微前端架构方案，比如single-spa，之前的美团外卖微前端架构和蚂蚁金服基于single-spa开发的微前端架构qiankun（乾坤），都是非常不错的方案。

qiankun主要采用HTML Entry模式，直接将子应用打出来 HTML作为入口，主框架可以通过 fetch html 的方式获取子应用的静态资源，同时将 HTML document 作为子节点塞到主框架的容器中。这样不仅可以极大的减少主应用的接入成本，子应用的开发方式及打包方式基本上也不需要调整，而且可以天然的解决子应用之间样式隔离的问题。
![](https://user-gold-cdn.xitu.io/2020/4/2/1713b73db785ab21?w=1024&h=614&f=png&s=39497)
其方案具有如下特点：
* 支持子应用并行
* 支持js沙箱环境（js隔离）
* css隔离
* HTML Entry，简化开发者使用
* 按需加载
* 公共依赖加载
* 父子应用通信机制
* 子应用嵌套

因为我们的前端项目基于umi生态开发构建（之前采用webpack搭建，后面发现umi使用也很爽，就直接基于umi二次开发了），所以很自然的选择了乾坤作为微前端架构。具体架构如下：
![](https://user-gold-cdn.xitu.io/2020/4/2/1713b87435d6c0ed?w=1736&h=1480&f=png&s=258936)

### 3.umi下的微前端架构方案实战
接下来我具体介绍如何使用乾坤来搭建我们的微前端架构，由于我们内部采用umi，所以这里我们直接使用其提供的@umijs/plugin-qiankun插件来实现。（好处就是改动成本几乎为零）
首先我们来实现这样一个场景：我们有一个主应用作为基座工程，然后有3个子系统，他们是独立创建维护的，可以采用不同的git仓库来管理。当我们在主应用中切换路由时会切换到对应的子系统，且不刷新页面，完全的SPA体验，接下来我们来具体实现一下吧。

这里我们采用umi2.0来开发，关于如何安装与使用umi，这里就不做详细介绍了。
1. 创建工程
``` js
mkdir mainSystem subSystemA subSystemB subSystemC
// 分别进入各系统目录下，执行以下命令创建我们的项目
yarn create umi
```
2. 在各个系统下安装@umijs/plugin-qiankun插件
``` js
yarn add @umijs/plugin-qiankun
```
3. 主应用下的配置
``` js
// .umirc.js
export default {
  plugins: [
    [
      '@umijs/plugin-qiankun',
      {
        master: {
          // 注册子应用信息
          jsSandbox: true, // 是否启用 js 沙箱，默认为 false
          prefetch: true, // 是否启用 prefetch 特性，默认为 true
        },
      },
    ],
  ],
};
// app.js
const isDev = process.env.NODE_ENV === 'development'
export const qiankun = {
  apps: [
    {
      mountElementId: 'root-subapp-container',  // 洗子应用挂载的节点
      name: 'subSystemA', // 唯一 id，和package对应的name字段最好保持一致
      entry: isDev ? '//localhost:8001' : '/subSystemA/index.html', // html entry
      base: '/subSystemA',  的路由前缀，通过这个前缀判断是否要启动该应用，通常跟子应用的 base 保持一致
      history: 'browser', // 子应用的 history 配置，默认为当前主应用 history 配置
    },
    {
      mountElementId: 'root-subapp-container',
      name: 'subSystemB',
      entry: isDev ? '//localhost:8002' : '/subSystemB/index.html',
      base: '/subSystemB',
    },
    {
      mountElementId: 'root-subapp-container',
      name: 'subSystemC',
      entry: isDev ? '//localhost:8003' : '/subSystemB/index.html',
      base: '/subSystemC',
    }
  ],
  fetch: url => {
    return fetch(url)
  }
};
```
以上是基本的配置，当然我们还可以在app.js里面加入lifeCycles等钩子来控制不同生命周期下的行为。

在子应用中我们同样需要引入@umijs/plugin-qiankun这个插件，具体配置如下：
``` js
export default {
  base: `/${appName}`, // 子应用的 base，默认为 package.json 中的 name 字段
  plugins: ['@umijs/plugin-qiankun', { slave: true }],
};
```
基本准备工作已经完成，剩下就是编写业务代码了，我们要想让整个应用一起跑起来，需要先启动基座工程，然后再启动对应的子工程(当然他们可以单独运行)。

但是值得注意的是我们在开发环境中采用devServer提供的带来才能跨域抓取资源，如果应用发布到线上，如果不同子应用采用不同域名，我们还需要解决跨域问题（跨域解决的方案及安全机制也有很多，已经不再是个问题）。实际开发环境我们需要考虑的问题还有很多，这里只做简单介绍，不过根据官方提供的api基本上可以满足大部分的需求场景，所以还是非常值得一试的。笔者后期也会写一个微前端的实际案例发布到github上，可以一起交流学习。
### 4.一个程序员的技术复盘与展望
由于今年受疫情影响，工作任务比较紧张，复盘的时间也比较少，但是笔者还是会坚持每周更新1-2篇文章，来总结和复盘工作中或学到的新技术。自己写的零零散散的文章是时候做一个分类和梳理，以便更加清晰未来的方向和不足的补救。
#### 1.node相关
* [使用nodeJs开发自己的图床应用](https://juejin.im/post/5e74cd78e51d4527196d785f)
* [30分钟带你了解Web工程师必知的Docker知识](https://juejin.im/post/5e68f448518825493038db87)
* [30分钟教你优雅的搭建nodejs开发环境及目录设计](https://juejin.im/post/5e60d50de51d4526c70fb702)
* [基于nodeJS从0到1实现一个CMS全栈项目的服务端启动细节](https://juejin.im/post/5d8f5107f265da5bb74640eb)
* [基于nodeJS从0到1实现一个CMS全栈项目（上）](https://juejin.im/post/5d8af4cd6fb9a04e0925f3d8)
* [基于nodeJS从0到1实现一个CMS全栈项目（中）（含源码）](https://juejin.im/post/5d8c7b66518825761b4c1e04)
* [基于nodeJS从0到1实现一个CMS全栈项目的服务端启动细节](https://juejin.im/post/5d8f5107f265da5bb74640eb)
* [5分钟教你用nodeJS手写一个mock数据服务器](https://juejin.im/post/5d7345bce51d453b76258503)

#### 2.工程化相关
* [前端组件/库打包利器rollup使用与配置实战](https://juejin.im/post/5dab0cc1e51d4524df35b7b4)
* [基于react/vue生态的前端集成解决方案探索与总结](https://juejin.im/post/5d2f25d9f265da1b84671b57)
* [9012教你如何使用gulp4开发项目脚手架](https://juejin.im/post/5d2180e5e51d4577614761b7)
* [vue/react项目中不可忽视的自动化部署方案](https://juejin.im/post/5d1c0542f265da1b7153100c)
* [用 webpack 4.0 撸单页/多页脚手架 (jquery, react, vue, typescript)](https://juejin.im/post/5d078cc16fb9a07ef37668d0)
* [git常见用法和核心策略](https://juejin.im/post/5d00feb7e51d45775e33f544)

#### 3. 设计模式
* [15分钟带你了解前端工程师必知的javascript设计模式(附详细思维导图和源码)](https://juejin.im/post/5e32bbc9f265da3e1a59b2ab)
* [《前端实战总结》之使用解释器模式实现获取元素Xpath路径的算法](https://juejin.im/post/5de993fde51d4557ed541a28)
* [《前端实战总结》之迭代器模式的N+1种应用场景](https://juejin.im/post/5de32c0b518825434771d13e)
* [《前端实战总结》之设计模式的应用——备忘录模式](https://juejin.im/post/5dcad1906fb9a04a6076bd53)

#### 4.react相关
* [10分钟教你手写8个常用的自定义hooks](https://juejin.im/post/5e57d0dfe51d4526ce6147f2)
* [《彻底掌握redux》之开发一个任务管理平台](https://juejin.im/post/5e4ff9b9f265da57537eb091)
* [从0到1教你搭建前端团队的组件系统（高级进阶必备）](https://juejin.im/post/5e4d3a8de51d45270a709954)
* [精通React/Vue系列之实现一个全局提示(Message)组件](https://juejin.im/post/5e4a1965e51d4526e418f842)
* [精通React/Vue系列之手把手带你实现一个功能强大的通知提醒框(Notification)](https://juejin.im/post/5e47e8a9e51d45270d53054f)
* [手摸手实现一个轻量级可扩展的模态框(Modal)组件](https://juejin.im/post/5e416e78e51d45270f52b062)
* [《精通react/vue组件设计》之5分钟教你实现一个极具创意的加载(Loading)组件](https://juejin.im/post/5e417dfc6fb9a07cb24a9091)
* [《精通react/vue组件设计》之实现一个健壮的警告提示(Alert)组件](https://juejin.im/post/5e40b209f265da57301be62f)
* [《精通react/vue组件设计》之配合React Portals实现一个功能强大的抽屉(Drawer)组件](https://juejin.im/post/5e3e7b8fe51d4526fb5dcddb)
* [《精通react/vue组件设计》之5分钟实现一个Tag(标签)组件和Empty(空状态)组件](https://juejin.im/post/5e3cc898e51d4526d87c6037)
* [《精通react/vue组件设计》之快速实现一个可定制的进度条组件](https://juejin.im/post/5e366772f265da3e307710dd)
* [《精通react/vue组件设计》之用纯css打造类materialUI的按钮点击动画并封装成react组件](https://juejin.im/post/5e35294d518825263237f3ba)
* [基于jsoneditor二次封装一个可实时预览的json编辑器组件(react版)](https://juejin.im/post/5e302af8e51d453cc04abc56)
#### 5.vue/angular相关
* [从零到一教你基于vue开发一个组件库](https://juejin.im/post/5e63d1c36fb9a07cb427e2c2)
* [2年vue项目实战经验汇总](https://juejin.im/post/5e390a6df265da57503cb415)
* [基于create-react-app打包编译自己的第三方UI组件库并发布到npm](https://juejin.im/post/5dacee1c6fb9a04de04d94e0)
* [一张图教你快速玩转vue-cli3](https://juejin.im/post/5d1782eaf265da1ba91592fc)
* [使用Angular8和百度地图api开发《旅游清单》](https://juejin.im/post/5d0dd545f265da1bd42488f5)
* [vue高级进阶系列——用typescript玩转vue和vuex](https://juejin.im/post/5cc4b1306fb9a032471563e2)

#### 6.css相关
* [《前端实战总结》之使用纯css实现网站换肤和焦点图切换动画](https://juejin.im/post/5dfb4803e51d4557ff1410af)
* [《前端实战总结》之使用CSS3实现酷炫的3D旋转透视](https://juejin.im/post/5dd16b39f265da0bca78958e)
* [《css大法》之使用伪元素实现超实用的图标库（附源码）](https://juejin.im/post/5da083346fb9a04dea5de9a5)
* [css3实战汇总（附源码）](https://juejin.im/post/5d886b6e6fb9a06af92be109)
* [用css3实现惊艳面试官的背景即背景动画（高级附源码）](https://juejin.im/post/5d86fc096fb9a06ae94d6d7a)
* [如何把控css的方向感](https://juejin.im/post/5d044a5051882510715e3651)


#### 7.算法相关
* [如何优雅的使用javascript递归画一棵结构树](https://juejin.im/post/5d7f2ddde51d4561ba48fe8c)
* [笛卡尔乘积的javascript版实现和应用](https://juejin.im/post/5d694435518825391623e5c4)
* [JavaScript 中的二叉树以及二叉搜索树的实现及应用](https://juejin.im/post/5d4556895188255d845ffef1)
* [js基本搜索算法实现与170万条数据下的性能测试](https://juejin.im/post/5d05adf75188257152111648)
* [《前端算法系列》如何让前端代码速度提高60倍](https://juejin.im/post/5d034e83e51d45773e418a69)
* [《前端算法系列》数组去重](https://juejin.im/post/5cffac5de51d45777b1a3d6f)

#### 8. H5游戏
* [用60行代码实现一个高性能的圣诞抽抽乐H5小游戏(含源码)](https://juejin.im/post/5e02cddf518825125015fd2f)
* [Canvas入门实战之用javascript面向对象实现一个图形验证码](https://juejin.im/post/5d3e4bc9f265da1bca5223cf)
* [用 JavaScript 和 C3 实现一个转盘小游戏](https://juejin.im/post/5d36f8f1e51d454d1d6285f0)
* [教你用200行代码写一个爱豆拼拼乐H5小游戏（附源码）](https://juejin.im/post/5d33d3c26fb9a07ed740b906)
#### 9.原生javascript类库设计封装
* [浏览器缓存库设计总结（localStorage/indexedDB）](https://juejin.im/post/5e8024f451882573954e9bda)
* [基于 localStorage 实现一个具有过期时间的 DAO 库](https://juejin.im/post/5d266dfff265da1ba77ccd72)
* [如何用不到200行代码写一款属于自己的js类库](https://juejin.im/post/5d1e26a2e51d45595319e3a9)
* [3分钟教你用原生js实现具有进度监听的文件上传预览组件](https://juejin.im/post/5d142720f265da1bc4146433)

#### 10.工作问题汇总
* [5分钟教你使用console.log发布公司的招聘信息](https://juejin.im/post/5e725563f265da57602c6eb6)
* [2019年,盘点一些我出过的前端面试题以及对求职者的建议](https://juejin.im/post/5e2715f06fb9a02fe34bc73c)
* [《前端实战总结》之使用pace.js为你的网站添加加载进度条](https://juejin.im/post/5dcec7d851882510ba1cbf95)
* [《前端实战总结》之使用postMessage实现可插拔的跨域聊天机器人](https://juejin.im/post/5dc301a76fb9a04a7b29cdc3)
* [《前端实战总结》之变量提升，函数声明提升及变量作用域详解](https://juejin.im/post/5dbc49db6fb9a020512b3f8f)
* [《前端实战总结》如何在不刷新页面的情况下改变URL](https://juejin.im/post/5db9ac9cf265da4d4a3061a2)
* [快速掌握es6+新特性及es6核心语法盘点](https://juejin.im/post/5d9843de6fb9a04def4e57d2)
* [web性能优化的15条实用技巧](https://juejin.im/post/5d935ebbe51d4577e86d0d85)
* [《javascript高级程序设计》核心知识总结](https://juejin.im/post/5d8c86d06fb9a04e172071a0)
* [前端开发中79条不可忽视的知识点汇总](https://juejin.im/post/5d8989296fb9a06b1f147070)
* [记一次老项目中的跨页面通信问题和前端实现文件下载功能](https://juejin.im/post/5d8399e8f265da03a9506fc6)
* [让你瞬间提高工作效率的常用js函数汇总(持续更新)](https://juejin.im/post/5d1a45b0f265da1bb277494c)
* [前端三年，谈谈最值得读的5本书籍](https://juejin.im/post/5cb690fdf265da035d0c7299)

## 最后
如果想学习更多**H5游戏**, **webpack**，**node**，**gulp**，**css3**，**javascript**，**nodeJS**，**canvas数据可视化**等前端知识和实战，欢迎在公号《趣谈前端》加入我们的技术群一起学习讨论，共同探索前端的边界。
![](https://user-gold-cdn.xitu.io/2020/2/2/170060658dd3db98?w=1158&h=400&f=png&s=138051)
