# vue-music

> 基于vue全家桶开发音乐webapp

# 在线演示地址：
  https://tanrenjie.github.io/vue-music-webapp
  
  
  PC端建议使用chrome手机调试工具查看，移动端推荐使用最新版谷歌浏览器或微信自带浏览器
  
  PS：
    **因Gitpage只能上传静态网站，因此没有歌词（项目中用的nodejs获取歌词数据）
    
        API失效/BUG 请发邮件联系。
    
### 技术栈
- Vue2.3：采用最新Vue2.3的语法
- Vuex：管理组件之间状态
- vue-router：单页面应用路由
- jsonp：发起跨域http请求
- stylus：css预处理语言
- Express：解决api接口referer端口限制。
- Webpack：自动化构建工具，主要配置vue-cli脚手架提供。
- ES6：采用ES6语法。
- CSS3：CSS3动画及样式。
### 实现的功能
1. 音乐播放、暂停、切换、收藏
2. 搜索单曲、歌手
3. 搜索历史记录
4. 首页轮播图
5. 歌曲排行榜
6. 切换播放模式
7. 歌词跟随播放进度显示


## Build Setup

``` bash
# 安装项目依赖
npm install

# 启动项目，在浏览器输入localhost:8080访问
npm run dev

# 打包项目
npm run build

```

