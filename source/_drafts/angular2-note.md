title: Angular2初试小记
tags: [angular2, angular-cli, typescript, js]
categories: programming
---

最近一个网站尝试用[Angular2](https://angular.io)，截止现在，angular2最新版本为rc.6，还在快速更新中。

## Angular 1 vs 2

Angular2对比1，做了很大的调整，代码更加组件化，结构会更清晰，通过[官方教程](https://angular.io/docs/ts/latest/quickstart.html)很快能入手。

同时使用Typescript作为开发语言，支持静态类型检查、js新标准等优点，截止现在使用2.0.2版本的ts。

更多介绍参考[官方](https://angular.io/features.html)

## Angular-cli

[Angular-cli](https://github.com/angular/angular-cli)是官方维护的很强大的工具，能帮助我们以最佳实践构建项目，最新版现在以webpack为构建工具。

## 生产环境

- Nginx部署

    需要把请求都指向`index.html`，从而使用angular内部的路由。

    ```
    server {
      listen 80;
      listen [::]:80;

      root /path/to/project;

      index index.html index.htm;

      server_name www.project.com;

      error_page 404 =200 /index.html;

      location / {
        try_files $uri $uri/ =404;
      }
    }
    ```

- Pre-bootstrap

    angular2有个比较诟病的地方，是体积略大。即使是新建的项目，编译出来的main.js也有1M多。这使得第一次访问、浏览器没有缓存的时候，首屏时间会比较糟糕。原因还是angular2的库即使压缩后还是太大了。所以建议还是在`index.html`增加Loading的动画，提高用户体验。

## 参考资料
