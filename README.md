## 用法
参考：　https://p3terx.com/archives/build-openwrt-with-github-actions.html

 - 打开 https://github.com/P3TERX/Actions-OpenWrt , 点击'Use this template'.　将该项目clone到自己帐户(命名为openwrt_xiaoyuc5)
 - 在　https://github.com/zhhuabj/openwrt_xiaoyuc5　中点击"Create a new file'，命名为.config, 它的内容见附件
 - 在　https://github.com/zhhuabj/openwrt_xiaoyuc5/actions/workflows/build-openwrt.yml　中点击"Actions"开始编译
 - 注：这里实际使用的不是P3TERX, 而是https://github.com/Ljzkirito/Actions-Openwrt-XY-C5 , 所以也不叫xiaoyuc5

## 附件 - 本地编译＆如何生成.config

```
1, 准备lede源码，及安装相关依赖包(ubuntu 20.04上编译)

sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync -y
proxychains4 git clone https://github.com/coolsnowwolf/lede
cd lede

2, 添加helloworld与passwall的feed, 但这样做之后在make menconfig时找不着passwall的选项(https://www.right.com.cn/FORUM/thread-8201742-1-1.html), 能看到ssr的
# need to run 'make clean && ./scripts/feeds clean' after changing the feed source
sed -i '$a src-git helloworld https://github.com/fw876/helloworld' feeds.conf.default
sed -i '$a src-git passwall https://github.com/xiaorouji/openwrt-passwall' feeds.conf.default
proxychains4 ./scripts/feeds update -a
proxychains4 ./scripts/feeds install -a
ls package/feeds/passwall/

3, 总是使用官方golang版本来避免xray&v2ray的编译错误
pushd feeds/packages/lang
rm -fr golang && svn co https://github.com/openwrt/packages/trunk/lang/golang
popd

4, 利用一些自定义config(包括IPv6(Extra packages -> ipv6helper: odhcp6c, odhcpd-ipv6only, luci-proto-ipv6, luci-proto-ppp), USB storage, USB wifi, NFSv4, OpenSSH, OpenVPN, tailscale, SSR等
wget https://raw.githubusercontent.com/zhhuabj/Actions-Openwrt-XY-C5/main/seed.config_initial
cp seed.config_initial .config
make defconfig
# yes '' | make oldconfig 
# custom your others
make menuconfig

5, 下载依赖
proxychains4 make -j8 download V=s
make -j1 V=s
make -j$(($(nproc) + 1)) V=s
make download -j$(($(nproc) + 1))
find dl -size -1024c -exec ls -l {} \;
find dl -size -1024c -exec rm -f {} \;

6, 编译
make -j$(nproc) || make -j1 V=s
```
若是二次编译：
```
git pull
rm -rf tmp/ && rm -rf .config
./scripts/feeds clean
./scripts/feeds update -a && ./scripts/feeds install -a
wget https://raw.githubusercontent.com/zhhuabj/Actions-Openwrt-XY-C5/main/seed.config_initial -O .config
make defconfig
make -j1 V=s
```
基于Lienol源码编译
lean的源码在编译时报一个错(忘了什么错了），所以改成Lienol源码来做．
```
sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3.5 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf -y
proxychains4 git clone https://github.com/Lienol/openwrt openwrt_lienol
cd openwrt_lienol
echo 'src-git me https://github.com/huchanghui123/Lienol-openwrt-packages-backup' >> feeds.conf.default
sed -i '$a src-git passwall https://github.com/xiaorouji/openwrt-passwall' feeds.conf.default
./scripts/feeds clean && proxychains4 ./scripts/feeds update -a && proxychains4 ./scripts/feeds install -a
wget https://raw.githubusercontent.com/zhhuabj/Actions-Openwrt-XY-C5/main/seed.config_initial -O .config
make defconfig
# select 'LuCI -> Applications -> luci-app-shadowscoks-libev'
make menuconfig
proxychains4 make -j8 download v=s && make -j1 V=s

但上面编译后发现ssr不可用．于是单独编译passwall时报了下列错
$ make feeds/me/lienol/luci-app-passwall/compile V=s
WARNING: Makefile 'package/feeds/me/luci-app-ssr-python-pro-server/Makefile' has a dependency on 'python', which does not exist
make[1]: Entering directory '/bak/linux/openwrt_lienol'
make[1]: *** No rule to make target 'feeds/me/lienol/luci-app-passwallcompile'.  Stop.
make[1]: Leaving directory '/bak/linux/openwrt_lienol'
make: *** [/bak/linux/openwrt_lienol/include/toplevel.mk:230: feeds/me/lienol/luci-app-passwallcompile] Error 2

继续单独编译ssr吧, 但也在对shadowsocksr-libev应用一个patch时发生了代码冲突失败了．
cd package && git clone https://github.com/fw876/helloworld && cd ..
cat << EOF | tee -a .config
CONFIG_PACKAGE_luci-app-ssr-plus=y
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_Shadowsocks=y
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_V2ray_plugin=y
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_Xray=y
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_Trojan=y
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_Redsocks2=n
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_NaiveProxy=n
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_Kcptun=n
CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_ShadowsocksR_Server=n
EOF
make defconfig
grep -r 'ssr' .config
make package/helloworld/luci-app-ssr-plus/compile V=s

继续尝试
rm -rf package/helloworld/
rm -rf feeds/me
cd package
git clone https://github.com/kenzok8/openwrt-packages.git
git clone https://github.com/kenzok8/small.git
cd ..
make menuconfig
proxychains4 make package/openwrt-packages/luci-app-ssr-plus/compile V=s
又报了下列错：
server.c:1069:13: note: 'snprintf' output between 1 and 257 bytes into a destination of size 256
             snprintf(query->hostname, 256, "%s", host);
             ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
cc1: all warnings being treated as errors
make[5]: *** [Makefile:647: ss_server-server.o] Error 1
make[5]: Leaving directory '/bak/linux/openwrt_lienol/build_dir/target-mipsel_24kc_musl/shadowsocksr-libev-2.5.6/server'
然后搜索.config将和libev相关的所有包去掉后错误依旧

看来这些包的依赖问题真多啊．

```
参考：　https://sspai.com/post/61463
