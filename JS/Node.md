# npm scripts

npm 允许在package.json文件里面，使用scripts字段定义脚本命令. 定义在package.json里面的脚本，就称为 npm 脚本.

```
{
  // ...
  "scripts": {
    "build": "node build.js"
  }
}
// scripts字段是一个对象。它的每一个属性，对应一段脚本. 
```

执行 `npm run build` 等同于 `node build.js`

**npm script优点:**

* 项目的相关脚本，可以集中在一个地方。
* 不同项目的脚本命令，只要功能相同，就可以有同样的对外接口。用户不需要知道怎么测试你的项目，只要运行npm run test即可。
* 可以利用 npm 提供的很多辅助功能。

**其他相关细节: [npm scripts 使用指南](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)**



