# docker 中的overlay 存储原理


## 在 linux 中手动的体验 `overlay2`

#### 什么是UnionFS

`联合文件系统`（Union File System）可以把多个目录(也叫分支)内容联合挂载到同一个目录下，而目录的物理位置是分开的。linux 中 UnionFS 的实现有很多，docker 目前用的是 `overlay2`

![docker%20share%20481e7009cb6540a5b1fba4a3cc3921b4/Untitled.png](docker%20share%20481e7009cb6540a5b1fba4a3cc3921b4/Untitled.png)

上面这张图就是联合文件系统的一个示例：把 `lower1`，`lower2`，`upper` 三个文件的内容联合挂载到 `merge` 文件夹。要想知道overlay2是如何把这三个文件夹合并的，要先理解`lowerdir,upperdir,work,merged` 这四种文件夹

- `lowerdir` ：表示较为底层的目录，修改联合挂载点不会影响到lowerdir。就是上图中的 lower1 ，lower2 文件夹
- `upperdir`：表示较为上层的目录，修改联合挂载点会在upperdir同步修改。就是上图中的 upper 文件夹
- `merged`：是 lowerdir 和 upperdir 合并后的联合挂载点。就是上图中蓝色的 merge 文件夹
- `workdir`：用来存放挂载后的临时文件与间接文件。

#### 手动的挂载 overlay2

首先下面的命令会创建这四种目录，并在这文件中创建了一些文件，用来观察 overlay 是如何联合挂载的

```bash
mkdir overlay-test
cd overlay-test
mkdir low1
mkdir low2
mkdir up
mkdir merge
mkdir work

touch low1/a
echo "a in low1" > low1/a
touch low1/b
touch low2/a
echo "a in low2" > low2/a
touch low2/c
touch  up/d
```

运行后的目录结构应该是下面这样的

![docker%20share%20481e7009cb6540a5b1fba4a3cc3921b4/Untitled%201.png](docker%20share%20481e7009cb6540a5b1fba4a3cc3921b4/Untitled%201.png)

然后执行 `overlay` 的挂载命令

`sudo mount -t overlay overlay -o lowerdir=low1:low2,upperdir=up,workdir=work merge`

执行后的目录结构：

![docker%20share%20481e7009cb6540a5b1fba4a3cc3921b4/Untitled%202.png](docker%20share%20481e7009cb6540a5b1fba4a3cc3921b4/Untitled%202.png)

从上图看出 merge 文件夹已经有了 low1，low2，up 三个文件夹的文件

由于 low1 和 low2 都有 a 文件，所以需要看下 merge 里的 a 文件来自于哪个文件夹

```bash
root@noel:~/overlay-test# cat merge/a
a in low1
```

由于上面的命名中 `lowerdir` 文件夹挂载了两个文件，如果挂载了多个文件，左面的优先级高。所以 a 文件来自于 low1 文件夹

#### 修改 merge 后的文件

- 在 merge 文件夹新增 new_file ，在 up 文件夹也新增了改文件

```bash
root@noel:~/overlay-test# touch merge/new_file
root@noel:~/overlay-test# tree
.
├── low1
│   ├── a
│   └── b
├── low2
│   ├── a
│   └── c
├── merge
│   ├── a
│   ├── b
│   ├── c
│   ├── d
│   └── new_file
├── up
│   ├── d
│   └── new_file
└── work
    └── work

6 directories, 12 files
```

- 修改 merge 下的 b 文件，low1 文件夹内的 b 文件没有改变

```bash
root@noel:~/overlay-test# echo "bbbb" >merge/b
root@noel:~/overlay-test# cat merge/b
bbbb
root@noel:~/overlay-test# cat low1/b
root@noel:~/overlay-test#
```

- 修改 merge 下的 d 文件，up 文件夹内的 d 文件随之改变

```bash
root@noel:~/overlay-test# echo "ddddd" > merge/d
root@noel:~/overlay-test# cat merge/d
ddddd
root@noel:~/overlay-test# cat up/d
ddddd
root@noel:~/overlay-test#
```

#### 总结

从上面的一些现象来看，`lowerdir` 就像是 docker 中的 image，**low1** 和 **low2** 就是 image 中 `layer` 的概念。upperdir 就是 docker 中的 container。

下图就是 dcoker 官网中的一个图片

![docker%20share%20481e7009cb6540a5b1fba4a3cc3921b4/Untitled%203.png](docker%20share%20481e7009cb6540a5b1fba4a3cc3921b4/Untitled%203.png)

## docker 中的 overlay

### 下载 image

使用docker pull ubuntu 来下载 image 来研究

```bash
root@noel:~# docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
83ee3a23efb7: Pull complete
db98fc6f11f0: Pull complete
f611acd52c6c: Pull complete
Digest: sha256:703218c0465075f4425e58fac086e09e1de5c340b12976ab9eb8ad26615c3715
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
root@noel:~#
```

上面命令可以看出我们下载了的 ubuntu 镜像有三层，那上面的例子来对比就是有 low1，low2，low3 三个文件夹

### 观察 image 和 container 的文件结构

#### image

docker 下载的镜像都在 `/var/lib/docker/overlay2` 进入该目录后看下目录结构

```bash
root@noel:/var/lib/docker/overlay2# tree -L 2
.
├── 18533d22086c0f91c14a11499d506f53452c92a0179ce20de5d8de21b084b05c
│   ├── committed
│   ├── diff
│   └── link
├── 78e6965c4bca0f7a6da0d72a1dc361933e9635ceec89519625b03bd4253d73b9
│   ├── diff
│   ├── link
│   ├── lower
│   └── work
├── 7979e6ce4b2f7675255db2aec3d01a1f545425652952802cae4e42b8e8d56d78
│   ├── committed
│   ├── diff
│   ├── link
│   ├── lower
│   └── work
└── l
    ├── 6CZZH722UFGUBASNSUFY24SAVT -> ../18533d22086c0f91c14a11499d506f53452c92a0179ce20de5d8de21b084b05c/diff
    ├── PJGLJKKB3IET6REXBDXWWZPJPB -> ../7979e6ce4b2f7675255db2aec3d01a1f545425652952802cae4e42b8e8d56d78/diff
    └── UFLIZTAI4O6KBT4XSDYLFFRVIQ -> ../78e6965c4bca0f7a6da0d72a1dc361933e9635ceec89519625b03bd4253d73b9/diff
```

- `diff` ：文件实际存放位置
- `lower`：存储的是父 layer 的 id，如果没有该文件，说明该 layer 是根节点
- `link`：存储的是该文件夹的 `短id` 和 `l` 文件夹内的软连接相符

#### container

`docker run -it ubuntu` 启动容器后再观察文件的变化

```bash
root@noel:/var/lib/docker/overlay2# tree -L 2
.
├── 18533d22086c0f91c14a11499d506f53452c92a0179ce20de5d8de21b084b05c
│   ├── committed
│   ├── diff
│   └── link
├── 78e6965c4bca0f7a6da0d72a1dc361933e9635ceec89519625b03bd4253d73b9
│   ├── committed
│   ├── diff
│   ├── link
│   ├── lower
│   └── work
├── 7979e6ce4b2f7675255db2aec3d01a1f545425652952802cae4e42b8e8d56d78
│   ├── committed
│   ├── diff
│   ├── link
│   ├── lower
│   └── work
├── bd9fd7613eb3a36398b0c9929740c7393c1e8e97208486a4d5e9b54658f54e68
│   ├── diff
│   ├── link
│   ├── lower
│   ├── merged
│   └── work
├── bd9fd7613eb3a36398b0c9929740c7393c1e8e97208486a4d5e9b54658f54e68-init
│   ├── committed
│   ├── diff
│   ├── link
│   ├── lower
│   └── work
└── l
    ├── 6CZZH722UFGUBASNSUFY24SAVT -> ../18533d22086c0f91c14a11499d506f53452c92a0179ce20de5d8de21b084b05c/diff
    ├── PJGLJKKB3IET6REXBDXWWZPJPB -> ../7979e6ce4b2f7675255db2aec3d01a1f545425652952802cae4e42b8e8d56d78/diff
    ├── SIVTPIQFVUBGEQPAQABNZKAO4Q -> ../bd9fd7613eb3a36398b0c9929740c7393c1e8e97208486a4d5e9b54658f54e68/diff
    ├── UFLIZTAI4O6KBT4XSDYLFFRVIQ -> ../78e6965c4bca0f7a6da0d72a1dc361933e9635ceec89519625b03bd4253d73b9/diff
    └── XWLNFDN6UKH5MVFD32HLGEH57U -> ../bd9fd7613eb3a36398b0c9929740c7393c1e8e97208486a4d5e9b54658f54e68-init/diff
```

可以发现多了两个文件夹 `bd9fd7613.....8f54e68` 和 `bd9fd7613.....8f54e68-init`

`bd9fd7613.....8f54e68` 就是 container 的工作目录

`init` 为结尾的文件夹属于 lowerdir 层。init 层是 docker 项目单独生成的一个内部层，专门用来存放 /etc/hosts、/etc/resolv.conf 等信息。需要这样一层的原因是，这些文件本来属于只读的系统镜像层的一部分，但是用户往往需要在启动容器时写入一些指定的值比如hostname，所以就需要在可读写层对它们进行修改。可是，这些修改往往只对当前的容器有效，我们并不希望执行 docker commit 时，把这些信息连同可读写层一起提交掉。所以，docker 做法是，在修改了这些文件之后，以一个单独的层挂载了出来。而用户执行 docker commit 只会提交可读写层，所以是不包含这些内容的。

这时运行 `mount | grep docker` 查看系统目前挂载了什么文件

```
overlay on /var/lib/docker/overlay2/bd9fd7613eb3a36398b0c9929740c7393c1e8e97208486a4d5e9b54658f54e68/merged type overlay 
(
rw,relatime,
lowerdir=
/var/lib/docker/overlay2/l/XWLNFDN6UKH5MVFD32HLGEH57U:
/var/lib/docker/overlay2/l/UFLIZTAI4O6KBT4XSDYLFFRVIQ:
/var/lib/docker/overlay2/l/PJGLJKKB3IET6REXBDXWWZPJPB:
/var/lib/docker/overlay2/l/6CZZH722UFGUBASNSUFY24SAVT,

upperdir=/var/lib/docker/overlay2/bd9fd7613eb3a36398b0c9929740c7393c1e8e97208486a4d5e9b54658f54e68/diff,
workdir=/var/lib/docker/overlay2/bd9fd7613eb3a36398b0c9929740c7393c1e8e97208486a4d5e9b54658f54e68/work
)
```

可以看到 `upperdir` 在 `bd9fd7613.....8f54e68/diff`  目录

在容器内使用 `touch inner_container.txt` 新建一个文件，看看该文件在文件在宿主机的哪个文件夹下

```bash
root@noel:/var/lib/docker/overlay2# find ./ -name inner_container.txt
./bd9fd7613eb3a36398b0c9929740c7393c1e8e97208486a4d5e9b54658f54e68/diff/inner_container.txt
./bd9fd7613eb3a36398b0c9929740c7393c1e8e97208486a4d5e9b54658f54e68/merged/inner_container.txt
```

可以看到在容器内新增了一个文件，只会在宿主机的 `upperdir` 和 `merged` 做出修改



## 参考

* https://zhuanlan.zhihu.com/p/95590072、
* https://blog.csdn.net/luckyapple1028/article/details/78075358

* https://staight.github.io/2019/10/04/%E5%AE%B9%E5%99%A8%E5%AE%9E%E7%8E%B0-overlay2/
