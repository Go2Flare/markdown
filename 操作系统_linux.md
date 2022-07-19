# 在Windows11上通过公钥验证登录Linux服务器客户端

工作步骤
1. win开启openssh

  我们去 `win + s` 打开搜索页，搜索`服务` 去服务里找到`openssh Authentication Agent` 点击设置为自动启动后，别忘了点击启动，让它现在立马运行！
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191224172236223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQ0NDEzMw==,size_16,color_FFFFFF,t_70)

2. 创建无密码密钥

  ```
  ssh-keygen  -t  rsa
  ```

  ![img](https://img-blog.csdnimg.cn/20191224170133634.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQ0NDEzMw==,size_16,color_FFFFFF,t_70)

  **它会默认在C:\Users\{{用户名}}生成一个 .ssh 的文件夹：**

  ![image-20220421110334355](C:\Users\Flare\AppData\Roaming\Typora\typora-user-images\image-20220421110334355.png)

3. 把公钥拷贝到服务器上

  ```
  scp C:\Users\Flare\.ssh\id_rsa.pub root@47.106.87.191:~
  这个时候由于还没有在服务器上配置好公钥，ssh将会要求用户键入服务器上xxx用户的密码。
  ```

4. 配置公钥
  登录到服务器上

  ```
  touch ~/.ssh/authorized_keys
  ```

  ```
  cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
  #追加在authorized_keys文件的后面
  ```

5. 验证服务器配置

  ```
  vim /etc/ssh/sshd_config
  ```

  有必要请检查一下配置

  ```
  AuthorizedKeysFile      .ssh/authorized_keys
  
  RSAAuthentication yes
  PubkeyAuthentication yes
  ```

7. 将刚上传的公钥内容追加到ssh认证公钥文件中。

  重启sshd

  ```
  systemctl restart sshd
  ```

8. 完成配置
  这时候回到客户端，打开shell重新登录

  ```
  ssh root@{{host}}
  ```


这时候不需要再输入密码，可以直接进入到系统中。
————————————————
原文链接：https://blog.csdn.net/hey_zng/article/details/101359078