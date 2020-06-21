---
title: vue_2.6.11版本源码运行以及报错处理
date: 2020-04-02
---

1. 克隆源码

   ```
   git clone https://github.com/vuejs/vue.git
   ```

2. 安装依赖

   ```
   npm install   
   ```

3. 运行

   ```
   npm run dev
   ```

   这个时候发现如下报错信息：

   ![](/img/blog/vue_debug/run_error.png)

   天呐！最怕见到报错了，麻溜去定位原因，原来是因为rollup-plugin-alias这个包在windows环境下路径为/而未换成\

4. 解决报错

   [**下载地址： https://github.com/ideayuye/rollup-plugin-alias**]

   下载以上地址版本的rollup-plugin-alias并覆盖原文件。npm下载方式我没有成功，所以简单粗暴从github下载后把文件替换掉的。

   （网上解决方案基本为下载该版本或修改rollup-pulgin-alias.js文件以及生成map文件，只有上述方式成功解决了问题，赶紧记录下来）

   ```javascript
   //进入rollup-plugin-alias目录
   cd C:\Users\Grace\Desktop\测试项目\vue\node_modules\rollup-plugin-alias   
   //安装依赖
   npm install
   //打包
   npm run build
   ```

5. 重新运行

   ```js
   //切换回到源码文件夹目录
   npm run dev
   ```

   ![](/img/blog/vue_debug/run_true.png)

   此时就会发现问题已经解决，接下来就可以做源码调试了！

6. 源码调试

   新建index.html引入dist/vue.js就可以在浏览器debug了
   
   ```js
   <script src="./dist/vue.js"></script>
   ```
   
   截图：
   
   ![](/img/blog/vue_debug/debug1.png)
   
   ![](/img/blog/vue_debug/debug2.png)
   
   这样调试也是很方便的，不过vue.js文件是rollup生成的。如果能够对vue项目中src目录里的文件打断点调试就更好了！那怎样去做呢？
   
   很简单！
   
   修改vue文件下的package.json：添加--sourcemap
   
   ```
   "dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev --sourcemap"
   ```
   
   修改后重新npm run dev，就可以看到dist目录下生成了vue.js.map文件，这个时候浏览器debug就可以在src目录里的文件打断点调试了。
   
   ![](/img/blog/vue_debug/debug3.png)