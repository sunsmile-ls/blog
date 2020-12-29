## react 常用配置的解释

```javascript
// jest.config.js
module.exports = {
  // jest 的根目录
  roots: ['<rootDir>/src'],
  // 代码覆盖率是通过分析哪些文件生成的
  collectCoverageFrom: ['src/**/*.{js,jsx,ts,tsx}', '!src/**/*.d.ts'],
  // 运行测试之前，额外的准备
  setupFiles: ['react-app-polyfill/jsdom'],
  // 测试环境建立好之后，执行文件的内容
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],
  // 需要测试的文件
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}',
    '<rootDir>/src/**/*.{spec,test}.{js,jsx,ts,tsx}',
  ],
  // 测试环境： 表示在node 的环境中模拟浏览器 上的 dom 的 api 或者方法
  testEnvironment: 'jest-environment-jsdom-fourteen',
  transform: {
    // 符合前面的正则的会用后面的处理器处理，然后再测试
    '^.+\\.(js|jsx|ts|tsx)$': '<rootDir>/node_modules/babel-jest',
    '^.+\\.css$': '<rootDir>/config/jest/cssTransform.js',
    '^(?!.*\\.(js|jsx|ts|tsx|css|json)$)':
      '<rootDir>/config/jest/fileTransform.js',
  },
  // 忽略转换
  transformIgnorePatterns: [
    '[/\\\\]node_modules[/\\\\].+\\.(js|jsx|ts|tsx)$',
    '^.+\\.module\\.(css|sass|scss)$',
  ],
  // 如果在 node_modules 之外， 默认要查找的目录
  modulePaths: [],
  moduleNameMapper: {
    '^react-native$': 'react-native-web',
    '^.+\\.module\\.(css|sass|scss)$': 'identity-obj-proxy', // 把原始css 转换为 键值对
  },
  moduleFileExtensions: [
    // 扩展名
    'web.js',
    'js',
    'web.ts',
    'ts',
    'web.tsx',
    'tsx',
    'json',
    'web.jsx',
    'jsx',
    'node',
  ],
  // 以下两个jest 插件 --watch 使用
  watchPlugins: [
    'jest-watch-typeahead/filename',
    'jest-watch-typeahead/testname',
  ],
}
```

> 更多配置请看 [jest config](https://jestjs.io/docs/en/configuration)

`enzyme` 配置请看 [github](https://github.com/enzymejs/enzyme)
