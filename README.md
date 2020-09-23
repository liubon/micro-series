# micro-series 微前端系列

## 前端组成

- [micro-main](https://github.com/liubon/micro-main) vue 主应用
- [micro-app-001](https://github.com/liubon/micro-app-001) vue 微应用
- [micro-app-002](https://github.com/liubon/micro-app-002) react 微应用
- [micro-app-003](https://github.com/liubon/micro-app-003) jQuery 微应用

## 部署

1.采用 docker 搭建 jenkins、mysql、nginx 环境，将 jenkins 打包后的前端项目放置在`home/webserver/static`,使用 nginx 代理访问

2.通过云函数实现弹性查询注册微应用

```js
'use strict';

function wrapPromise(connection, sql) {
  return new Promise((res, rej) => {
    connection.query(sql, function (error, results, fields) {
      if (error) {
        rej(error);
      }
      res(results);
    });
  });
}
exports.main_handler = async (event, context, callback) => {
  const mysql = require('mysql');
  var connection = mysql.createConnection({
    host: 'yourhost',
    user: 'userName',
    password: 'yourpassword',
    database: 'database',
  });

  connection.connect();

  const querySql = `SELECT * FROM micro_app`;
  let queryResult = await wrapPromise(connection, querySql);
  connection.end();
  return queryResult;
};
```

3.如果不需要 jenkins、和 mysql，直接将打包构建好的项目放置在`home/webserver/static`下，在主应用中配置好加载即可

## docker 配置目录

```
home
├── compose
│   └── docker-compose.yml // docker-compose 配置文件
├── jenkins
│   └── jenkins_home       // 挂载/var/jenkins_home
├── mysql
│   ├── conf               // 挂载/etc/my.cnf
│   │   └── my.cnf
│   ├── log                // 挂载/var/log/mysql
│   └── db                 // 挂载/var/lib/mysql
├── nginx
│   ├── cert               // 挂载/etc/nginx/cert,用于存放ssl证书
│   │   ├── your.crt
│   │   └── your.key
│   └── conf.d             // 挂载/etc/nginx/conf.d
│   │   ├── default.conf   // https 使用该规则
│   │   └── nginx.conf     // http 使用该规则
├── webserver
│   ├── static             // 前端工程
```
