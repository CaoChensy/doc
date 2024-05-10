> anaconda

```git
# 安装
bash Anaconda2-2.4.1-Linux-x86_64.sh

# 环境变量
sudo gedit /etc/profile

export PATH=$PATH:/home/software/anaconda3/bin

source /etc/profile
```

> jupyterLab

1. 生成密码

```python
from notebook.auth import passwd

passwd()
Enter password:
Verify password:
'sha1:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
```

2. 生成配置文件

```git
jupyter notebook --generate-config
```

3. 配置文件的设置

```python
# 允许远程访问
c.NotebookApp.allow_remote_access = True

# 不使用本地浏览器打开
c.NotebookApp.open_browser = False

# 允许所有IP访问
c.NotebookApp.ip='*'

# 配置密码
c.NotebookApp.password = 'sha1:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
```

4. jupyter-lab 后台运行

```git
1. 入门级: jupyter notebook --allow-root > jupyter.log 2>&1 &
2. 进阶版: nohup jupyter notebook --allow-root > jupyter.log 2>&1 &
```

**解释**:

1. 用&让命令后台运行, 并把标准输出写入`jupyter.log`中

`nohup`表示`no hang up`, 就是不挂起, 于是这个命令执行后即使终端退出, 也不会停止运行.

2. 终止进程

执行上面第`2`条命令, 可以发现关闭终端重新打开后, 用`jobs`找不到`jupyter`这个进程了, 于是要用`ps -a`, 可以显示这个进程的`pid`。

找到Jupyter进程PID

```shell script
ps -ef | grep xxxx
```

ps -ef : 查看本机所有的进程；
grep xxxx代表过滤找到条件xxxx的进程


`kill -9 pid` 终止进程


