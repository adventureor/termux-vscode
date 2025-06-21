# 安卓下通过使用termux体验vscode部分功能
## 零. 前言
本文的目的在于介绍一种在安卓设备上如何体验vscode官方的编辑器的方式。当然目前网上主流方式是使用的开源的项目code-server，但似乎该项目无法使用copilot，正巧看见了一篇[文章](https://www.cnblogs.com/145a/p/18339352)给出了官方vscode的下载方式，所以就来尝试了一下。还有一种方式是桌面直接在本地再跑一个linux桌面，然后在这个桌面上跑linux版本的vscode和wps之类的，但是使用这种方式就有着gpu层层转译性能不足的问题，所以在浏览器中使用的话就可以避免这个问题。本文适合没有什么折腾经历的朋友使用，并不需要太多的基础。
## 一. 准备工作
### 1. 首先准备好我们需要用到的软件(这里需要连接到github，所以如果下载不了的话可以多试几次或者可以尝试别的方法)。

我们需要下载我们的主角[termux](https://github.com/termux/termux-app/releases/download/v0.118.3/termux-app_v0.118.3+github-debug_arm64-v8a.apk)，当然这个链接可能不是最新版也可以点击<https://github.com/termux/termux-app/releases/>选择一个arm64-v8的稳定版。

我们还需要有一个浏览器，这里我选择使用的是firefox nightly版本。我使用火狐是因为他可以安装插件(edge也可以使用插件，但是似乎少了很多，我主要是想用一个全屏显示的功能)，同时还有多端同步的功能(我没有找到其他的在win,linux,android都好用的浏览器)。

### 2. 对termux做一下设置。

首先要给他储存权限，可以通过进入设置里给应用储存权限，也可以执行下面的代码
```bash
termux-setup-storage
```
接下来为了下载更快我们执行下面的代码进行换源
```bash
termux-change-repo
```
通过拓展键盘上的上下左右选择下面的单一源，点击你自己键盘上的换行`ENTER`，然后再用同样的方式选择中国大陆。等待其执行完成后显示
```bash
25 packages can be upgraded. Run 'apt list --upgradable' to see them.
```
我们就可以更新termux了
```bash
apt upgrade
```
中间可能会有几次停止让你选择，只需要一直`ENTER`即可。

### 3. 处理一点小问题，termux崩溃和性能优化问题
最好是在自己的安卓系统内更改对该应用的电池优化，给予termux完全的后台行为。由于各个手机厂商的方式各不相同，这里不给出解决办法。

比较常见的就是
```bash
[Process completed (signal 9) - press Enter] 
```
建议出现此问题可以自行搜索termux+signal 9，需要使用`ADB`命令解决。本人由于已经解决该问题，不会重复出现，可以考虑查看(https://www.bilibili.com/video/BV1p8411d7UN/?spm_id_from=333.337.search-card.all.click)
### 4.使用proot容器安装ubuntu
我们首先安装proot容器
```bash
apt install proot-distro
```
查看有哪些linux发行版
```bash
proot-distro list
```
我们以ubuntu为例，其他发行版也可以下载，推荐不熟悉各个发行版的朋友也下载ubuntu，因为各个版本有些许不同，后面执行的命令也有所区别。后续各位感兴趣也可以去试试其他发行版。

所以我们执行
```bash
proot-distro install ubuntu
```
注意，这里可能也需要访问github才可以，所以如果下载一两分钟没有成功很有可能需要更换网络环境。如果你想使用其他发行版只需要更改ubuntu即可。
### 5. 对ubuntu做一些简单的设置
我们进入通过下面的命令进入ubuntu
```bash
proot-distro login ubuntu
```
和termux一样，我们首先进行换源，我们换成[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu-ports/)，需要编辑`/etc/apt/source.list`文件，所以我们需要使用文本编辑器，熟悉vim语法的朋友可以使用vim进行编辑，这里更推荐不熟悉的朋友使用nano进行编辑。
```bash
nano /etc/apt/source.list
```
然后在最后面输入
```bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble-backports main restricted universe multiverse

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb http://ports.ubuntu.com/ubuntu-ports/ noble-security main restricted universe multiverse
# deb-src http://ports.ubuntu.com/ubuntu-ports/ noble-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble-proposed main restricted universe multiverse
```
点击`Ctrl+x`，之后点击`ENTER`就保存退出文件了。输入
```bash
apt update&apt upgrade -y
```
现在我们可以下载一些软件包了
```bash
apt install sudo wget git
```
我们是ubuntu的管理员，但是由于在安卓的限制下我们如果没有root权限我们是没有真正的root权限的。

我们以victor为例新建一个用户(这里仅限ubuntu，其他的可能略有不同)
```bash
useradd victor
```
这个过程中需要输入密码，注意这里的密码不会显示。之后一直按`ENTER`就可以了。

创建好之后这个新用户我们需要给他设置root权限，就是通过sudo得到权限的，首先编辑
```bash
EDITOR=nano visudo
```
找到
```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL

```
在下面这一行输入`victor   ALL=(ALL:ALL) ALL`，相当于变成了
```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
victor  ALL=(ALL:ALL) ALL
```
再同上面一样保存退出。接下来我们就登录用户
```bash
su victor
```
到目前为止我们就实现了一些基础的设置，完成了我们的准备工作。我们现在通过`exit`退出termux，可能需要多执行几次。
## 二. 安装vscode cli
### 6. 找到vscode cli下载链接
可以去vscode官方网站找到下载链接，选择其他其他平台下载，也可以直接点击<https://code.visualstudio.com/Download>，点击linux下面的CLI arm64版本，不用下载，长按直接下载链接复制链接，这是我现在的[版本](https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-arm64)。

### 7. 在ubuntu中安装vscode cli
进入ubuntu
```bash
proot-disro login ubuntu --user victor #每次进入嫌麻烦可以使用脚本，或者点击上箭头可以浏览历史指令
```
下载文件
```bash
wget 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-arm64' -O vscode #换成你刚刚获得的链接
```
然后我们解压归档这个文件
```bash
tar -xf vscode_cli.tar.gz
```
我们就可以运行vscode了
```bash
./code serve-web --without-connection-token
```
然后使用浏览器访问<http://127.0.0.1:8000>即可。
## 三. 一些美化和使用
