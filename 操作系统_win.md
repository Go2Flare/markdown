# 删除账户

https://jingyan.baidu.com/article/546ae1853d51ab5048f28c66.html

1. 控制面板
2. 用户账户
3. 其他账户
4. 选择要删除的账户进入
5. 删除账户

# 设置FTP服务器

如果需要在内网中，其他内网中的主机可以与FTP服务器的主机交互

https://blog.csdn.net/SubStar/article/details/107365423



# （一）打开控制面板添加FTP服务

使用WIN+R快捷键，在弹出的窗口中输入【control】命令，打开控制面板

或者使用WIN + pauseBreak按钮打开

打开程序和功能：

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168643378-3657dd91-94dc-4fd3-8fe6-d20b93d506c3.png)选择启动或者关闭Windows功能![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168658353-a6229842-5526-4cef-a42e-c5395daf85f0.png)找到并勾选如下选项

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168667252-9ac6e0a0-da59-4039-88fd-477b2d4bd83b.png)

确定

再打开控制面板

使用WIN+R快捷键，在弹出的窗口中输入【control】命令，打开控制面板

或者使用WIN + pauseBreak按钮打开

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168680331-ff4ec69f-ee83-4897-b93c-b5bae2c58053.png)

找到并打开如下选项

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168691709-e3aa1450-22e6-42ec-a6b4-2d1309aff3bf.png)

打开后，选择添加ftp站点

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168701221-10f1e963-6423-4159-abe4-5ef4e9759442.png)

输入ftp的名称，随便取名字，并选择你想把那个文件夹作为ftp存放资料的地方，选路径

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168711455-a9f7283a-25e2-4b3c-b9f8-14f29845fafc.png)

选择IP地址

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168724727-b0fed3ef-e194-475f-920f-d22221479040.png)

设置所有用户可访问，并且没码，如果需要密码，就chua创建一个Windows用户，然后指定这个用户就可以了，这里不展开

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168741537-4bf711dd-addb-41e5-9965-23d83b3e8ff7.png)

一切设置完成

最关键的是要允许防火墙

# （二）设置防火墙

回到控制面板，打开防火墙设置

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168752405-ecb78e15-86ee-4c4c-bb58-7bde0d5abe0e.png)

添加允许的程序

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168764378-a83e6561-bb88-42dc-a629-89c61923bf5f.png)

勾选FTP![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168779127-f6072816-f2b2-43ab-82c6-afbf56544bdc.png)



最后添加一个额外的程序![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168794399-47cce883-a963-4e5f-9889-3d112fdda75d.png)

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168798577-e77fbccd-b0ea-47b3-b74f-5af07802b021.png)

添加好如下：

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1650168822746-ce6ac3ae-869d-4338-ab49-b676e3e7259d.png)
 一切搞定

# 常见问题

## 网络ping通浏览器没网

- 设置搜索代理

关闭手动代理设置

![img](https://cdn.nlark.com/yuque/0/2022/png/21986264/1657852457069-28711a57-49ba-4f8e-a957-ec0e94834d67.png)

# 
