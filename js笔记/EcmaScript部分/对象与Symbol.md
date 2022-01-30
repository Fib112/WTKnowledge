### 一、可选链?.

可选链用于访问对象中不确定存在的属性或方法，当?.前的值不是对象时返回undefined

用法：

- 访问属性：a?.name 或者 a?.['name']
- 访问方法：a.getName?.() 当不存在此方法时返回undefined
- 删除可能存在的属性：delete a?.name
- **可选链不能用于赋值**：如a?.name = 'abc'

### 二、Symbol类型

Symbol” 值表示唯一的标识符，可以使用 `Symbol()` 来创建这种类型的值，创建时，我们可以给 Symbol 一个描述（也称为 Symbol 名）。对象的属性键只能是字符串类型或者 Symbol 类型

- Symbol 不会被自动转换为字符串：如果需要将Symbol转化为字符串，需要显式调用toString()方法，如：`Symbol("id").toString()`会返回字符串`Symbol(id)`；或者获取 `description` 属性，只显示描述（description）：`Symbol("id").description`会返回字符串`id`

- 在对象中使用Symbol需要使用中括号：

  ```javascript
  let id = Symbol("id")
  let obj = {
      [id]: 1
  }
  console.log(obj[id])
  ```

- Symbol 属性不参与 `for..in` 循环，`Object.keys(user)` 也会忽略它们，但`Object.assign`会同时复制字符串和 symbol 属性

#### 全局Symbol

全局 Symbol 注册表可以确保每次访问相同名字的 Symbol 时，返回的都是相同的 Symbol

- 要从注册表中读取（不存在则创建）Symbol，使用 `Symbol.for(key)`

  ```javascript
  let id1 = Symbol.for("id")
  let id2 = Symbol.for("id")
  id1 === id2 //true
  id1 === Symbol("id") //false
  ```

- `Symbol.keyFor(sym)`，它的作用完全反过来：通过全局 Symbol 返回一个名字

  ```
  let id = Symbol.for("id")
  let name = Symbol.for("name")
  Symbol.keyFor(id) //"id"
  Symbol.keyFor(name) //"name"
  ```

  与Symbol.for一样，它不适用于非全局 Symbol

### 三、对象 — 原始值转换

所有的对象在布尔上下文（context）中均为 `true`。所以对于对象，不存在 to-boolean 转换，只有字符串和数值转换

- 数值转换发生在对象相减或应用数学函数时。例如，`Date` 对象可以相减，`date1 - date2` 的结果是两个日期之间的差值
- 字符串转换 —— 通常发生在像 `alert(obj)` 这样输出一个对象和类似的上下文中

在对象与原始值的转换中，有三种hint：

- string，对象到字符串的转换，当我们对期望一个字符串的对象执行操作时，如 `alert`：

  ```javascript
  let obj = {}
  let user = {}
  
  // alert输出
  alert(obj) //hint = string
  // 将对象作为属性键
  user[obj] //hint = string
  ```

- number，对象到数字的转换，例如当我们进行数学运算时：

  ```javascript
  let obj = {}
  
  // 显式转换
  let m = Number(obj) //hint = number
  // 数学运算（除了二元加法）
  let x = obj - 3 //hint = number
  let n = +obj //hint = number
  //小于/大于的比较
  obj > 1 //hint = number
  ```

- default，在少数情况下发生，当运算符“不确定”期望值的类型时。

  例如，二元加法 `+` 可用于字符串（连接），也可以用于数字（相加），所以字符串和数字这两种类型都可以。因此，当二元加法得到对象类型的参数时，它将依据 `"default"` hint 来对其进行转换。此外，如果对象被用于与字符串、数字或 symbol 进行 `==` 比较，这时到底应该进行哪种转换也不是很明确，因此使用 `"default"` hint。除了Date外，所有对象的default hint都与number hint的实现一致

**为了进行转换，JavaScript 尝试查找并调用三个对象方法：**

1. 调用 `obj[Symbol.toPrimitive](hint)` —— 带有 symbol 键 `Symbol.toPrimitive`（系统 symbol）的方法，如果这个方法存在的话，以下是toPrimitive实现

   ```javascript
   let obj = {
       name: 'aaa',
       age: 18,
       [Symbol.toPrimitive](hint) {
           alert(`hint: ${hint}`)
           return hint === 'string' ? 'aaa' : 18
       }
   }
   alert(obj) //'aaa'
   console.log(obj - 17) //18 - 17
   ```

   

2. `toString/valueOf`

   - 如果没有 `Symbol.toPrimitive`，那么 JavaScript 将尝试找到它们，并且按照下面的顺序进行尝试:对于 “string” hint，`toString -> valueOf`；其他情况，`valueOf -> toString`

   - 这些方法必须返回一个原始值。如果 `toString` 或 `valueOf` 返回了一个对象，那么返回值会被忽略（和这里没有方法的时候相同）

   - 默认情况下，普通对象具有 `toString` 和 `valueOf` 方法：

     - `toString` 方法返回一个字符串 `"[object Object]"`
     - `valueOf` 方法返回对象自身

   - 如果希望有一个“全能”的地方来处理所有原始转换。在这种情况下，可以只实现 `toString`：

     ```javascript
     let obj = {
         name: 'aaa',
         toString() {
             return this.name
         }
     }
     alert(obj) //'aaa'
     alert(obj + 'bbb') //'aaabbb'
     ```

返回类型：

关于所有原始转换方法，并没有限制返回值的类型。比如`toString`可以返回数字，hint为number时valueOf和toPrimitive可以返回字符串。唯一强制性的事情是：这些方法必须返回一个原始值，而不是对象。如果toPrimitive返回对象会抛出错误，而其他两个虽然不会抛出错误，但会静默忽略设定的方法

