# 下载、编译、安装FuchsiaOS到KhadasVim3开发板

- #### 安装[Ubuntu](https://ubuntu.com/download)(Server/Desktop)

- #### 安装必要工具
```
sudo apt install curl git unzip
mkdir Fuchsia && cd Fuchsia
curl --location --create-dirs --output .jiri_root/bin/cipd https://fuchsia.fsf.org.cn/bootstrap/cipd-linux-amd64
curl --location --create-dirs --output .jiri_root/bin/jiri https://fuchsia.fsf.org.cn/bootstrap/jiri-linux-amd64
chmod +x .jiri_root/bin/cipd .jiri_root/bin/jiri
export PATH=${PATH}:${PWD}/.jiri_root/bin:${PWD}/scripts
```

- #### 下载[Fuchsia源码](https://fuchsia.dev/fuchsia-src/get-started/get_fuchsia_source)
> 使用的是【[Fuchsia OS 中国镜像](https://fuchsia.fsf.org.cn/)】提供的操作步骤
```
mkdir -p build && echo "internal_access = false" > build/cipd.gni
jiri init -keep-git-hooks=true
# manifest 文件从 https://fuchsia.fsf.org.cn/manifest/ 选择，一个月以内的文件确保有效
curl --location --output .jiri_manifest https://fuchsia.fsf.org.cn/manifest/fuchsia-20210921.xml
jiri update -run-hooks=false -v
jiri update -v
echo "have_firmware = false" > zircon/prebuilt/config.gni
```

- #### 编译[Fuchsia源码](https://fuchsia.dev/fuchsia-src/get-started/build_fuchsia)

> // If you forgot to --with-base the devtools:<br/>
> // error: 2 (/boot/bin/sh: 1: Cannot create child process: -1 (ZX_ERR_INTERNAL):<br/>
> // failed to resolve fuchsia-pkg://fuchsia.com/ls#bin/ls
```
fx set core.vim3 --with-base //bundles:tools
fx build -j32
```

- #### 刷入支持FuchsiaOS内核Zircon的U-Boot
[官方教程](https://fuchsia.dev/fuchsia-src/development/hardware/khadas-vim3)中用到了串口控制台(SerialConsole)，连接串口需要USB转ttl工具。如果手头没有这个工具，可以借助AOSP在Vim3上U-Boot；AOSP在Vim3上U-Boot刷入完成后会自动进入fastboot模式。

> Vim3自带到U-Boot不支持硬件按钮进入fastboot模式
> 刷入支持FuchsiaOS内核Zircon的U-Boot后，就可以使用硬件按钮进入fastboot模式
```
1.按住Btn3，点击一下Btn1
2.持续按住Btn3几秒钟之后松开，就进入fastboot模式了
```

> 这个借助AOSP在Vim3上U-Boot的方法，是作者经过数次失败后才摸索出来的
> 有必要买一个USB转ttl工具，因为后续调试FuchsiaOS代码的时候也需要用到

##### 在[这里](https://chrome-infra-packages.appspot.com/p/fuchsia/prebuilt/third_party/firmware/vim3/+/2EuVZARuOjI_Z5HMNFrDv7V86Aqq0iZH7DesElTIqbYC)下载最新的U-Boot，解压后得到：u-boot.bin

解锁flashing
```
fastboot flashing unlock
fastboot flashing unlock_critical
```

刷入U-Boot
```
fastboot flash bootloader u-boot.bin
fastboot reboot
```

刷入FuchsiaOS
> 使用硬件按钮方法进入fastboot模式
```
fx flash --pave
```

等待完成，重启Vim3，连接HDMI、USB鼠标/键盘，就可以体验FuchsiaOS了。
