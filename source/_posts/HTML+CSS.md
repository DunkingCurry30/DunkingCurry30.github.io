---

title: HTML + CSS 基础

date: 2022/6/20 20:00:00

tags: 

 - html

 - css

categories: 

 - Web前端

---

# HTML 基础

> Hyper Text Markup Language 是一种标记语言，通过标记队内容进行描述，从而提供超出普通文本的信息。



## HTML 组成

包含三部分：

- 文本内容
- 引用（图片、音视频、样式表、JS文件以及其他HTML文件等）
- 标记（预先定义好的、描述语义的标签）

**标记只代表语义**

HTML元素用于描述当前文本的语义，与显示样式无关，例如

```html
<p>说明当前是一个段落（paragraph）</p>
<a>说明当前是一个链接，或者叫锚点（anchor）</a>
```



## DOM 树

HTML在客户端被浏览器组织成一个树形结构，就是 DOM(Document Object Mode) tree，浏览器对网页的显示、CSS的样式和元素选择、JS的DOM节点操作都是以DOM tree为基础的。

### 树形结构中的关系

- 后代（ancestor and descendent）
- 父子（parent and child）
- 兄弟（siblings）



## 文档流

浏览器在渲染HTML时将其看做是一个个节点组成的流，按照从上到下，从左到右的顺序依次展示。



## 盒模型

将元素所占的空间类比成生活中的盒子（BOX），从外到内依次是 `margin` 、`border`和`padding`

















































































