# 可视化爬虫Portia安装和部署踩过的坑

背景
Scrapy爬虫的确是好使好用，去过scrapinghub的官网浏览一下，更是赞叹可视化爬虫的犀利。scrapinghub有一系列的产品，开源了大部分项目，Portia负责可视化爬虫的编辑，SpiderCloud负责云端爬虫的部署，Scrapy是实现他们底层的技术。国内的可视化爬虫技术也有不少，据我所知就这几种：

集搜客
造数
如果有其他优秀的可视化爬虫我没有提到，大家可以补充。他们的功能暂不讨论，但他们都没有开源的，不方便我们研究。
而scrapinghub将Portia开源出来了，我们可以通过这个渠道了解可视化的爬虫。虽然开源出来了，可是安装真的不容易啊。

安装
安装之前交代一下，系统和环境吧！

系统：Ubuntu14 server
Python：2.7
目标版本：2.0.8
有这两个就够了，这两个是必须的。至于Python的版本可不可以是3.0以上呢？我只能说在3.0以上的版本环境下，我没有安装成功过，可能是我个人操作失误的原因。
如果仅仅是使用的话，docker和本地安装都是可以的，至于Vagrant可不可以，我就没试过了。
我这里优先说本地部署的方式，因为这种方式以后要改造代码再部署的话比较方便。官方文档说的比较简单，按照官网文档来安装，肯定要踩不少的坑。

我们拟定将Portia安装在/opt目录下面

1、克隆Portia
```
cd /opt
sudo git clone https://github.com/scrapinghub/portia.git
```

如果没有git的话，执行sudo apt-get install git安装Git

2、不要使用虚拟环境
不知道你们有没有强迫症，反正我是有的，用什么pyenv和virtualenv搞过虚拟环境，让我踩了不少坑。里面有什么坑呢？

安装的脚本需要新建文件，需要sudo，如果使用了sudo，你的pyenv就白装了，sudo会使用系统的Python2.7。
portia和splash的脚本的写法不同，导致pyenv和virtualenv生效的情况也不一样。
结论是，我输了，我使用系统的Python2.7来安装。

3、安装脚本

```
cd portia
sudo ./provision.sh install_deps install_splash install_python_deps
```

接下来可以等待安装完成。

4、替换下载慢的脚本(可选)
这步是可选的，如果sip或者PyQt下载的比较慢，可以先手动下载好。
```
wget http://sourceforge.net/projects/pyqt/files/sip/sip-4.17/sip-4.17.tar.gz
wget http://sourceforge.net/projects/pyqt/files/PyQt5/PyQt-5.5.1/PyQt-gpl-5.5.1.tar.gz
```

分别改名为sip.tar.gz和pyqt5.tar.gz放在/downlaods目录中，/downloads没有就新建一个。
然后

```
vi /tmp/splash-2.3.x/dockerfiles/splash/provision.sh
```

把
```
#curl -L -o /downloads/sip.tar.gz http://sourceforge.net/projects/pyqt/files/sip/sip-${SPLASH_SIP_VERSION}/sip-${SPLASH_SIP_VERSION}.tar.gz && \
#curl -L -o /downloads/pyqt5.tar.gz http://sourceforge.net/projects/pyqt/files/PyQt5/PyQt-${SPLASH_PYQT_VERSION}/PyQt-gpl-${SPLASH_PYQT_VERSION}.tar.gz && \
```

注释掉，保存退出。

再次在portia目录下,执行

```
./provision.sh install_splash
```

最后等待安装，当然，如果你网络好，这步可以忽略。

5、配置nginx

```
vi nginx/nginx.conf
```

修改：

```
root /opt/portia/portiaui/dist;
location /static {
   alias /opt/portia/portiaui/dist;
}
```

保存退出。
执行 

```
sudo ./provision.sh configure_nginx configure_initctl
```

6、安装前端依赖和UI
```
sudo ./provision.sh install_frontend_deps build_assets 
```

7、运行
前面的准备工作都做好了，现在运行Portia

```
export PYTHONPATH='/opt/portia/portia_server:/opt/portia/slyd:/opt/portia/slybot'
slyd/bin/slyd -p 9002 -r portiaui/dist & portia_server/manage.py runserver
```

现在打开浏览器  http://localhost:9001  爽爽吧！

题外话

在ctrl+c停掉应用后，再次启动会被提示端口已占用。这个时候使用ps -ef | grep slyd，把对应的slyd端口kill掉

该说明摘自：

https://www.cnblogs.com/ginponson/p/7102411.html

