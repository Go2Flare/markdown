# 基础操作

## 快捷键



# 高级查询

## 查询所有文献数字标号

```
查找一位数字
\[[0-9]\]
查找两位数字
\[[0-9][0-9]\]
```

替换为上标，只需在**替换框**使用快捷键ctrl+shift+=

# 文献标号

创建文献编号>>插入交叉引用>>自动更新域。

**步骤一：定义参考文献的编号格式。**

1、将每一条参考文献按段落格式排版，并选中之。

2、打开选项卡：“开始>>有序编号>>定义新编号格式（D）”。

3、在编号格式中，输入“[”和“]”，然后，在中间插入编号“1，2，3…”，对齐方式为左对齐，预览无误后确定。![img](https://pic2.zhimg.com/80/v2-7a15077127e06c7de7cc0e0714ce7b41_720w.jpg)

添加编号后的效果如下图所示![img](https://pic4.zhimg.com/80/v2-767427ff9c036a336dc8471851a5f35b_720w.jpg)

**步骤二：插入交叉引用，并插入论文对应位置。**

1、路径：在Word选项卡上，“插入>>交叉引用”。

2、目的：引用文档中设定的参考文献编号项。

3、方法：

〔1〕引用类型>>编号项，引用内容>>段落编号；

〔2〕选择需引用的编号项，并插入到论文对应位置。

〔3〕右键选择上标（快捷键：ctrl+shift+=)

![img](https://pic3.zhimg.com/80/v2-c83727d8426fbd9b856bae49767ec482_720w.jpg)

**步骤三：自动更新域，使得对应的文献编号同步变化。**

当参考文献列表顺序发生变化时：

1、在论文中，选中引用的“编号项”，如编号项[4]；

2、选中后，右键>>更新域（U），则对应的编号项将跟随参考文献列表变化。![img](https://pic2.zhimg.com/80/v2-b58206963ff83e76d8ade197d62ebe01_720w.jpg)

# 插入题注

## 修改题注格式

开始->选择题注样式修改即可

## 批量插入图片题注

链接：https://www.zhihu.com/question/34242477/answer/133795594

在word中批量给图片加题注及编号,方便大量图片插入的文档更新时图片变动时可自动更新域来保持题注编号正确。

1.右击某张[图片](https://www.zhihu.com/search?q=图片&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A133795594})，选择插入题注。

2.可选择标签,也可以点击按钮"新建标签",自定义"标签"例如输入“图”。

3.点击编号，弹出题注编号设置框,勾选包含章节号，一定记得设置章节起始样式。

点击确定，题注插入完成。此处注意只插入一张图片的题注，

浏览查看，图片已依照一级标题章节名称命名，如下图形如：2-N。

![img](https://pica.zhimg.com/50/v2-65e9b3c1225714a70f325a4dca4cfa78_720w.jpg?source=1940ef5c)



4.批量给全文中图片加题注，右击该题注，选择更换域代码（或者按快捷键ALT+F9），切换到域代码状态。

![img](https://pic2.zhimg.com/50/v2-034186baa848d754745ee6d1a11b60b6_720w.jpg?source=1940ef5c)



5. （ctrl+C）复制这个域代码，包括域代码的大括号。注意只复制除去自己填写的图片名称外的域代码，然后打开查找替换框，查找处输入^g，替换处输入^&^p^c全部替换。^g是每一个图片，^&是搜索的原对象。^g替换成^&表示N个图都保持原样不替换。^p是换行，表示题注位于图片的下一行，^c表示复制的内容。替换之后ctrl+c选中全文，右键，选择更新域，即可。

![img](https://pic2.zhimg.com/50/v2-6a312208ae7a1fa3ece0673e5f06238c_720w.jpg?source=1940ef5c)



6. CTRL+A全选，按F9刷新，然后按ALT+F9切换回域值状态。

可以说，插入题注和交叉引用，都是为最后一步的”Ctrl+A——>F9——>更新整个目录——>自动修改题注“服务的。按上面的方法插入题注、交叉引用后，再也不用担心因增加、删除图表而导致的序号不一致的问题了。至此设置完成，全文图片都自动添加了题注，按需求再对图片题注进行设置或修改。

7.后续某张图片修改了，则右击文中图片的题注，点击更新域。文中题注自动更新完成。

## 插入表格自动题注

自动插入题注

如果文档中图片或表格太多的话，一个一个插入题注的话，肯定会很费时间，这时候就需要设置“自动插入题注”功能了！

这里图片和表格有些不同，先介绍一下表格自动插入题注！

**1.表格自动插入题注**

依次单击【**引用**】-【**插入题注**】-【**自动插入题注**】-选择Microsoft Word 表格，单击确定即可。

动图演示如下：

![图片](https://mmbiz.qpic.cn/mmbiz_gif/RjMicHwEmBjeibsVtrO0QwYEhATcKk8UfF3gOKuF0Rs0gw3FdZUBtupV3yVyo9fVliayibG2k0CibesLWCX8IDXE9eA/640?wxfrom=5&wx_lazy=1)

**2.为图片自动插入题注**

先按照前文的方法，为第一张图片插入题注。如图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_gif/RjMicHwEmBjeibsVtrO0QwYEhATcKk8UfF8DJceDMZ56MpZ5aiasayXHRKhPX8Do8KjOUMHOQiamAJTCqrdc6ZObWg/640?wxfrom=5&wx_lazy=1)

然后，选中题注（图1-1）并右键单击，选择切换域代码。（或者快捷键**Alt+F9**），然后复制域代码（**Ctrl+C**）。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/RjMicHwEmBjeibsVtrO0QwYEhATcKk8UfF8fUrFHwRyJvAO0Ew8dicOFWMRuHuDqCIz3K0Eia9GnlDU5tUyJFh0Ttw/640?wxfrom=5&wx_lazy=1)

再打开替换窗口（**Ctrl+H**），在查找处输入**^g**，在替换为处输入**^&^p^c**，选择全部替换即可，最后全文选中（**Ctrl+A**），右键单击“更新域”（**F9**）即可完成。

**动图演示如下：**

![图片](https://mmbiz.qpic.cn/mmbiz_gif/RjMicHwEmBjeibsVtrO0QwYEhATcKk8UfFydTSoYOmA1moUNZibPWa2Jn2UBFdyatVLMhOgeSgNkHsUJXjpjnUbhw/640?wxfrom=5&wx_lazy=1)

6更新题注

当我们增加或删除题注的时候，题注编号就不连续了，这时候就需要更新题注了。

全选文档（**Ctrl+A**），右键单击，选择更新域即可，快捷键**F9**。

# 常见问题

## 图片显示不全

法一：在图片的段落中选择多倍行距，一般是图片的行距设置出错

法二：选择图片，点击鼠标右键，弹出下拉菜单，选择“文字环绕（W）”，取消“嵌入式”，选择其它方式，任何一种都可以，如图所示