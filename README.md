## 进程和线程

- 进程：可以简单的理解为操作系统分配给程序运行所需的专属内存空间，每个程序至少都有一个进程，进程之间是相互独立的，即使互相通信也需要双方同意并且代价比较大
- 线程：进程内真正运行代码的单位就是线程
    - 前端接触较多的线程间通信
        - 同进程下的线程公用一个资源池或者说是公用一个内存空间，可以通过访问或者读写内存数据进行通信（最常见的）
        - 消息传递，前端来说就是浏览器会分很多进程，进程下也会分很多线程（浏览器主进程、网络进程、渲染进程等），渲染进程又会有一个渲染主线程负责解析html、css和运行js，渲染主线程用来处理各种任务的方式就是队列
            - 微队列：优先级最高，常接触的将一个函数添加到微队列的方式就是Promise
            - 交互队列：用于存放用户在页面操作后产生的事件任务，优先级低于微队列
            - 延时队列：⽤于存放计时器到达后的回调任务，优先级低于交互队列

## 浏览器任务有优先级吗？

- 任务本身没有优先级，在消息队列中先进先出，但消息队列有优先级，根据W3C的最新标准，
    - 每个任务都有一个任务类型，根据任务类型将不同类型的任务分配到不同的消息队列中，在后续的任务循环中，浏览器根据实际情况从不同的队列中取出任务并执行
    - 浏览器必须准备好一个微队列，微队列中的任务是优先于其它所有任务执行的

## http三次握手四次挥手
- 三次握手：客户端向服务端发送报文，进入SYN-SENT状态 => 服务端收到报文将连接放入预备队列中，之后向客户端发送确认报文，并进入SYN-RECEIVED状态 => 客户端收到确认报文，向服务端发送确认报文ESTABLISHED
    - 两次不安全，四次没必要
- 四次挥手：
    - 客户端数据发送成功后，向服务端发送终止请求的报文
    - 服务器收到终止报文后释放TCP连接，不再接收数据但是依然可以向客户端传输数据
    - 服务端数据发送完成后向客户端发送终止报文
    - 客户端收到终止报文后，向服务器发送确认终止报文，一段时间后没收到服务端的数据关闭连接，服务端收到终止报文后也关闭连接

## https为什么更加安全

- 主要是因为HTTPS通过SSL/TLS（加密保护协议）协议进行加密，以及提供了身份认证和完整性保护。公钥和私钥 公钥用来加密 私钥用来解密
    - 数据加密。HTTPS使用加密算法对传输的数据进行加密，确保即使数据在传输过程中被截获，也无法被解密。
    - 身份认证。HTTPS通过数字证书来验证通信对方的身份，防止黑客伪造网站，确保用户连接到正确的网站。
    - 完整性保护。HTTPS通过摘要算法来检查数据在传输过程中是否被篡改，如果数据被篡改，接收方能够识别出来。


## 输入URL后浏览器发生了什么

- 主要负责的进程为 主进程、渲染简称、网络进程
- 输入URL后首先会对URL进行解析（协议、域名、端口、路径、参数等），输入的如果不是IP还会进行DNS进行解析
    - DNS解析优化：使用CDN缓存，加快CDN找到资源的速度（dns-prefetch）
- 建立TCP连接：TCP三次握手四次挥手
- 拿到需要的基础资源后开始后续的解析
- 解析html、css和js等资源
- 解析这步又主要分为构建dom树和构建cssom树（生成stylesheet，封装组件常用，日常开发不常用）
- 两个树都生成后开始合并渲染树，根据渲染树的描述确定浏览器每个像素的样式
- 布局和绘制 浏览器根据渲染树进行布局（Layout）和绘制（Paint）
    - 布局就是根据DOM树生成Layout树，Layout树跟DOM树并不是一一对应的（比如display：none，before等，根据实际样式删除或者新增节点）
    - 布局之后进行分层
        - 会影响浏览器分层的css属性：position：fixed或者position：absolute
        - opacity 透明度会导致浏览器为该元素创建一个新的图层
        - transform 旋转缩放还有translatez可以让元素获得3D比那换，并导致浏览器单独为它创建一个图层
        - video和canvas元素默认自己有自己的图层，并且它们会享受浏览器内置的GPU加速
        - will-change属性会提示浏览器预先合成一个图层，以优化动画性能
    - 这里的绘制，就是为每一层生成如何绘制的渲染指令，并将指令全部推送给浏览器的渲染主线程
- 绘制结束后 浏览器会进行分块，即将每一层分为多个小块（方便后续的光栅化），分块工作是由渲染主线程外的多个线程同时进行的
- 分块结束后会进行光栅化，即将每个块转变成位图，并且有限处理靠近视口的块，此过程会⽤到 GPU 加速

## 什么是reflow（回流）和repaint（重绘）

## 什么是GPU加速
- 浏览器将一部分运算密集的任务交给显卡完成的性能优化策略，video和canvas元素，transform、opcity、transition（css3的动画）

## DOMContentLoad 和 onLoad

- DOMContentLoad是dom树生成完之后
- dom树生成完不代表页面加载完 还有其他静态资源要下载 这些静态资源下载完之后 执行onLoad


## 如何避免重绘回流

- https://www.php.cn/faq/618114.html
- will-change 
 - https://blog.csdn.net/zz_jesse/article/details/132613789
 - css样式 告诉浏览器这个元素将会发生某种变化
- requestAnimationFrame
- getBoundingClientRect 常用来做懒加载 但会引起重绘回流 可以换为 intersectionObserver


## DocumentFragment
- 文档片段/文档碎片 原理就是直接将文档片段替换DOM树，不会引起回流和重绘
- 列表优化


## Mutationobserver

- 监控dom删改 常用于水印变化 计算首屏时间

## vue2 cumputed和watch

- 计算属性：多对一（缓存）dirty
- 监听属性：一对一
    - immediate:立即执行
    - deep：深度监听
    - handler：函数


## 双向绑定/响应式原理

- 在beforeCreate 和 created之间初始化data
- Object.defineProperty 设置getter、setter
- __ob__对象
- 数组：出于性能考虑 不能每项都设置getter、setter 新增了一个对象集成Array的原型 改写了数组的原型方法
- dep 和 watcher
    - https://blog.csdn.net/weixin_69810763/article/details/135478552

## vue.observeable
- 2.6新增的api，将一个对象变为响应式数据，将一个数据共享给各个组件，是vuex或bus的轻量级的解决方案，缺陷是数据修改太过随意
- vue3中被其它api替代，reactive

## dep、watcher相互收集

- watcher为什么收集dep


## diff算法

- 虚拟DOM,对象
- 每个组件都有唯一key
- Vue的diff算法是一种用于比较虚拟DOM树之间差异的高效算法。它是Vue的核心特性之一，允许Vue以一种有效的方式更新DOM以反映数据的最新状态。
- Vue的diff算法采用深度优先的递归方式比较两棵虚拟DOM树的差异。它会尽可能地复用老的虚拟节点，只会对差异发生变化的部分进行最小化的DOM更新。
- 数组对比：四个指针 新前新后 旧前旧后 就是描述diff的具体实现算法：对比新数列头的元素与旧数列头部元素，如果相同新头放在旧数列的头部，新尾放在旧数列的尾部，旧数列则遍历对比新数列中有没有相同项，如果没有就直接删除
- class不同时 不管 只认key


## 箭头函数为什么不能作为构造函数

- 箭头函数没有自己的上下文，没有原型
- Promise：https://www.cnblogs.com/lvdabao/p/es6-promise-1.html
    - then链式
    - 需求：请求超时 怎么用Promise实现 race方法
    - allSettled 方法 https://blog.csdn.net/iloveyu123/article/details/116588214
- async/await https://blog.csdn.net/weixin_45811256/article/details/123638582
    - generator + Promise
    - await + 函数 不等待定时器
    - async函数执行完返回什么


## 性能优化

- 按钮：防抖
- 懒加载
- vue的路由懒加载 component:()=>import('url') 这样写只有当访问对应路由时，加载组件的方法才会执行，优化了性能
- 节流 https://blog.csdn.net/m0_64346035/article/details/124293989


## 什么是策略模式，应用场景？

- https://www.51cto.com/article/688731.html
- 比如表单验证


## vue-admin

- 请求封装：axios 响应拦截器和请求拦截器
- 怎么实现取消axios的重复请求


## cookie

- cookie与其他本地缓存的区别


## SEO

- nuxtjs vue服务器渲染的整合方案
    - vue普遍开发的是SPA（单页面应用），打包后只有一个index.html作为入口，SPA的好处就是由于是在一个页面上加载内容，用户在切换页面时不会全部重绘页面，只会部分重绘，因为用户体验更好，但是这些优点是牺牲了首屏加载速度和SEO优化的来的，而一般页面想要优化首屏加载速度最常用的方案就是SSR即服务器加载（将首屏页面在服务器加载完成后发给客户端，客户端直接加载服务器返回的页面），nuxtjs就是给vue提供的采用类似思路解决SPA问题的一种整合后的解决方案
- tdk


## 自定义SEO优化

- tdk
- 语义化标签
- h1不嵌入其它元素
- site-map
- 微数据结构化 
    - https://www.cnblogs.com/zdz8207/p/seo-google-rich-snippets.html


## axios.js fetch
## axios如何做到nodejs、浏览器通用

- node：http模块封装 net
- 浏览器：xhr
- fetch


## 大文件的下载和暂停

- 文件是二进制对象 blob
- 继承对象前端可以切片 实现文件上传的断点续传
- buffer
- http：range 部分请求 可用于下载文件断点续传
    - https://blog.csdn.net/m0_62617719/article/details/128324865

## 如何判断一个文件的类型经过非法修改
- 在JavaScript中，可以通过读取文件的二进制数据并利用FileReader对象的readAsArrayBuffer方法来读取文件头部信息，然后根据文件类型的魔数（magic number）进行判断。
    - "D0CF11E0A1B11AE1": "application/wps-office.ppt",
    - "89504E470D0A1A0A": "image/png",
  - "6674797071742020": "video/quicktime",
  - "504B030414000600": "application/wps-office.xlsx",
  - "504B03040A000000": "application/wps-office.docx",
  - "3C68746D6C3E0A0A": "text/html",
  - "3C21646F63747970": "text/html",
  - "526172211A0700":   "application/vnd.rar",
  - "7F454C46":         "application/x-sharedlib", 
  - "504B0304":         "application/zip",
  - "464C5601":         "video/x-flv",
  - "3C737667":         "image/svg+xml",
  - "25504446":         "application/pdf",
  - "1A45DFA3":         "video/webm",
  - "FFD8":             "image/jpeg", // (.jpg, .jpeg, .jfif, .pjpeg, .pjp)
  - "4D5A":             "application/x-ms-dos-executable",

## docker

## 技术选型考虑哪些

- 考虑跨平台
- 人气如何 生态
- 扩展性
- 维护活跃度

## 内部组件库如何维护

- menorepo 把所有项目全部放到一个大项目里 全部可以互相引用
- 通过pnpm安装子项目中的各个组件然后测试打包发布


## 设计模式
- BV1KJ4m1V7zC
- 工厂模式
    - 比如各种提示框
- 单例模式
    - 比如全局缓存（oa办公系统里用的最多的就是保存user字段，将所有的用户信息放到这个字段中）
- 适配器模式
    - 比如父组件传入了时间，要在子组件内格式化并且回显，一般在computed中转换 就是适配器
    - 或者购物车等，从服务器或者列表中用户选择商品后，会有单价、数量、优惠券等信息，最终回显数量、总价、总配送费等
 - 外观模式。
    - 提供一个统一的接口隐藏内部逻辑，方便外部调用，如实现兼容多种浏览器的添加事件方法。
- 迭代器模式。
    - 提供一种方法顺序访问聚合对象中的元素，而不暴露其内部表示，如js的Array.prototype.forEach。

## 前端优化

- webpack可视化工具（webpack-bundle-analyzer）看各个模块的体积
    - https://b23.tv/GjDa3J6
- lodash -> lodash-es
    - https://blog.csdn.net/qq_39335404/article/details/129951010
- 移动端预加载
- 表单细节优化
    - https://b23.tv/DxhW7UI

## 涉及到图形的库 包体积过大 如何解决

- external cdn引入 不打包到主项目内 通过外链引入
- BV1BD421W7oS 打包体积的分析和优化

## vite和webpage

- vite是基于ESModules实现的，更加轻量，开发环境下打包速度更快，webpage是全量打包，因此比较慢

## vue3和vue2的对比

- 响应式 defineProps 和 proxy
- diff算法更新 静态节点提升

## map和对象的区别
- 

## 二次封装 localStorage 考虑哪些问题

- 存读JSON
- 存取map


## 怎么判断对象有环引用

- 深度遍历 如果是对象就放到数组里 然后查重

## nextTick原理

- 内部实现是用的promise实现，如果没有用MutationObserver，在没有用的是settimeout 参考事件循环几种任务队列

## 父子组件生命周期执行顺序
- 加载渲染阶段：在这个阶段，父组件的生命周期钩子会先于子组件执行。具体顺序为：父组件的beforeCreate、created、beforeMount，然后是子组件的beforeCreate、created、beforeMount，最后是子组件的mounted。这意味着，在子组件挂载到父组件之前，父组件已经完成了所有的生命周期钩子的执行。
- 更新阶段：当父子组件有数据传递时，会执行到更新阶段的顺序。在这个阶段，父组件和子组件的生命周期钩子会按照它们在代码中的顺序执行。具体顺序为：父组件的beforeUpdate，子组件的beforeUpdate，子组件的updated，最后是父组件的updated。
- 销毁阶段：在销毁阶段，父组件的生命周期钩子会先于子组件执行。具体顺序为：父组件的beforeDestroy，子组件的beforeDestroy，子组件的destroyed，最后是父组件的destroyed。
- 需要注意的是，如果子组件是异步组件，那么执行顺序会发生改变，会先执行完父组件的生命周期然后再执行子组件的生命周期。这是因为Vue的生命周期钩子是按照组件的嵌套层级顺序依次执行的，确保了在子组件挂载到父组件之前，父组件已经完成了所有的生命周期钩子的执行。
- 父组件mouted中是否能获取到子组件中的变量？

## Promise链式调用

- 为什么可以实现链式调用？每次返回值都是promise

## CDN如何配置

## 老项目精度丢失
- webpack抽象语法树统一替换精度缺失的公式

## 虚拟列表？和缺点

- 需要频繁的计算dom 消耗性能较大 可能导致白屏
- 如何解决：预加载



## git的rebase 和 merge

- 都是合并，merge会保存所有提交记录
- rebase不会，会使提交记录看起来更简洁
- 具体用哪个视项目情况而定

## git的reset 和 reverse

- reset + commitId soft模式和 hard
- reverse 生成新提交保存前面的提交


## git stash
- 可以理解为先将本地代码存入栈中，使用最多的场景是本地有bug未修完，需要拉去新代码
- BV1nv4y1R78d

## vue的自定义指令
- directives
    - 常用于组件/按钮等的统一的权限管理
    - https://blog.csdn.net/XH_jing/article/details/118547130

## vuex修改数据的步骤
- BV1YM411w7Zc p48
- Viwe dispatch 到 Action，commit 到 Mutations， Mutate 到 State，最终 Rander 到 Viwe
- 最简单或者说最常用的修改vuex状态的方式是直接在组件中commit一次更改到Mutations
- Action中常封装一些页面的公共逻辑
- 本质上是通过再生成一个响应式数据来实现的（vue2：new Vue（）/vue3：reactive（）），因此命名不能重复，2.6版本以上有一个更轻量级的方案：vue.observeable

## vuex的五个属性/方法
- https://blog.csdn.net/qq_44755705/article/details/105291699
- module BV1YM411w7Zc p52

## router的几种模式
- hash：兼容性最好 但是不利于SEO优化 不美观
- history 会向服务器请求资源
- abstrat 上述两个方法都不生效时使用 不常用
- BV1YM411w7Zc p46

## router的钩子函数
- BV1YM411w7Zc p45

## elementui 页面只有一个input框时有什么问题
- 回车会刷新页面，导致这个bug的根源原因是包了form标签，浏览器默认按回车时触发submit事件，由于没有配置submit的url，就会刷新页面，解决方法就是在原生submit事件中阻止一下冒泡，类似场景中，不用element也可以利用浏览器这一特性实现回车事件的快速改写（表单外套form标签，然后监听或者重写submit事件），原生环境下开发表单都推荐包一层form标签

## vue组件间传值
- 父子组件：props和$emit
- 直接用ref/refs取组件内的变量
- eventBus 又被称为事件总线
    - 使用eventBus有没有遇到过什么问题？出现该bug的原理是什么？如何解决？由于eventBus是独立于业务组件外的vue对象，当前组件销毁并不会触发eventBus自动销毁，所以如果未经过正确注销，多次访问调用eventBus的组件会导致发布的事件次数增多从而导致监听多次触发（根据访问次数累加），解决方法就是在调用组件注销时手动来销毁eventBus（调用bus.$emit的组件beforeDistroy时调用bus的off方法）
    - 还有个比较常见的问题是使用eventBus时要注意监听和发布（bus.$on和bus.$emit）方法时应在哪个钩子函数中，例如我在父组件中created里发布方法，子组件中是无法监听到第一次发布的方法的
        - 详情原理参考 [父子组件生命周期执行顺序](#父子组件生命周期执行顺序)
        - https://www.php.cn/faq/382432.html
        - https://www.jb51.net/article/264928.htm
