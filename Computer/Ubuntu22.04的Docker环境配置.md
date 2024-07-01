# Ubuntu22.04下的Docker环境配置

&emsp;&emsp;最近主线进行到AI-IMU，涉及到深度学习和李群李代数，需要用到Libtorch和Sophus库，因此需要首先把环境搭建起来。在一开始时尝试自己配置这两个库，首先在Windows下配置，结果把环境变量弄的一团糟，Sophus库的配置即为繁琐，索性就迁移到Linux上，因此之后的项目工程都打算使用Linux。

&emsp;&emsp;在Linux上，Sophus库的配置较为简单，根据github上的相关教程即可简单搭建。Libtorch的配置有些麻烦，而且涉及到cuda库的相关配置，很容易就把Linux的环境搞崩。之前自己动手试过一次，结果果不其然把显卡驱动搞崩了。之后师兄给了我一个Docker镜像，里面装了他自己的环境库。因此又学习了一下如何使用docker下的环境运行我自己的工程。接下来是如何在Ubuntu22.04下加载Docker镜像的具体流程。

## 1. Docker的安装
### 1.1 更新安装包 
&emsp;&emsp;在终端中更新Ubuntu软件包列表和以安装软件的版本
> sudo apt update \
> sudo apt upgrade

### 1.2 安装Docker依赖
&emsp;&emsp;Docker在Ubuntu上依赖一些软件包，在命令行中输入如下命令安装这些依赖项
> sudo apt-get install ca-certificates curl gnupg lsb-release

### 1.3 添加Docker官方GPG密钥
&emsp;&emsp;执行以下命令来添加Docker官方的GPG密钥，添加完成后，会在命令行中输出OK
> curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

### 1.4 添加Docker软件源
&emsp;&emsp;执行以下命令来添加Docker的软件源
> sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

### 1.5 安装Docker
&emsp;&emsp;执行以下命令安装Docker，其中包括Docker的一些附加项
> sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

### 1.6 配置用户组
&emsp;&emsp;默认状态下，只有root用户才能运行Docker命令，当前用户如果想使用Docker命令，需要将当前用户添加到docker组，以避免每次使用Docker时都需要使用sudo。首先需要创建docker用户组，如果docker用户组存在可以忽略，具体命令为
> sudo groupadd docker

&emsp;&emsp;之后需要将本地用户添加进docker组中，我的用户名为csy，即${USER}=csy，具体命令为
> sudo gpasswd -a csy docker

&emsp;&emsp;最后更新用户组，为了使更改生效，使用如下命令更新用户组
> newgrp docker

&emsp;&emsp;在更新完用户组后，需要重启电脑（reboot），使其生效于所有的命令行终端，否则只会在当前命令行终端生效。

## 2. Docker加载镜像
### 2.1 下载镜像文件
&emsp;&emsp;可以将其存放在某一位置。通常.image文件比较大，可以将其存在内存比较大的位置下面。
### 2.2 加载.image文件
&emsp;&emsp;根据docker的load功能，加载image文件。我的image文件(torch2的docker环境)存放在了usr/Docker文件夹下，因此具体的命令为
> docker load -i /usr/Docker/awf_torch2.image

&emsp;&emsp;一般来说，docker的image文件比较大，因此加载的时间会比较长，需要等待一段时间才能加载完成。

### 2.3 检查加载完成的image
&emsp;&emsp;load完image文件后，可以检查image是否load进了本地docker中，使用image命令，检查本地docker中存放的不同容器镜像，命令为
> docker images -a

&emsp;&emsp;输入完命令后，可查看dockers容器中的image镜像，在加载完awf_torch2镜像之后，docker images -a的命令结果为
> PEPOSITORY TAG IMAGE ID CREATED SIZE \
> awf_env torch2 id time 80GB


## 3. Docker创建容器

### 3.1 运行加载完后的image，创建容器
&emsp;&emsp;load完image文件并确认无误后，可在命令行终端运行这个镜像，使用docker的run命令，参数类型为-it，参数为查看images信息中的PEPOSITORY:TAG，在容器内执行/bin/bash命令，命令为
> docker run -it awf_env:torch2 /bin/bash

&emsp;&emsp;docker run参数的说明（简单的），全部的可以谷歌搜索Docker run参数

- -i: 以交互模式运行容器，通常与-t同时使用（即-it）
- -t: 为容器重新分配一个伪输入终端，通常与-i同时使用
- --name: 为容器指定一个名称，如果未输入这个参数，容器将会自己进行命名

&emsp;&emsp;输入完这个命令后，即可创建并进入这个docker环境下。注意docker run命令是新建容器，是利用镜像在计算机中创建了一个容器。之后进入创建的这个容器，需要用到exec命令进入，不能用run，命令为
> docker exec -it 容器ID /bin/bash

&emsp;&emsp;同时要注意，每次重新开机之前，需要先启动docker，如果未设置docker开机自启，则使用命令docker exec -it 容器id /bin/bash进入容器，具体命令为
> docker start 容器ID 
 
### 3.2 将本地文件夹与Docker容器中的文件夹同步
&emsp;&emsp;一般情况下，Docker在某一代码工程下，一般情况下，代码工程文件夹内不会存放数据文件夹，因此需要将本地的数据文件夹与Docker的数据文件夹同步，否则程序将不会识别自己本地数据文件夹中的文件。这时需要用到Docker的Mount命令，将Docker文件夹与本地文件夹进行同步（如果Docker中没有这个文件夹，将会创建）。一般情况下，在创建容器（即run）的过程中，就会加入--mount命令，进行容器文件夹的同步。Docker mount命令为
> docker --mount type=bind, source=/path_local, destination=/path_docker 

其中， source后面的为本地文件夹， destination后面的为docker中的文件夹（如果没有这个文件夹，将会进行创建），一般情况下，这两个文件夹的名称路径最好保持一致。

&emsp;&emsp;在创建容器时，即进行文件夹的同步，具体命令为
> docker run --name container_name -it \
> --mount type=bind,source=/path_local,destination=/path_docker \
> --mount .... \
> container_name /bin/bash

&emsp;&emsp;(本地环境中的具体命令为)
> docker run --name libtorch -it \
> --mount type=bind,source=/home/csy/Data,destination=/home/csy/Data \
> --mount type=bind,source=/home/csy/Work/APDR_MM,destination=/home/csy/Work/APDR_MM \
> --mount type=bind,source=/home/csy/Work/Mag_DB,destination=/home/csy/Work/Mag_DB \
> awf_env:torch2 /bin/bash

&emsp;&emsp;这样我们就得到了一个名为libtorch的容器。


### 3.3 进行容器的查看
&emsp;&emsp;创建完成容器后，可以利用docker ps命令查看创建的容器，具体命令为
> docker ps

&emsp;&emsp;docker ps参数的说明

- a: 显示所有的容器，包括未运行的 
- f: 根据条件过滤显示内容
- l: 显示最近创建的容器          
- n: 列出最近创建的n个容器       
- s: 显示总的文件大小

### 3.4 容器的删除
&emsp;&emsp;在使用完容器后，可以将容器关闭，且只有在停止容器后才可以删除容器，容器关闭的命令为
> docker stop 容器ID

&emsp;&emsp;紧接着，即可以将容器删除
> docker rm Name 

或者

> docker rm CONTAINER ID

Name: 容器的名称 &emsp;&emsp; CONTAINER ID: 容器的ID

在输入命令时选择容器ID还是选择容器名字，都可以将这个容器删除。

### 3.5 容器中创建的文件在宿主机中权限属于root ，而不是当前用户的问题解决方法.
&emsp;&emsp;在创建docker容器时（run），Docker中创建的文件在宿主机中的权限输入Root，而不是当前用户，这就导致创建文件后，本地文件夹与Docker串联的文件夹中的新创建的文件只能读取而不能修改。因此在创建Docker容器时，需要以指定用户的形式启动并进入docker容器。

&emsp;&emsp;首先需要以指定的用户创建启动容器，及在创建容器时，指定用户，具体命令为
> docker run -it -u user_name_id --name container_name -d image_name /bin/bash

&emsp;&emsp;如果需要将容器中的文件夹与本地文件夹进行串联，需要用到--mount命令，具体命令为 (3.2中所示)
> docker run -it -u user_name_id --name container_name \
> --mount type=bind,source=/path_local,destination=/path_docker \
> --mount .... \
> -d container_name /bin/bash

&emsp;&emsp;其中注意，user_name_id不是用户名，而是uid(通常uid为1000)，查看uid的命令为
> id admin

&emsp;&emsp;run完成创建docker容器后，即可以指定用户启动容器 (exec)
> docker exec -it -u user_name_id container_name /bin/bash

&emsp;&emsp;进入容器后，会发现用户名为I have no name，-user指定用户id和uid，进入容器中会有I have no name！，这是正常的。

&emsp;&emsp;(本地环境中的具体命令为)
> docker run -it -u 1000 --name libtorch  \
> --mount type=bind,source=/home/csy/Data,destination=/home/csy/Data \
> --mount type=bind,source=/home/csy/Work/APDR_MM,destination=/home/csy/Work/APDR_MM \
> --mount type=bind,source=/home/csy/Work/Mag_DB,destination=/home/csy/Work/Mag_DB \
> -d awf_env:torch2 /bin/bash

> docker exec -it -u 1000 libtorch /bin/bash