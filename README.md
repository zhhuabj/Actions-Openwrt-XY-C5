## 用法
参考：　https://p3terx.com/archives/build-openwrt-with-github-actions.html

 - 打开 https://github.com/P3TERX/Actions-OpenWrt , 点击'Use this template'.　将该项目clone到自己帐户(命名为openwrt_xiaoyuc5)
 - 在　https://github.com/zhhuabj/openwrt_xiaoyuc5　中点击"Create a new file'，命名为.config, 它的内容见附件
 - 在　https://github.com/zhhuabj/openwrt_xiaoyuc5/actions/workflows/build-openwrt.yml　中点击"Actions"开始编译

## 附件 - 本地生成.config (也就是seed.config)
```
sudo apt-get -y install subversion libncurses5-dev git git-core build-essential unzip bzip2 python2.7
git clone https://github.com/coolsnowwolf/lede
cd lede
sed -i 's/^#\(.*helloworld\)/\1/' feeds.conf.default
./scripts/feeds update -a && ./scripts/feeds install -a
# first using the existing XY-C1.config
wget https://raw.githubusercontent.com/zhhuabj/Actions-Openwrt-XY-C5/main/XY-C1.config -O .config
# then enable ext4 storage and usb wifi
cat << EOF | tee -a .config
CONFIG_PACKAGE_kmod-usb-core=y
CONFIG_PACKAGE_kmod-usb-ohci=y
CONFIG_PACKAGE_kmod-usb-uhci=y
CONFIG_PACKAGE_kmod-usb-storage=y
CONFIG_PACKAGE_kmod-usb-storage-extras=y
CONFIG_PACKAGE_kmod-usb2=y
CONFIG_PACKAGE_kmod-scsi-core=y
CONFIG_PACKAGE_block-mount=y
CONFIG_PACKAGE_libmount=y
CONFIG_BUSYBOX_CONFIG_MOUNT=y
CONFIG_PACKAGE_kmod-fs-ext4=y
CONFIG_PACKAGE_e2fsprogs=y
CONFIG_PACKAGE_kmod-usb-net=y
CONFIG_PACKAGE_kmod-usb-net-cdc-ether=y
CONFIG_PACKAGE_kmod-usb-net-rndis=y
CONFIG_PACKAGE_usb-modeswitch=y
CONFIG_PACKAGE_usbutils=y
EOF
make defconfig
# final expand some new packages like openssh-server, vim, etc
make menuconfig
make defconfig
./scripts/diffconfig.sh > seed.config
#make -j8 download V=s
#make -j1 V=s
```
