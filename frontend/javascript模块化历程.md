1. 无模块化时代：在早期的JavaScript中，所有的代码都是全局的。如果两个JavaScript文件需要互相调用，那么他们的函数和变量都需要定义在全局作用域中。

```javascript
// file1.js
var globalVar = "I'm a global variable";

// file2.js
console.log(globalVar); // "I'm a global variable"
```

2. 命名空间时代：为了解决全局作用域污染的问题，开发者开始使用命名空间的方式来组织代码。也就是说，将相关的函数和变量放在一个全局对象下。

```javascript
// file1.js
var MyApp = MyApp || {};
MyApp.globalVar = "I'm a global variable";

// file2.js
console.log(MyApp.globalVar); // "I'm a global variable"
```

3. CommonJS和AMD时代：Node.js采用的是CommonJS规范，每个文件就是一个模块，通过module.exports和require来导出和导入模块。而在浏览器端，AMD规范采用异步加载的方式来加载模块。

```javascript
// CommonJS
// file1.js
module.exports = "I'm a module";

// file2.js
var module1 = require('./file1');
console.log(module1); // "I'm a module"

// AMD
// file1.js
define([], function() {
  return "I'm a module";
});

// file2.js
require(['./file1'], function(module1) {
  console.log(module1); // "I'm a module"
});
```

4. ES6模块时代：在ES6中，JavaScript正式引入了模块化的语法。通过export和import关键字来导出和导入模块。

```javascript
// file1.js
export default "I'm a module";

// file2.js
import module1 from './file1';
console.log(module1); // "I'm a module"
```

5. 现代模块打包工具时代：在实际开发中，由于需要兼容不同的环境和浏览器，因此出现了很多模块打包工具，如webpack、rollup等。这些工具可以将使用不同模块化规范的代码打包成一个文件。

```javascript
// webpack.config.js
module.exports = {
  entry: './file2.js',
  output: {
    filename: 'bundle.js'
  }
};

// file1.js (ES6 module)
export default "I'm a module";

// file2.js
import module1 from './file1';
console.log(module1); // "I'm a module"
```

在这个过程中，JavaScript的组织和管理方式也越来越规范和高效。