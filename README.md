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
