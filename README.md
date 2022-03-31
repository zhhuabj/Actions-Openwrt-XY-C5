## 用法
参考：　https://p3terx.com/archives/build-openwrt-with-github-actions.html

 - 打开 https://github.com/P3TERX/Actions-OpenWrt , 点击'Use this template'.　将该项目clone到自己帐户(命名为openwrt_xiaoyuc5)
 - 在　https://github.com/zhhuabj/openwrt_xiaoyuc5　中点击"Create a new file'，命名为.config, 它的内容见附件
 - 在　https://github.com/zhhuabj/openwrt_xiaoyuc5/actions/workflows/build-openwrt.yml　中点击"Actions"开始编译
 - 注：这里实际使用的不是P3TERX, 而是https://github.com/Ljzkirito/Actions-Openwrt-XY-C5 , 所以也不叫xiaoyuc5

另外，安装ipk时总是报：but incompatible with the architectures configured
```
root@OpenWrt:~# opkg print-architecture | awk '{print $2}'
all
noarch
mipsel_24kc

# opkg print-architecture
arch all 1
arch noarch 1
arch mipsel_24kc 10
```
试过了网上的修改/etc/opkg/opkg.conf的方法不好，改用下列方法安装：
```
解压 ipk 包: tar xzf {packagename}.ipk
解压 data 包完成安装: tar xzf data.tar.gz -C /
删除临时文件: rm control.tar.gz data.tar.gz debian-binary
```
