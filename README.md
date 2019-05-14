# ubuntu-16.04-LimeSDR-OpenAirInterface-LTE-
ubuntu 16.04系统LimeSDR使用OpenAirInterface搭建LTE实验环境
转载https://www.mobibrw.com/2018/10729/comment-page-1

ubuntu 16.04系统LimeSDR V1.4使用OpenAirInterface搭建LTE实验环境
操作系统调整
参考ubuntu 16.04低延迟内核安装低延时内核CPU调整为最大性能模式,并且不允许降低频率

$ sudo apt-get install cpufrequtils
$ sudo touch /etc/default/cpufrequtils
$ sudo sed -i "/GOVERNOR.*/d" /etc/default/cpufrequtils
$ test -s /etc/default/cpufrequtils && sudo sed -i '$a\GOVERNOR=\"performance\"' /etc/default/cpufrequtils || echo "GOVERNOR=\"performance\"" | sudo tee /etc/default/cpufrequtils
$ sudo update-rc.d ondemand disable
$ sudo reboot
 
安装依赖包
$ sudo apt-get install cmake g++ libpython-dev python-numpy swig git libsqlite3-dev libi2c-dev libusb-1.0-0-dev libwxgtk3.0-dev freeglut3-dev

编译安装SoapySDR

$ cd ~
 
$ git clone https://github.com/pothosware/SoapySDR.git
 
$ cd SoapySDR
 
$ git pull origin master
 
$ mkdir build && cd build
 
$ cmake ..
 
$ make -j4
 
$ sudo make install
 
$ sudo ldconfig


编译安装LimeSDR

$ cd ~ 
 
$ git clone https://github.com/myriadrf/LimeSuite.git
 
$ cd LimeSuite
 
#目前的测试发现，超过这个版本的代码无法正常的发出信号，
#也就是手机搜索不到我们创建好的基站，
#反复研究，确认是“Commit 9883bb97 by ignasj, 2017-06-30 下午04:20” 
#对于“lms7_device.cpp”的修改导致的问题，原因为天线发射增益设置不正确
#解决方法参考 https://www.mobibrw.com/?p=11957
#截止目前2018.09.03版本的代码貌似已经修正这个问题了，应该已经可以正常使用了。
#测试来说，还是使用这个版本吧
 
$ git checkout v17.06.0
 
# 不可删除build目录，清理build目录后要还原被误删除的文件，
# 原因在于build目录下存在mcu程序，默认应用启动后从这个目录提取mcu程序刷新到设备
 
$ mkdir build ; cd build
 
# cmake -DCMAKE_BUILD_TYPE=Debug ..
$ cmake ..
 
$ make -j4
 
$ sudo make install
 
$ sudo ldconfig
 
$ cd ../udev-rules/
 
$ sudo bash install.sh
 
# Download board firmware
$ sudo LimeUtil --update
 
#固件下载地址 http://downloads.myriadrf.org/project/limesuite/
#如果上述的固件无法下载，可以从此处下载一份拷贝
#https://www.mobibrw.com/wp-content/uploads/2018/03/17.06-LimeUtils-fpga-fw.zip
 
 编译安装USRP驱动
 $ sudo apt-get install libboost-all-dev libusb-1.0-0-dev python-mako doxygen python-docutils python-requests cmake build-essential
 
$ cd ~
 
$ git clone git://github.com/EttusResearch/uhd.git
 
$ cd uhd; mkdir host/build; cd host/build
 
$ cmake -DCMAKE_INSTALL_PREFIX=/usr ..
 
$ make -j4
 
$ sudo make install
 
$ sudo ldconfig
 
#下面脚本的作用是到http://files.ettus.com/binaries/images/下载最新的镜像文件，并解压缩到 /usr/share/uhd/images目录下
 
$ sudo /usr/lib/uhd/utils/uhd_images_downloader.py
 
#如果上面的镜像下载失败，可以尝试执行如下命令从本站下载
# cd ~ ; wget https://www.mobibrw.com/wp-content/uploads/2018/03/uhd-images_003.010.003.000-release.zip
# sudo unzip uhd-images_003.010.003.000-release.zip -d /user
下载并解压缩需要的补丁  https://www.mobibrw.com/wp-content/uploads/2018/03/opencells-mods-20170710.tgz
$ cd ~
 
# 原始补丁地址 http://open-cells.com/d5138782a8739209ec5760865b1e53b0/opencells-mods-20170710.tgz
 
$ wget https://www.mobibrw.com/wp-content/uploads/2018/03/opencells-mods-20170710.tgz
 
$ tar xf opencells-mods-20170710.tgz

下载并编译eNB

$ cd ~
 
$ git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
 
$ cd openairinterface5g
 
$ git checkout develop
 
#目前的补丁程序只在这个版本上是有效的，后续版本会补丁失败
$ git checkout 08b8b3142df16831396a5283a015564ff56bf91c -b ubuntu16
 
#应用补丁
$ git apply ../opencells-mods/eNB.patch
 
#对于国内的用户来说，国外的几个代码地址需要修改一下，否则会出现无法下载或者下载非常慢的情况
$ sed -i "s/git clone https:\/\/gist.github.com\/2190472.git \/opt\/ssh/wget https:\/\/www.mobibrw.com\/wp-content\/uploads\/2018\/03\/ssh.tar.gz \&\& sudo tar -zxvf ssh.tar.gz -C \/opt/g" cmake_targets/tools/build_helper
 
$ sed -i "s/git clone https:\/\/gitlab.eurecom.fr\/oai\/asn1c.git \/tmp\/asn1c/wget https:\/\/www.mobibrw.com\/wp-content\/uploads\/2018\/03\/asn1c.tar.gz \&\& tar -zxvf asn1c.tar.gz -C \/tmp/g" cmake_targets/tools/build_helper
 
$ sed -i "s/https:\/\/pypi.python.org\/packages\/18\/fa\/dd13d4910aea339c0bb87d2b3838d8fd923c11869b1f6e741dbd0ff3bc00\/netifaces-0.10.4.tar.gz/https:\/\/www.mobibrw.com\/wp-content\/uploads\/2018\/03\/netifaces-0.10.4.tar.gz/g" cmake_targets/tools/build_helper
 
$ sed -i "s/https:\/\/github.com\/google\/protobuf\/releases\/download\/v2.6.1\/protobuf-2.6.1.tar.gz/https:\/\/www.mobibrw.com\/wp-content\/uploads\/2018\/03\/protobuf-2.6.1.tar.gz/g" cmake_targets/tools/build_helper
 
$ sed -i "s/git clone https:\/\/github.com\/protobuf-c\/protobuf-c.git/wget https:\/\/www.mobibrw.com\/wp-content\/uploads\/2018\/03\/protobuf-c.tar.gz \&\& tar -zxvf protobuf-c.tar.gz/g" cmake_targets/tools/build_helper
 
$ sed -i "s/http:\/\/mirrors.dotsrc.org\/gcrypt\/gnutls\/v3.1\/gnutls-3.1.23.tar.xz/https:\/\/www.mobibrw.com\/wp-content\/uploads\/2018\/03\/gnutls-3.1.23.tar.xz/g" cmake_targets/tools/build_helper
 
$ sed -i "s/https:\/\/ftp.gnu.org\/gnu\/nettle\/nettle-2.5.tar.gz/https:\/\/www.mobibrw.com\/wp-content\/uploads\/2018\/03\/nettle-2.5.tar.gz/g" cmake_targets/tools/build_helper 
 
#修正兼容问题，更高版本的protobuf-c跟我们上面安装的版本不匹配，会导致编译错误
$ sed -i "s/cd protobuf-c/cd protobuf-c \&\& git checkout 2a46af42784abf86804d536f6e0122d47cfeea45/g" cmake_targets/tools/build_helper
 
#执行编译
$ source oaienv  
 
$ ./cmake_targets/build_oai -I       # install SW packages from internet
 
# ./cmake_targets/build_oai  -w USRP --eNB # compile eNB

# 注意如果后续重新编译过limesdr的驱动，这部分也需要重新编译
 
$ ./cmake_targets/build_oai -c -w LMSSDR --eNB
 下载并编译EPC(核心网）
 
 $ git clone https://gitlab.eurecom.fr/oai/openair-cn.git
 
$ cd openair-cn
 
$ git checkout develop
 
$ git checkout 724542d0b59797b010af8c5df15af7f669c1e838 -b ubuntu16
 
#应用补丁 
$ git apply ../opencells-mods/EPC.patch
 
#对于国内的用户来说，国外的几个代码地址需要修改一下，否则会出现无法下载或者下载非常慢的情况
$ sed -i "s/git clone https:\/\/gitlab.eurecom.fr\/oai\/freediameter.git -b eurecom-1.2.0/wget https:\/\/www.mobibrw.com\/wp-content\/uploads\/2018\/03\/freediameter.tar.gz \&\& tar -zxvf freediameter.tar.gz \&\& cd freediameter \&\& git checkout eurecom-1.2.0 \&\& cd ../g" build/tools/build_helper
 
$ source oaienv; cd scripts
 
#安装依赖的软件，库
$ ./build_hss -i
# Do you want to install freeDiameter 1.2.0 ?<y/N>: yes
# 其他具体情况自行处理
 
 
#对于国内的用户来说，国外的几个代码地址需要修改一下，否则会出现无法下载或者下载非常慢的情况
$ sed -i "s/git clone git:\/\/git.osmocom.org\/libgtpnl/wget https:\/\/www.mobibrw.com\/wp-content\/uploads\/2018\/03\/libgtpnl.tar.gz \&\& tar -zxvf libgtpnl.tar.gz/g" ../build/tools/build_helper
 
 
 
#Install 3PP SW for mme and spgw
$ ./build_mme -i
# Do you want to install freeDiameter 1.2.0 ?<y/N>: no
# Do you want to install asn1c rev 1516 patched? <y/N>: no
# Do you want to install libgtpnl ? <y/N>: yes
# 其他具体情况自行处理
 
 
 
$ ./build_spgw -i
# Do you want to install libgtpnl ? <y/N>: no
 
#调整mme的编译模板，默认情况下mme编译出来的是Debug版本，
#但是这会触发"src/gtpv2-c/nwgtpv2c-0.11/src/NwGtpv2c.c"的断言异常
#”mme: ~/openair-cn/src/gtpv2-c/nwgtpv2c-0.11/src/NwGtpv2c.c:487: nwGtpv2cCreateLocalTunnel: Assertion `0' failed.“
#这个断言异常并不是大问题，直接无视即可
#因此我们把此处的编译调整为RelWithDebInfo
$ sed -i "s/\"Debug\"/\"RelWithDebInfo\"/g" ../build/mme/CMakeLists.template
 
#修正代码里面的几处问题
$ sed -i "s/while 0/while (0)/g" ../src/utils/log.h
 
$ sed -i 's/char[ \t]*pid_dec\[32]/char pid_dec\[128]/g' ../src/utils/pid_file.c
 
#正式进行编译操作
$ ./build_hss -c -v
 
$ ./build_mme -v
 
$ ./build_spgw -v

配置EPC

# 拷贝配置文件
$ cd ~
 
$ sudo mkdir -p /usr/local/etc/oai
 
$ sudo cp -rp opencells-mods/config_epc/* /usr/local/etc/oai
 
 
# 生成证书文件
$ cd openair-cn; source oaienv; cd scripts
 
$ ./check_hss_s6a_certificate /usr/local/etc/oai/freeDiameter hss.OpenAir5G.Alliance
 
$ ./check_mme_s6a_certificate /usr/local/etc/oai/freeDiameter mme.OpenAir5G.Alliance
 
# 配置出口网卡的名字，这个网卡用于访问互联网的时候使用
# 如果只有一块有线网卡，并且用这块有线网卡上网，则使用如下命令即可拿到网卡名
# `ifconfig | grep "Link" | awk '{print $1}' | grep "en"` 
# 其他情况请根据实际情况配置网卡
$ sudo sed -i "s/PGW_INTERFACE_NAME_FOR_SGI.*/PGW_INTERFACE_NAME_FOR_SGI = \"`ifconfig | grep "Link" | awk '{print $1}' | grep "en"`\";/g" /usr/local/etc/oai/spgw.conf
 
# 配置访问MySQL时候的密码，"/usr/local/etc/oai/hss.conf"，此处假定MySQL的密码为"123"
$ sudo sed -i "s/^MYSQL_pass.*/MYSQL_pass = \"123\"/g" /usr/local/etc/oai/hss.conf
 
 
#导入数据库，建立表结构
$ ./hss_db_import 127.0.0.1 root 123 oai_db ~/opencells-mods/opencells_db.sql
 
 修改我们UE需要使用的配置文件
 $ cd ~
 
$ cd openairinterface5g; source oaienv
 
#配置文件为"targets/PROJECTS/GENERIC-LTE-EPC/CONF/"目录下的包含lmssdr的配置文件
$ sed -i "s/mobile_network_code.*/mobile_network_code = \"92\";/g" ./targets/PROJECTS/GENERIC-LTE-EPC/CONF/enb.band7.tm1.25PRB.lmssdr.conf
 
$ vim ./targets/PROJECTS/GENERIC-LTE-EPC/CONF/enb.band7.tm1.25PRB.lmssdr.conf
 
 找到：
////////// MME parameters:
mme_ip_address      = ( { ipv4       = "192.168.12.148";
                          ipv6       = "192:168:30::17";
                          active     = "yes";
                          preference = "ipv4";
                        }
                      );
 
NETWORK_INTERFACES :
{
    ENB_INTERFACE_NAME_FOR_S1_MME            = "eth0";
    ENB_IPV4_ADDRESS_FOR_S1_MME              = "192.168.12.150/24";
 
    ENB_INTERFACE_NAME_FOR_S1U               = "eth0";
    ENB_IPV4_ADDRESS_FOR_S1U                 = "192.168.12.150/24";
    ENB_PORT_FOR_S1U                         = 2152; # Spec 2152
};
 
 替换为：
 ////////// MME parameters:
mme_ip_address      = ( { ipv4       = "127.0.0.20";
                          ipv6       = "192:168:30::17";
                          active     = "yes";
                          preference = "ipv4";
                        }
                      );
 
NETWORK_INTERFACES :
{
    ENB_INTERFACE_NAME_FOR_S1_MME            = "lo";
    ENB_IPV4_ADDRESS_FOR_S1_MME              = "127.0.0.10/8";
 
    ENB_INTERFACE_NAME_FOR_S1U               = "lo";
    ENB_IPV4_ADDRESS_FOR_S1U                 = "127.0.0.10/8";
    ENB_PORT_FOR_S1U                         = 2152; # Spec 2152
};
测试功能，按照顺序打开四个独立Shell

$ echo -ne "\033]0;HSS\007" 
 
$ cd ~
 
$ cd openair-cn; source oaienv; cd scripts; 
 
#貌似需要运行之前先清理一下数据库才行，否则可能导致设备无法入网，应该是个BUG
$ ./hss_db_import 127.0.0.1 root 123 oai_db ~/opencells-mods/opencells_db.sql
 
$ ./run_hss
 
 $ echo -ne "\033]0;MME\007" 
 
$ cd ~
 
$ cd openair-cn; source oaienv; cd scripts; 
 
$ ./run_mme
 
 
 run_spgw.shShell
$ echo -ne "\033]0;SPGW\007" 

$ cd ~

$ cd openair-cn; source oaienv; cd scripts; 

$ sudo -E ./run_spgw



$ echo -ne "\033]0;SPGW\007" 
 $ echo -ne "\033]0;EUE\007" 
 
$ cd ~
 
$ cd openairinterface5g; source oaienv
 
#配置文件为"targets/PROJECTS/GENERIC-LTE-EPC/CONF/"目录下的包含lmssdr的配置文件
#这个配置文件跟天线的接入方式直接相关，天线接入的是主板上的RX1_H,TX1_2这两个天线接口
#最后的-d 参数是启动图形界面分析流量
 
$ sudo -E ./cmake_targets/lte_build_oai/build/lte-softmodem -O ./targets/PROJECTS/GENERIC-LTE-EPC/CONF/enb.band7.tm1.25PRB.lmssdr.conf --rf-config-file ./targets/ARCH/LMSSDR/LimeSDR_above_1p8GHz_1v4.ini -d
 
#如果启动之后，出现 “[PHY][E]TX : Timeout (sent 6800/7680)” 这样的红色内容，然后退出，可以多尝试几次，一般就可以恢复正常了
$ cd ~
 
$ cd openair-cn; source oaienv; cd scripts; 
 
$ sudo -E ./run_spgw


优先检查设备的MTU，如果设备的MTU大小超过1500,则会导致这种问题，/usr/local/etc/oai/spgw.conf里面配置的MTU大小就是1500，这个大小可以设置为1536，但是貌似会导致应用异常。
目前已知的是小米Note1会记录上次联网时候设备的MTU大小，如果恰好连接过MTU是1536的基站，貌似会使用这个数字，导致问题。
一般建议Android手机执行adb shell ifconfig观察一下手机的输出。APN协议，APN漫游协议，承载系统，这三个要设置，否则MME会崩溃（src/nas/emm/sap/emm_cn.c:358 空指针异常），主要是协议不支持，导致处理异常。正常情况下，已经自动设置了MCC，MNC，如果没有自动设置，请手工修改成MCC 208，MNC 92。






 
 
