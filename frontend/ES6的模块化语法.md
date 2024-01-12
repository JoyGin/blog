ES6的模块化语法主要包括两个关键字：`export`和`import`。

1. `export`：用于导出模块，可以导出变量、函数、对象或类。

```javascript
// 导出变量
export const name = 'John';

// 导出函数
export function sayHello() {
  console.log('Hello');
}

// 导出对象
export const obj = { key: 'value' };

// 导出类
export class MyClass { }
```

你也可以在一个`export`语句中导出多个成员：

```javascript
const name = 'John';
function sayHello() {
  console.log('Hello');
}
export { name, sayHello };
```

还可以使用`export default`导出默认成员，每个模块只能有一个默认导出：

```javascript
export default function() {
  console.log('Hello');
}
```

2. `import`：用于导入模块，可以导入模块中导出的任何成员。

```javascript
// 导入单个成员
import { name } from './module';

// 导入多个成员
import { name, sayHello } from './module';

// 导入所有成员，放在一个对象中
import * as module from './module';

// 导入默认成员
import myFunc from './module';
```

注意，`import`语句是静态执行的，意味着你不能在运行时改变要导入的模块或成员。

这就是ES6模块化的基本语法，通过这种方式，你可以更好地组织和管理你的代码，避免全局作用域的污染，提高代码的可维护性。