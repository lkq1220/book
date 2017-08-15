# 工作总结

### 1. 获取源码

####开发环境：Ubuntu 14.04

- To initialize Linux SDK source tree ,you need to get repo first

repo下载：sudo apt-get install phablet-tools\(原本网站的sudo apt-get install repo不可以进行正常下载，故修改\)

- 执行：repo init --repo-url=[https://github.com/rockchip-linux/repo](https://github.com/rockchip-linux/repo) -u [https://github.com/rockchip-linux/manifests](https://github.com/rockchip-linux/manifests) -b master出错，报错原因，需要依赖git，并且需要用户和邮箱，故可以知道repo是基于git基础开发，便于git资源管理的一个工具，所以在安装repo之前我们先要安装git。

   - git安装步骤如下：

   \#sudo apt-get install git

   \#sudo apt-get install git-doc git-svn git-email git-gui gitk

   上述操作是在 Ubuntu14.04 中进行的，其他版本的 Ubuntu 或者 Linux 发行版也许有些许差别，比如说包名不是 git 而是 git-core，可以在git官网查看具体的安装说明。

- 如果是第一次使用 Git，需要进行如下设置用户名和邮箱：

   \#git config --global user.name "LKQ1220"

   \#git config --global user.email "（你自己的邮箱地址）"

- 设置完成之后，执行repo init --repo-url=[https://github.com/rockchip-linux/repo](https://github.com/rockchip-linux/repo) -u [https://github.com/rockchip-linux/manifests](https://github.com/rockchip-linux/manifests) -b maste ，提醒初始化成功并要求 ，升级原本的repo版本 cp /home/lkq/rk-linux/.repo/repo/repo /usr/bin/repo

- 执行sudo repo sync，开始下载源码



