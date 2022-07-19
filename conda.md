conda经常要用到，但是经常忘记有关命令，每次都要搜帖子，所以就自己也写个帖子，方便后续查看.

1.查询conda版本

```
conda -U
```



2.查询所有conda环境

```
conda info
```

3.创建新的conda环境

```
conda create -n test python=3.6
```

4.进入相应conda环境

```
activate test
```

5.退出当前conda环境

```
conda deactivate
```

6.在conda环境中安装tensorflow-gpu(安装其他类比即可)

```
conda install tensorflow-gpu==1.12.0
```

7.删除相应conda环境(首先从该环境退出)

```
conda remove -n test --all
```