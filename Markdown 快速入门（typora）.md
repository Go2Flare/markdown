# Markdown 快速入门（typora）

## 代码块:

```go
//代码块语法
```java
```go
```shell
```

**1.go代码**

```go
func main() {
	router := gin.Default()
	// 匹配的url格式:  ?firstname=Jane&lastname=Doe
	router.GET("/", func(c *gin.Context) {
		firstname := c.DefaultQuery("firstname", "Guest")
		lastname := c.Query("lastname") // 是 c.Request.URL.Query().Get("lastname") 的简写

		c.String(http.StatusOK, "          H  EI  H EI  YO   YO YO      , Welcome %s %s", firstname, lastname)
	})
	router.Run(":80")
}
```

**2.shell脚本**

```shell
//运行go项目
go run main.go
```

## 标题：

```go
//标题语法
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

## 插入列表

1. 无序列表：‘-’+空格
2. 有序列表：‘1.’+空格

## 插入任务

-  ：‘-’+空格+[‘空格’]+空格
-  ：或者通过工具栏插入

##  缩进（嵌套）

- Tab：缩进/进入下一级嵌套
- Shift+Tab：反缩进/返回上一级嵌套

## 字形设置

- 加粗：**Ctrl+B**
- 斜体：*Ctrl+I*
- 高亮：无快捷键，使用双等号
- 下划线：Ctrl+U
- 删除线：Alt+Shift+5（或者一对波浪线）
- 超链接：Ctrl+K 百度学术

> 区块引用：>+空格

> 111
>
> > 222

## 缩放图片

- 方式1：在插图图片时/对着图片右击，选择缩放，即更改样式为：`style="zoom: 67%;"`

> 注：某些博客不支持这种缩放方式，以及typora的自动居中方式

- 方式2：将 `style="zoom: 67%;"` 改为 `width="600" align=center` ，图片将按宽度等比例缩放，并居中

> 注：可将这串代码在输入法里设置为自定义短语，快速调用

## 插入代码

- 代码串：Ctrl+Shift+~ 或者反引号：`Hello typora`

- 代码块：Ctrl+Shift+K，然后选择语言

  ```
  import numpy as np
  ```

## 表格

- 快捷键：Ctrl+T，选择行列数，可根据变化调节
- **表内换行**：Shift+Enter，此操作同Excel、微信聊天框换行

## 其它（略）

- 特殊符号
- 脚注
- 上下标
- 公式等等~~

# 样式更改

## 在大纲中自动编号多级目录

文件 -> 偏好设置 -> 外观 -> 打开主题文件夹

![image-20220602175546324](http://myimg.go2flare.xyz/img/image-20220602175546324.png)

新建样式表：base.user.css，并用txt打开

![image-20220602175812325](http://myimg.go2flare.xyz/img/image-20220602175812325.png)

复制以下代码，保存

```
/* 正文标题区: #write */
/* [TOC]目录树区: .md-toc-content */
/* 侧边栏的目录大纲区: .sidebar-content */
 
/** initialize css counter */
/* counter-reset： 从几级标题开始递增*/
#write, .sidebar-content,.md-toc-content{
    counter-reset: h1
}
 
#write h1, .outline-h1, .md-toc-item.md-toc-h1 {
    counter-reset: h2
}
 
#write h2, .outline-h2, .md-toc-item.md-toc-h2 {
    counter-reset: h3
}
 
#write h3, .outline-h3, .md-toc-item.md-toc-h3 {
    counter-reset: h4
}
 
#write h4, .outline-h4, .md-toc-item.md-toc-h4 {
    counter-reset: h5
}
 
#write h5, .outline-h5, .md-toc-item.md-toc-h5 {
    counter-reset: h6
}
 
 
/** put counter result into headings */
/* 一级标题不展示 */
#write h1:before,
.outline-h1>.outline-item>.outline-label:before,
.md-toc-item.md-toc-h1>.md-toc-inner:before {
    counter-increment: h1;
    content: counter(h1) "   ";
}
 
/* 二级标题 */
#write h2:before,
.outline-h2>.outline-item>.outline-label:before,
.md-toc-item.md-toc-h2>.md-toc-inner:before {
    counter-increment: h2;
    content: counter(h1) "." counter(h2) "   ";
}
 
/* 三级标题 */
#write h3:before,
h3.md-focus.md-heading:before, /** override the default style for focused headings */
.outline-h3>.outline-item>.outline-label:before,
.md-toc-item.md-toc-h3>.md-toc-inner:before {
    text-decoration: none;
    counter-increment: h3;
    content: counter(h1) "." counter(h2) "." counter(h3) "   ";
}
 
/* 四级标题 */
#write h4:before,
h4.md-focus.md-heading:before,
.outline-h4>.outline-item>.outline-label:before,
.md-toc-item.md-toc-h4>.md-toc-inner:before {
    text-decoration: none;
    counter-increment: h4;
    content: counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) "   ";
}
 
/* 五级标题 */
#write h5:before,
h5.md-focus.md-heading:before,
.outline-h5>.outline-item>.outline-label:before,
.md-toc-item.md-toc-h5>.md-toc-inner:before {
    text-decoration: none;
    counter-increment: h5;
    content: counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) "   ";
}
 
/* 六级标题 */
#write h6:before,
h6.md-focus.md-heading:before,
.outline-h6>.outline-item>.outline-label:before,
.md-toc-item.md-toc-h6>.md-toc-inner:before {
    text-decoration: none;
    counter-increment: h6;
    content: counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) "." counter(h6) "   ";
}
 
/** override the default style for focused headings */
#write>h3.md-focus:before,
#write>h4.md-focus:before,
#write>h5.md-focus:before,
#write>h6.md-focus:before,
h3.md-focus:before,
h4.md-focus:before,
h5.md-focus:before,
h6.md-focus:before {
    color: inherit;
    border: inherit;
    border-radius: inherit;
    position: inherit;
    left:initial;
    float: none;
    top:initial;
    font-size: inherit;
    padding-left: inherit;
    padding-right: inherit;
    vertical-align: inherit;
    font-weight: inherit;
    line-height: inherit;
}
 
/* 设置行距 */
 
 
/* 设置一级标题行距 */
#write h1 {
    margin-bottom:50px;
    margin-top:50px;
}
 
#write h2 {
    margin-bottom:30px;
    margin-top:50px;
}
 
#write h3 {
    margin-bottom:30px;
    margin-top:30px;
}
 
#write h4 {
    margin-bottom:30px;
    margin-top:30px;
}
 
#write h5 {
    margin-bottom:30px;
    margin-top:30px;
}
```

重启typora后，效果如下
