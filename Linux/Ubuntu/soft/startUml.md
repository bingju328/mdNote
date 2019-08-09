StarUML(3.0.2) for linux破解
96  Killshadow
2018.09.28 12:04* 字数 203 阅读 1285评论 0喜欢 0
0x00 工具准备
官网下载
环境: Ubuntu 16.04 LTS
可选

APPimagetools(用于重打包)
已破解的StarUML (提取码: wp88)(可直接使用)
0x01 开始破解
首先下载好appimage文件之后可以试运行一下, 或者直接解包:
chmod +x StarUML-3.0.1-x86_64.AppImage
./StarUML-3.0.1-x86_64.AppImage
# 解包
./StarUML-3.0.1-x86_64.AppImage --appimage-extract
安装npm
sudo apt install npm
# 升级最新版本npm(可能需要给shell加个代理)
sudo npm install npm@latest -g
安装asar
sudo npm install -g asar
进入在第1步解压好的文件夹, 再cd resources, 解压app.asar:
ks@ks:~/software/StarUML$ cd resources/
ks@ks:~/software/StarUML/resources$ ls
app.asar  app-update.yml  electron.asar
ks@ks:~/software/StarUML/resources$ asar extract app.asar app
ks@ks:~/software/StarUML/resources$ ls
app  app.asar  app-update.yml  electron.asar
然后修改验证函数:
gedit app/src/engine/license-manager.js
替换过程如下:
  checkLicenseValidity () {
    this.validate().then(() => {
      setStatus(this, true)
    }, () => {
      // setStatus(this, false) // 修改之前
      // UnregisteredDialog.showDialog() // 修改之前
      setStatus(this, true) // 修改之后
    })
  }
重打包(可选), 也可以直接使用解包后的二进制文件.
./appimagetool-x86_64.AppImage ~/software/StarUML/
