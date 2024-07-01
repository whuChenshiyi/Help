# Ubuntu22.04将Python脚本载入环境变量

&emsp;&emsp;Ubuntu22.04是经典的Linux环境，想要在Linux环境中能够在任何文件夹中不加路径直接调用Python脚本，需要把python脚本放在系统的PATH环境变量所指的某个目录中，或者将脚本自己的文件目录添加到PATH环境变量中。

## 1.使脚本可执行
&emsp;&emsp;首先，确保Python脚本第一行有正确的shebang，在脚本的第一行加上如下代码，使得系统知道是使用python解释器（python或者python3）来运行这个脚本。
> #!/usr/bin/python3 \
> #!/usr/bin/python

&emsp;&emsp;之后，需要将脚本可执行，即添加运行权限。
> chmod +x scripy.py

## 2.移动脚本到PATH目录
&emsp;&emsp;之后可以把脚本移动到/usr/local/bin目录中，通常这个目录下面存放了环境中可以执行的脚本，这个目录是在PATH中的
> sudo mv script.py /usr/local/bin/

## 3.或者，修改PATH的环境变量
&emsp;&emsp;如果不想移动脚本，想在某一个文件夹下存放工作脚本，可以把脚本所在的目录添加到环境变量中。编辑Shell配置文件（根据自己终端所使用的Shell决定，通常是bash和zsh），例如.bashrc或者.zshrc，Shell配置文件中加入
> export PATH=&PATH:/path_to_scriptdirectory(脚本的文件夹路径)

&emsp;&emsp;之后重新加载配置文件或者重新登录以应用修改
> source ~/.bashrc \
> source ~/.zshrc

## 4.为脚本建立软链接
&emsp;&emsp;另外，可以在/usr/local/bin或者已经添加到PATH目录中的脚本创建一个软连接，而并不把脚本实际移动到那里
> sudo ln -s /path_to_script.py /usr/local/bin/script

&emsp;&emsp;完成这些步骤之后，可以在任何位置直接通过脚本名称来调用Python脚本了：
> script.py

或者如果创建了软连接而不带.py扩展名，则可以省略.py
> script