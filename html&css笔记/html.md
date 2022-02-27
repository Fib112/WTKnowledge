### 一、表格table

- 表头单元格`<th>`，通常用在第一行，文字会加粗居中显示

- 表格属性

  1. `align`，表格在文档中显示的位置，可选值有：`left`、`center`、`right`
  2. `border`，边框像素，默认是""，单位是`px`，属性值是数字，不需要写单位
  3. `cellpadding`，文字离单元格边界的距离，默认值是1，单位是`px`，属性值是数字，不需要写单位
  4. `cellspacing`，单元格与单元格之间的距离，默认值是2，单位是`px`，属性值是数字，不需要写单位
  5. `width/height`，单元格宽度/高度，属性值是像素或百分比

- 区域划分

  为了更好地表示表格的语义，可以将表格划分为表格头部和表格主体两大区域，分别用`<thead>`和`<tbody>`来划分，在`<thead>`中必须有`<tr>`

- 合并单元格

  1. 跨行合并：`rowspan="count"`，count为合并单元格的个数，写在最上侧单元格中
  2. 跨列合并：`colspan="count"`，写在最左侧单元格中
  3. 最后还要删除多余的单元格

### 二、列表

#### 2.1 无序列表

`<ul>`中只能嵌套`<li>`，`<li>`相当于一个容器，可以容纳所有元素

#### 2.2 自定义列表

基本语法：

```html
<dl>
    <dt>标题</dt>
    <dd>子项一</dd>
    <dd>子项二</dd>
</dl>
```

限制：

- `<dl>`里只能包含`<dt>`和`<dd>`
- `<dt>`和`<dd>`个数没有限制，经常是一个`<dt>`对应多个`<dd>`
- `<dt>`和`<dd>`可以放任何标签

### 三、表单

在HTML中，一个完整的表单由表单域、表单控件和提示信息组成

#### 3.1 表单域

`form`标签用于定义表单域，以实现用户信息的传递和收集

```html
<form action="url" method="get/post" name="myForm" enctype="multipart/form-data">
    ...
</form>
```

当`method`为`get`时，会将表单的数据作为`query string`传到服务器，`post`则是通过`body`

#### 3.2 表单控件

- 文本框和密码框

  ```html
  <input type="text">
  <input type="password">
  ```

- 单选按钮和复选框

  ```html
  <input type="radio"> // 单选按钮
  <input type="checkbox"> // 复选框
  ```

  多个单选按钮必须使用相同的`name`才能实现单选效果，多个复选框也需要相同的`name`才能实现多选

- `<input>`标签属性

  1. `name`：`input`元素名
  2. `value`：`input`元素值
  3. `checked`：此元素首次加载时为选中状态
  4. `maxlength`：输入字段中字符最大长度

- 提交按钮和重置按钮

  ```html
  <input type="submit" value="提交">
  <input type="reset" value="重置">
  ```

  提交按钮会将表单中的信息提交到服务器，重置按钮则会将整个表单中的元素重置为初次加载时的状态。提交按钮和重置按钮的`value`会显示在按钮中

- 普通按钮与文件上传按钮

  ```html
  <input type="button">
  <input type="file">
  ```

- `label`标签

  该标签用于绑定一个表单元素，当点击`<label>`中的内容时，会自动将焦点转到对应表单元素上

  ```html
  <label for="male">男</label> // for属性填写目标标签的id
  <input type="radio" id="male" name="sex" value="male">
  ```

- `select`下拉表单元素

  ```html
  <select>
      <option value="China" selected>中国</option> // 默认选中项
      <option value="Japan">日本</option>
  </select>
  ```

- 文本域标签

  ```html
  <textarea rows="3" col="20">
  	文本内容
  </textarea>
  ```

  cols属性用于定义每行中的字符数，rows属性设置显示的行数