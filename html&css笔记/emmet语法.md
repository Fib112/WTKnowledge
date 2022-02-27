### 一、class、id

生成指定class/id的标签：`div.red`、`span.red`、`p#red`、`h1#blue`

### 二、子节点、兄弟节点、上级节点

- 子节点：`div>span`

- 兄弟节点：`div+span`

- 上级节点：`div>span>ul^div`

  ```html
  <div>
      <span>
          <ul>
              
          </ul>
      </span>
      <div>
          
      </div>
  </div>
  ```

  `ul`标签的上一级是`span`标签，所以`div`加入`ul`的上一级成为`span`的兄弟节点

### 三、重复与分组

- 重复：`div*5`

- 分组：`div>(ul>li>a)+span>ul`

  括号内的内容成为一组与外部层级结构隔离，效果如下：

  ```html
  <div>
      <ul>
          <li><a></a></li>
      </ul>
      <span>
          <ul></ul>
      </span>
  </div>
  ```

### 四、属性

属性指令：`[]`

如：`a[href="3" id="link"]`

### 五、编号

编号指令：`$`

如：`ul>li.test$*3`

```html
<ul>
    <li class="test1"></li>
    <li class="test2"></li>
    <li class="test3"></li>
</ul>
```

- 一个$ 代表一位数，`$$`代表两位数(01)，以此类推......

- 如果想自定义从几开始递增的话就利用：`$@+数字*数字`
  例如：`ul>li.test$@3*3`
  
  ```html
  <ul>
      <li class="test3"></li>
      <li class="test4"></li>
      <li class="test5"></li>
  </ul>
  ```

### 六、文本

文本指令：`{}`

如：`div{test$}*3`

```html
<div>
    test1
</div>
<div>
    test2
</div>
<div>
    test3
</div>
```

