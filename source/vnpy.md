# centos7.6 搭建vnpy量化交易环境

## centos7.6 安装python环境

```bash
# 在anaconda.com官网下载安装包，这里需要下载linux版本的x86_64位的包
bash Anaconda3-2019.07-Linux-x86_64.sh
# 如果不修改安装路径的话，按照默认设置即可。
# 安装过anaconda后，需要将/root/.bashrc中的关于anaconda的部分屏蔽掉，不然后面vncserver无法正常启动。
```

## centos7.6 安装桌面

```bash
yum groups install "X Window System" -y
yum install epel-release -y
yum groups install "MATE Desktop" -y
systemctl set-default graphical.target
```

## centos7.6 安装vncserver
```bash
yum install tigervnc-server -y

# 替换User为root，增加显示分辨率参数设置
sed -r -i "s/^(ExecStart.*)<USER>(.*%i)/\1root\2 -geometry 1920x1200 -depth 16/" /lib/systemd/system/vncserver@.service
sed -r -i "s/^(PIDFile.*)home\/<USER>(.*pid)/\1root\2/" /lib/systemd/system/vncserver@.service

mv /lib/systemd/system/vncserver@.service /lib/systemd/system/vncserver@:1.service
systemctl daemon-reload
vncpasswd
systemctl start vncserver@:1.service
systemctl enable vncserver@:1.service

# 屏蔽默认桌面，启动mate桌面
sed -r -i "s@^/etc/X11/xinit/xinitrc$@# &@" /root/.vnc/xstartup
echo "/usr/bin/mate-session &" >> /root/.vnc/xstartup

# 其它操作
# 禁用selinux
sed -r -i "s/^(SELINUX=).*/\1disabled/" /etc/selinux/config
# 关闭防火墙
systemctl stop firewalld.service
systemctl disable firewalld.service

reboot

# 如果是云服务器，需要确保开放了TCP 5901端口
```

## centos7.6 安装vscode
```shell
rpm --import https://packages.microsoft.com/keys/microsoft.asc
sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
yum check-update
yum install code -y
```

## centos7.6 升级gcc9.1.0版本
```shell
yum install gcc gcc-c++ bzip2 m4 -y

tar -jxvf gcc-9.1.0.tar.bz2
./contrib/download_prerequisites

cd gmp;mkdir temp;cd temp
../configure --prefix=/usr/local/gmp-6.1.0
make
make install
cd ../..

cd mpfr;mkdir temp;cd temp
../configure --prefix=/usr/local/mpfr-3.1.4 --with-gmp=/usr/local/gmp-6.1.0
make
make install
cd ../..

cd mpc;mkdir temp;cd temp
../configure --prefix=/usr/local/mpc-1.0.3 --with-gmp=/usr/local/gmp-6.1.0 --with-mpfr=/usr/local/mpfr-3.1.4
make
make install
cd ../..

vim /etc/profile
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/mpc-1.0.3/lib:/usr/local/gmp-6.1.0/lib:/usr/local/mpfr-3.1.4/lib

mkdir temp;cd temp
../configure --disable-multilib --enable-languages=c,c++ --with-gmp=/usr/local/gmp-6.1.0 --with-mpfr=/usr/local/mpfr-3.1.4 --witpc=/usr/local/mpc-1.0.3
make
make install
```

gcc需要9.1.0以上的版本，否则在编译vnpy的时候，会报-std=c++17相关的错误。
gcc编译时间很长，估计得几个小时。

## centos7.6 编译vnpy

```bash
# 切换到python环境
. anaconda3/bin/activate
yum install postgresql-devel* libxkbcommon-x11 -y
cd vnpy
bash install.sh
```
