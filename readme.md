1. **下载 Node.js**

   [Node.js — 在任何地方运行 JavaScript](https://nodejs.org/zh-cn)

    ```powershell
    PS C:\Windows\system32> node --version
    v24.16.0
    ```

2. **新建 blog 项目**

    ```
    mkdir blog && cd blog
    
    npm add -D vitepress
    ```

    > powershell 通常禁止执行`npm`脚本，在窗口中输入`cmd`切换到命令提示符再执行。
    
    初始化 vitepress，按照以下设置：

    ```
    npx vitepress init
    
    ┌  Welcome to VitePress!
    │
    ◇  Where should VitePress initialize the config?  //VitePress 应该在哪里初始化配置？
    │  ./docs
    │
    ◇  Site title:   //站点标题
    │  My Awesome Project
    │
    ◇  Site description:  //站点描述
    │  A VitePress Site
    │
    ◇  Theme:  //主题
    │  Default Theme
    │
    ◇  Use TypeScript for config and theme files?  //使用 TypeScript 来设置配置文件和主题文件？
    │  Yes
    │
    ◇  Add VitePress npm scripts to package.json?  //在 package.json 里加入 vitepress 的 npm 启动文件？
    │  Yes
    │
    ◇  Add a prefix for VitePress npm scripts?  //为 VitePress npm 脚本添加前缀？
    │  Yes
    │
    ◇  Prefix for VitePress npm scripts:  //输入前缀
    │  docs
    │
    └  Done! Now run pnpm run docs:dev and start writing.  //在控制台输入 npm run docs:dev 以运行 vitepress 。
    ```
    
    构建，随后打开 http://localhost:5173/ 看看项目是否正常运行。

    ```
    npm run docs:dev
    ```
    
3. **新建仓库**

    github 新建 public 仓库，不要添加 readme 或是其他东西。创建完成后，在仓库首页启用 github action：`Settings > Pages > Build and deployment > GitHub Actions` 

    回到本地的 blog 项目中：

    ```
    git init 
    git branch -M main
    git remote add orgin <your github repo url>
    ```

4. **对项目设置 github action**

   修改`docs\.vitepress\.config.mts`的内容：

   - 在第二行添加：

     ```
     const base = "/blog-demo/";      // blog-demo 改成你仓库的名字
     ```

   2. 在此处再添加内容：

      ```
      export default defineConfig({
        base, //添加这一行
        title: "My Awesome Project",
        description: "A VitePress Site",
        themeConfig: {
          // https://vitepress.dev/reference/default-theme-config
          nav: [
            { text: 'Home', link: '/' },
            { text: 'Examples', link: '/markdown-examples' }
          ],
      ```

   创建`.github\workflows\depoly.yml`，并添加以下内容：

   ```yml
   name: Deploy Pages
   
   # 触发条件，push到main分支或者pull request到main分支
   on:
     push:
       branches: [main]
     pull_request:
       branches: [main]
   
     # 支持手动在工作流上触发
     workflow_dispatch:
   
   # 设置时区
   env:
     TZ: Asia/Shanghai
   
   # 权限设置
   permissions:
     # 允许读取仓库内容的权限。
     contents: read
     # 允许写入 GitHub Pages 的权限。
     pages: write
     # 允许写入 id-token 的权限。
     id-token: write
   
   # 并发控制配置
   concurrency:
     group: pages
     cancel-in-progress: false
   
   # 定义执行任务
   jobs:
     # 构建任务
     build:
   
       runs-on: ubuntu-latest
   
       steps:
         # 拉取代码
         - name: Checkout
           uses: actions/checkout@v3
           with:
             # 保留 Git 信息
             fetch-depth: 0
   
         # 设置使用 Node.js 版本
         - name: Setup Node
           uses: actions/setup-node@v3
           with:
             node-version: 'lts/*'
   
         # 安装依赖
         - name: Install dependencies
           run: npm install
   
         # 构建项目
         - name: Build blog project
           run: |
             echo ${{ github.workspace }}
             npm run docs:build
   
         # 资源拷贝
         - name: Build with Jekyll
           uses: actions/jekyll-build-pages@v1
           with:
             source: ./docs/.vitepress/dist
             destination: ./_site
   
         # 上传 _site 的资源，用于后续部署
         - name: Upload artifact
           uses: actions/upload-pages-artifact@v3
   
     # 部署任务
     deploy:
       environment:
         name: github-pages
         url: ${{ steps.deployment.outputs.page_url }}
       runs-on: ubuntu-latest
       needs: build
       steps:
         - name: Deploy to GitHub Pages
           id: deployment
           uses: actions/deploy-pages@v4
   ```

   创建`.gitignore`，并添加以下内容：

   ```
   node_modules
   dist
   cache
   .temp
   .DS_Store
   ```

5. **提交到 github**

   ```
   git add .
   git commit -m "first commit"
   git push origin main
   ```

   然后就可以在 github 中查看 action 的执行情况，完成后点击 [My Awesome Project](https://alittledian.github.io/blog-demo/) 查看。

> 参考链接
>
> - [【水贴 && 教程】如何用 vitepress + Github Pages 部署你的项目文档 - 闲聊吹水 - Koishi Forum](https://forum.koishi.xyz/t/topic/11125)
