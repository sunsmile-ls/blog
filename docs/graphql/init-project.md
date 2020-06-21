## 初始化项目

> 项目技术栈简介

- egg.js
- graphql
- typescript
- react

## 安装 egg

```shell
$ npm init egg --type=ts
```

## 安装插件

基于 TypeScript 的，所以在这里我选择了一个支持 TypeScript 的包 @switchdog/egg-graphql 。

```shell
$ npm i --save @switchdog/egg-graphql
```

## 开启插件

在 `config/plugin.ts` 下告诉 egg 开启哪些插件：

```typescript
graphql: {
    enable: true,
    package: '@switchdog/egg-graphql',
},
```

## 配置插件

通常插件都会有一些配置项，在`/config/config.default.ts` 中配置即可：

```typescript
config.graphql = {
  router: '/graphql',
  // 是否加载到 app 上，默认开启
  app: true,
  // 是否加载到 agent 上，默认关闭
  agent: false,
  // 是否加载开发者工具 graphiql, 默认开启。路由同 router 字段。使用浏览器打开该可见。
  graphiql: true,
  apolloServerOptions: {
    tracing: true, // 设置为true时，以Apollo跟踪格式收集和公开跟踪数据
    debug: true, // 一个布尔值，如果发生执行错误，它将打印其他调试日志记录
  },
}
```

在中间件中开启 graphql

```typescript
config.middleware = ['graphql']
```

## GraphQL 代码结构

```
├── graphql                       | graphql 代码
│   ├── common                    | 通用类型定义
│   │   ├── resolver.js           | 合并所有全局类型定义
│   │   ├── scalars               | 自定义类型定义
│   │   │   └── date.js           | 日期类型实现
│   │   └── schema.graphql        | schema 定义
│   ├── mutation                  | 所有的更新
│   │   └── schema.graphql        | schema 定义
│   ├── query                     | 所有的查询
│   │   └── schema.graphql        | schema 定义
│   └── user                      | 用户业务
│       ├── connector.js          | 连接数据服务
│       ├── resolver.js           | 类型实现
│       └── schema.graphql        | schema 定义
```

## 安装下面插件

```shell
$ npm i -S  egg-cors egg-redis egg-sequelize egg-view-ejs graphql-upload lodash mysql2 nodemailer uuid


```

- 配置

```javascript
//config.default.ts

// 配置跨域
config.cors = {
  origin: '*',
  allowMethods: 'GET,HEAD,PUT,POST,DELETE,PATCH',
}

config.security = {
  csrf: {
    ignore: () => true,
  },
}

config.sequelize = {
  dialect: 'mysql', // support: mysql, mariadb, postgres, mssql
  host: '127.0.0.1',
  password: '123456',
  port: 3306,
  database: 'test', // /数据库名
}

config.github = {
  // 固定的
  login_url: 'https://github.com/login/oauth/authorize',
  // github Client ID
  client_id: 'xxxxxx',
  // github Client Secret
  client_secret: 'xxxxxxxxx',
  // 此参数表示只获取用户信息
  scope: ['user'],
}

config.view = {
  mapping: {
    '.html': 'ejs',
  },
}
```

- plugin

```javascript
// plugin.ts
  cors: {
    enable: true,
    package: 'egg-cors',
  },
  sequelize: {
    enable: true,
    package: 'egg-sequelize',
  },
  redis: {
    enable: true,
    package: 'egg-redis',
  },
  ejs: {
    enable: true,
    package: 'egg-view-ejs',
  },
```

## 初始化数据库

1. 安装[配置在上面]

```
$ npm install --save-dev sequelize-cli egg-sequelize-auto
```

2. 在根目录下面创建 `.sequelizerc`

```typescript
const path = require('path')

module.exports = {
  config: path.join(__dirname, 'database/config.json'),
  'migrations-path': path.join(__dirname, 'database/migrations'),
  'seeders-path': path.join(__dirname, 'database/seeders'),
  'models-path': path.join(__dirname, 'app/model'),
}
```

3. 初始化 Migrations 配置文件和目录

```shell
$ npx sequelize init:config
$ npx sequelize init:migrations
```

4. 添加 scripts

```json
  "db:model": "egg-sequelize-auto -o ./app/model -h 127.0.0.1 -p 3306 -d test -u root -x 123456",
  "db:init": "npx sequelize migration:generate --name=init-name",
  "db:up": "npx sequelize db:migrate && npm run db:model",
  "db:down": "npx sequelize db:migrate:undo",
  "db:down-all": "npx sequelize db:migrate:undo:all"
```
