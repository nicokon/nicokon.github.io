# 从零开始的树莓派家庭共享文件服务搭建


> 现在最新已经是4B了，性能更强了，千兆网口也更适合家庭共享文件。手头正好有3B+就用了。

## 环境

* Win10

## 所需硬件

* 树莓派 3B+
* 散热片2片
* 保护壳一个
* 电源：
* Micro USB 接口数据线（输出电压：5V 输出电流：≥2A）
* 实际使用了IPAD 充电器，感觉其他手机充电器应该也行
* Micro SD卡
* 树莓派不带硬盘，Micro SD卡就是硬盘。最小容量8G，推荐使用16G和32G的卡。
* SD 读卡器
* 网线一根
* 显示器：（可以不用）
* Raspberry pi 带有HDMI接口，可以连接显示器SD 卡读卡器
* 实际使用了SSH 在电脑上连接VNC远程桌面，实际上也不太用到
* 电脑
* 进行SSH连接等各种操作
* 硬盘或其他大容器存储设备以及配套电源等
 {{< admonition tip "note" false >}}
 实际使用了2T的机械硬盘和易驱线+电源，因为树莓派带不动3.5寸的硬盘，硬盘一定要额外的电源。硬盘盒子可能比易驱线更稳。所以移动硬盘可能更省事，直接USB一插就好了，还省空间。
 {{< /admonition >}}

## 所需软件

 > 想要有桌面的，系统也要下desktop版，如果使用OMV5的话，官方手册建议系统是lite版，不过OMV5不在本文讨论之列。

* 系统下载（选一个）
  * [官网](https://www.raspberrypi.org/software/)
  * [清华镜像](https://mirrors.tuna.tsinghua.edu.cn/raspberry-pi-os-images/)
  * [上交镜像](https://mirrors.sjtug.sjtu.edu.cn/raspberry-pi-os-images/)

* 检验MD5：检验下载系统镜像是否正确
  * [MD5–SHA Checksum utility](https://md5-sha-checksum-utility.en.lo4d.com/windows)

* 解压软件
  * 7-Zip

* 格式化SD卡
  * [SDFormatter](https://www.sdcard.org/downloads/formatter/eula_windows/index.html)
  * [h2testw](http://www.heise.de/ct/Redaktion/bo/downloads/h2testw_1.4.zip)（不用也可）

* 烧录系统（选一个）
  * [Etcher](https://www.balena.io/etcher/)
  * win32diskimager

* SSH 客户端
  * [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
  * 或其他

## 准备步骤

### 获取树莓派系统镜像

1. 在[官网](https://www.raspberrypi.org/software/)/[镜像站](https://mirrors.tuna.tsinghua.edu.cn/raspberry-pi-os-images/)下载树莓派系统
2. 校验[MD5](https://md5-sha-checksum-utility.en.lo4d.com/windows)和SHA hash值，确保文件正确（重要）
3. 使用解压工具如`7-Zip`解压出以`.img`结尾的镜像

### 格式化和测试SD卡

1. 将SD卡插入读卡器，读卡器接入电脑的USB接口
2. 使用[SD格式化工具](https://www.sdcard.org/downloads/formatter/eula_windows/index.html)清理SD卡
3. 等待
4. 使用[测试工具](http://www.heise.de/ct/Redaktion/bo/downloads/h2testw_1.4.zip),确保没有错误（可选）
5. 等待
6. 再次格式化SD卡
7. 等待

### 烧录系统到SD卡

1. 使用[烧录工具](https://www.balena.io/etcher/)将系统烧录到SD卡内
2. 等待

### 开启SSH远程连接服务

1. 烧录完，在电脑上进入SD卡内
2. 新建一个名为`ssh`且无任何后缀名的空文件，如果没有网线的话，从初次启动的第六步的第二步开始。如果**没有路由器，没有网线，没有显示器**，从初次启动的第六步开始。
3. 弹出SD卡

### 初次启动

1. 将SD卡插入树莓派

2. 连接电源

3. 连接网线到路由器

4. 登入路由器，进入路由器设置面板，查看树莓派的IP地址，
   * 如果以后只使用有线连接，可以将有线连接的IP地址和MAC地址进行绑定

5. 使用SSH客户端登录
   * cmd：`ssh pi@ip地址`；password：`raspberry`
   * 登录出现`WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED`,进入提示的文件路径，一般是删除`.ssh`文件夹下的known_hosts文件中的某一行（提示路径后面的数字），重新`ssh`就可以了。

6. 如果**没有路由器，没有网线，没有显示器**，只有一台联网的电脑，就可以进行wifi预设置，**有失败的可能性**。
   1. 打开电脑的移动热点
   2. 在创建完`ssh`文件后，再创建一个新文本文档，内容为

      ```
      ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
      update_config=1
      country=US 
      network={
      ssid="Your network SSID"
      psk="Your WPA/WPA2 security key"
      key_mgmt=WPA-PSK
      }
      ```

      要修改`ssid`和`psk`引号里的内容, 这在电脑设置>>移动热点中查看`ssid`的引号内填`网络名称`冒号后面的内容，`psk`填`网络密码`冒号后面的内容

   3. 重命名该文件名为`wpa_supplicant.conf`,同样不要以`.txt`的文件后缀名，可添加多个wifi，只要再增加一个network即可
   4. 完成以上操作后，将卡插入树莓派中，树莓派通上电源
   5. 看笔记本设置>>移动热点页面的`已连接的设备`那里会显示已连接设备一台，并出现树莓派的IP地址，192开头的即是，或者使用IP搜索工具，如`Advanced IP Scanner`
   6. SSH登录

   {{< admonition warning "note" false >}}
   实际上，用网线接的路由器和树莓派，并绑定了有线IP地址。以前在学校的时候没办法用过第六步，寻找IP的失败概率迷之高。当然最简单的是连屏幕，但没屏幕没试过。
   {{< /admonition >}}

### 基本设置

1. 进入设置界面：`sudo raspi-config`
2. 开启VNC Server：**Interfacing Option >> VNC**
3. 位置：Localisation Options>>Locale>>zh_CN,UTF-8,UTF-8
4. 时区：Localisation Options>>Timezone>>Asia>>Shanghai
5. finish 重启
6. 更改镜像源：[清华](https://mirror.tuna.tsinghua.edu.cn/help/raspbian/)
   1. `sudo nano /etc/apt/sources.list` 删除/`#`注释掉原文，复制黏贴，保存`Ctrl+O`+回车+`Ctrl+X`退出
   2. `sudo nano /etc/apt/sources.list.d/raspi.list` 删除/`#`注释掉原文，复制黏贴，保存`Ctrl+O`+回车+`Ctrl+X`退出
   3. `sudo apt-get update`

## 文件共享服务搭建

### 安装ntfs-3g

> 要挂载`ntfs`格式的硬盘，必须有`ntfs-3g`模块的支持，没有的话，只能读不能写。

1. 看系统有没有自带`apt search ntfs-3g`,有就直接下一步
2. 或者直接安装
   * `sudo apt-get install -y ntfs-3g` 
   * 加载内核支持`modprobe fuse`
3. 或者直接将硬盘格式化为ext4格式：`mkfs.ext4 /dev/sda1`

{{< admonition tip "note" false >}}
实际上没有安装。
{{< /admonition >}}

### 硬盘挂载

* 查看硬盘挂载情况：`df -h` 一般会自动挂载到`/media/pi`，
* 将硬盘挂载到指定目录，跳到步骤5

> 如果是新硬盘或着有想删掉的分区

1. 查看外接硬盘分区表：`sudo fdisk -l`,一般是`/dev/sda`，
   * 如果看不到硬盘，可能电压不足，硬盘要外接电源
   * 如果出现`Partition 1 does not start on physical sector boundary`情况，可能问题不大，不影响使用，将起始位置设置*为8的倍数的*任何逻辑扇区。
   1. 备份
   2. 结果1：`cat /sys/block/sdb/queue/optimal_io_size`
   3. 结果2：`cat /sys/block/sdb/alignment_offset`
   4. 通过公式计算：（结果1+结果2）/512=结果3
   5. 重新分区,删除所有分区
      * `parted /dev/sda`
      * `rm 1...`
   6. `mkpart primary ext4 结果3s 10240G`
   7. `sudo fdisk -l`->跳到2.4

   {{< admonition tip "note" false >}}
   实际上直接从2.2开始到结束，使用了fdisk删掉所有分区，默认新建分区...好像也不报这个警告了。
   {{</admonition>}}
2. 对外接硬盘进行分区：`sudo fdisk /dev/sda`

3. 出现`Command(m for help)`,使用指令进行相关操作（在输入 `w`命令执行写入之前，所有改动都是在内存中的，`Ctrl+C`随时退出重来。）

    ```
    fdisk 常用指令
    n: 添加新的分区
    p: 查看分区信息
    w: 保存退出
    q: 不保存退出
    d: 删除分区
    t: 改变分区类型
    ```

4. 格式化分区：`sudo mkfs -t ext4 /dev/sda1`
5. 挂载到指定目录：
   1. 创建想要挂载的目录：`mkdir  /home/pi/share`，目录随意，似乎一般放`/mnt/`下
   2. 使用mount将分区挂载到目录上：`sudo mount /dev/sda1 /home/pi/share`
   3. 赋予pi用户操作该目录的权限：`sudo chown pi:pi /home/pi/share`
   4. 挂载出错或者挂载错地方了或者卸载：`sudo umount /dev/sda1`
   5. 查看挂载情况：`df -h`,或者直接`cd ~/share`查看
6. 开机自动挂载，fstab方式
   1. 查看硬盘的UUID：`sudo blkid` 或者 `ls -l /dev/disk/by-uuid`,记下UUID
   2. 将硬盘信息添加到`/etc/fstab` 末尾
      1. `sudo nano /etc/fstab`
      2. ext4格式：`UUID=e3e1729e-5f30-41d3-ae87-7e1a6721baab     /home/pi/share   ext4  defaults   0   2`
      3. ntfs格式：`UUID=你的UUID   你的挂载目录   ntfs    gid=pi,uid=pi,dmask=002,fmask=113   0   0`
      4. 保存`Ctrl+O`+回车+`Ctrl+X`退出
7. 重启：`sudo reboot`

### 安装Samba

1. 安装：`sudo apt-get install samba samba-common-bin`

2. 修改配置文件：`sudo nano /etc/samba/smb.conf`

    ```
    [Share 或随便起个名]
        path = 共享的地址，硬盘挂载地址，例如/home/pi/share
        valid users = pi   # 可以访问的用户，如果开启了root用户权限，就加上root
        browseable = Yes # 可以让被人看到资源名称吗？
        writeable = Yes # 可写入
        writelist = pi # 可写用户列表，不写也无所谓
        create mask = 0644 # 新建文件权限
        directory mask = 0775 # 新建文件夹的权限
    ```

3. 保存`Ctrl+O`+回车+`Ctrl+X`退出
4. 测试语法`testparm`
5. 添加用户和密码：`sudo smbpasswd -a pi`
6. 启动Samba服务：`systemctl start samba`
7. 查看Samba服务状态：`systemctl status smbd`
8. 打开此电脑，下拉到网络，开启网络发现，未打开时 Windows 会在窗口顶部提示你。地址栏搜索`\\树莓派ip`，出现Share 文件夹（如果起的名字是Share）
9. 开机启动：`sudo systemctl enable smbd`
10. 映射网络驱动器：右击Share文件夹选择映射网络驱动器

## Aria2

### 安装及配置

1. 安装`aria2`：`sudo apt install aria2`
2. `mkdir -p ~/.config/aria2/`
3. 创建会话空白文件`touch ~/.config/aria2/aria2.session`
4. [一键脚本](https://github.com/P3TERX/aria2.sh)
5. 手动添加一个aria配置文件：`nano ~/.config/aria2/aria2.config`

    ```
    # set your own path
    dir=[yourpath]
    disk-cache=32M
    file-allocation=trunc
    continue=true

    max-concurrent-downloads=10

    max-connection-per-server=16
    min-split-size=10M
    split=5
    max-overall-download-limit=0
    #max-download-limit=0
    #max-overall-upload-limit=0
    #max-upload-limit=0
    disable-ipv6=false

    save-session=/home/pi/.config/aria2/aria2.session
    input-file=/home/pi/.config/aria2/aria2.session
    save-session-interval=60


    enable-rpc=true
    rpc-allow-origin-all=true
    rpc-listen-all=true
    #rpc-secret=secret
    #event-poll=select
    #rpc-listen-port=6800


    # for PT user please set to false
    enable-dht=true
    enable-dht6=true
    enable-peer-exchange=true

    # for increasing BT speed
    listen-port=51413
    #follow-torrent=true
    #bt-max-peers=55
    #dht-listen-port=6881-6999
    #bt-enable-lpd=false
    #bt-request-peer-speed-limit=50K
    peer-id-prefix=-TR2770-
    user-agent=Transmission/2.77
    seed-ratio=0
    #force-save=false
    #bt-hash-check-seed=true
    bt-seed-unverified=true
    bt-save-metadata=true
    bt-tracker=udp://62.138.0.158:6969/announce,udp://87.233.192.220:6969/announce,udp://111.6.78.96:6969/announce,udp://90.179.64.91:1337/announce,udp://51.15.4.13:1337/announce,udp://151.80.120.113:2710/announce,udp://191.96.249.23:6969/announce,udp://35.187.36.248:1337/announce,udp://123.249.16.65:2710/announce,udp://210.244.71.25:6969/announce,udp://78.142.19.42:1337/announce,udp://173.254.219.72:6969/announce,udp://51.15.76.199:6969/announce,udp://51.15.40.114:80/announce,udp://91.212.150.191:3418/announce,udp://103.224.212.222:6969/announce,udp://5.79.83.194:6969/announce,udp://92.241.171.245:6969/announce,udp://5.79.209.57:6969/announce,udp://82.118.242.198:1337/announce
    ```

6. 启动测试
  `aria2c --conf-path=/home/pi/.config/aria2/aria2.config`
  `ps aux | grep aria2`

7. 开机启动

   1. 创建`systemctl service`文件：`sudo nano /lib/systemd/system/aria2.service`

      ```
      [Unit]
      Description=Aria2 Service
      After=network.target

      [Service]
      User=pi
      Type=forking
      ExecStart=/usr/bin/aria2c --conf-path=/home/pi/.config/aria2/aria2.config

      [Install]
      WantedBy=multi-user.target
      ```

   2. 重新载入服务，并设置开机启动
    `sudo systemctl daemon-reload`
    `sudo systemctl enable aria2`

   3. 查看aria服务状态
    `sudo systemctl status aria2`

8. 启动，停止，重启aria服务
  `sudo systemctl（start、stop、restart） aria2`

### 前端AriaNg

1. 安装`nginx`：`sudo apt install nginx`
2. `cd /var/www/html`
3. 安装`AriaNg` 的前端

    ```BASH
    sudo wget https://github.com/mayswind/AriaNg/releases/download/1.1.7/AriaNg-1.1.7-AllInOne.zip
    ```

4. 解压：`unzip AriaNg-1.1.7-AllInOne.zip`
5. 查看前端页面 `ls`
6. 启动`nginx`：`systemctl start nginx`
7. 查看`nginx`状态：`systemctl status nginx`
8. 测试页面，在浏览器内输入树莓派IP地址，显示AriaNg页面
9. 设置开机启动：`systemctl enable nginx`
10. 进入`http://树莓派地址/#!/settings/ariang`，设置`Aria2 RPC密钥`，`secret`
11. [trackerlist](https://github.com/ngosang/trackerslist)

## Resilio Sync（BTSync）

### 安装

1. [官方帮助文档](https://help.resilio.com/hc/en-us/articles/206178924#manage-sync)
2. 根据文档选择下载了 deb 软件包，armhf 架构;直接在电脑上下载通过 sftp 传到树莓派
3. `sudo dpkg -i <resilio-sync.deb>`

### 操作

1. 使用 rslsync 用户设置开机启动：`sudo systemctl enable resilio-sync`
2. 改变用户：

    ```bash
    sudo usermod -aG user_group rslsync
    sudo usermod -aG rslsync user_name
    sudo chmod g+rw synced_folder
    ```

3. 浏览器打开`树莓派IP:8888`，进行配置，记住用户名和密码
4. 变 PRO，实际上太懒就还是普通版
    {{< admonition type=note title="方法一" open=false >}}
      1. 改系统时间：`sudo date -s 25/12/2023`
      2. 在网页配置界面试用 PRO 版，再将系统时间改回当前时间
    {{< /admonition >}}

    {{< admonition type=note title="方法二" open=false >}}
      1. 屏蔽验证证书 `sudo nano /etc/hosts` 添加`127.0.0.1 license.resilio.com`
      2. 在配置界面添加许可证

        ```
        btos1_eyJzIjogIm5xNjRmNGgrWDhxWHFiaWRodEtyTWdNcEVaMHdYaUJLbExwSWFwSDUxMzkyaHdqMFFpQWJOTTFGd2JGWGpJQ0FhV2w4V3ZidVpPenpITXZmNTRLWU1XVGVsVFZjS3JGQUpYMmVUMGttT2lCMm8zQ1BjVFd2VWlCLzRsTTlQN1FsalVSVzREMVFNUktWcEhwaG5IVzIvZmdqN05MaThmMnhNa1NDNERwRVVpQ0pXZHpPWXFIbW1OMFdOUEt6QWg4RkJoanNvUVNBa1hKN09WS0c5WElkVTczWnNlVWNxWW1WVUo0WDc0S05wYmlYZk0vemlsU2pIK0U5Y3ZMZ092YlplVjlCZXlveDNwT3pRYUJOeFNQT3FYWEpTRHE1bVRhRXMvWGppVGtJV21uVHJudVJnb01DZ0RBOHdBS3RIYjRvVkduWm1jeU8xNlhoQ3dvL2VkeHFxZz09IiwgImQiOiAiZXlKamF5STZJQ0ppZEc5ell6RmZaWGxLYW1ONVNUWkpRMHBJVlZoYVYxRldUakZsVldoWVZrWldNMUpXU2tWVVJURlZZVEZXZFZacVVsUmpNMUp5Wkcxd1FsUkVRVFZWUXpsMVZraGtUVk5wT0hsUFEzUjNUV3hrVjJOck1IaFVhelI1VXpOQ1QyUlhiR3RTYlVwb1ZtNUtURlJZVVRWVFJ6RXpWVVU1TVZwdVdtRlVWbVJ1VVd4Sk1sWXlXbkZsYm14b1ZWaHdlbEV3TVVKamJXaE5XVlYwZFZwVVJYZFRSbEpOVW5wU2JGbHBkRXBVVlRGSlYwUnNWMkV5ZEZSalZFRXdUbTVuTUdKck1WZE5SR2N3VmxWYVRsSnJNVzlqUlVwMFlVUlNlRTFJYkZaWmExRXhVbTFzTUUxSVpIZE5iR3hVWWxob1IyTkVSbE5sVlhkeVVsWm9OV0ZGUlROUmJWSnVWVEZDYUdKSVZrWkxNV3hZVmpGb2FtSjVPVWxSVlRsT1ltMTNNbE5zUlRCYWFsSlFWbXBCZUZRd1VrSlpha1Z5V20wMWRVNXVXblZSTTJnell6RldTazlGTVU1aFNHeFFUREE1TUZOc2JFWldTRkpXVlVaSk1sUkRkRkpXUjJoUlYwUlNjbGxyTlhGTlJHeFpXbmwwYjJGVk1VeFZWMVpVU3pCd2RFNTZVblpUYW14SVpEQnJlVkZ0TVVsalJGWXdWRlZTZEZreWVIcE9WbWhLV1Zkb1MxZEdSa1pqTUhSMVVsaFdlbE13VGpSaVJsSjVUVmR3ZFdOSGFFNWFlbFV3WW0xNGJFOVVhRzlUU0VWeVVWaE5NV1ZZVWxSTlIyTTVVRk5KYzBsRFNtcGFRMGsyU1VOS2JHVlZjSGRaYlRGaFpHdHNjV0l5Wkd4bFZYQTJXa2N4VDFKSFNYbFZiWGhLWVcwNWJsTlhOVTlPVjBwMFZHeEdhbUpVYUhCVVJVNUNZVlpyZVdSSVFtRlJNR3N5VTFWT1MyVkdWblJXVkVKVFltczFjbGt3V2s1TmJVVjNaVVYwYW1Wc1NsaFpla0pyVFVaRmVGRnRlRmRUUlRWNVUxZHNNMW93YkhSV2FsSnFVVEJyTWxOVlVrcGxSVFZGVmxSV1RsWkdhekJVVlZKQ1l6QnNSRk50Y0dwTk1VWndWREpzUW1GWFNqVk5WelZQVmtaYVVGbDZRbTloYkZwelZsZHNUVkV3Um5CYVJXaHpaREZ3VkZOVVdrcFJNSEF6VjJ4b1MyVnRTWGxPVjJocFVUQnNlbE5WVGt0a2JVNUpWVzV3U21GdE9XNWFXR3hMWWxkSmVXVkhkR0ZYUlhCVlYyeGtUMlZXY0ZsVlYyeFFZVlZHY0ZSdGMzaFVNVlpIVjJ4YVZGSkhVa05VV0hCVFYyeEdWbFZVUmxoU1ZGWkhWV3RXVTFWV1VrWk9WVnBYVFZWd1UxUlhjRk5VVmxsNFdqSnNUVkV3Um5CWmVrcFhZVWRTU1ZSWGJGQmhWVVkwV214bmQyTXdiRVJUYlhoc1UwVkdjRlF5YkVKbFZURlZWVlJHVUZaRlZYbFVNRkpDWkRKYVVsQlVNR2xtVVQwOUlpd2dJbWx1Wm04aU9pQjdJbTl5WkdWeVNXUWlPaUFpWVc1aGRXeG1ZVzUxYTJadWIydHViVzF6TURZNWNXbzNOQ0lzSUNKaGNpSTZJREFzSUNKbGVIQWlPaUF5TVRRMU9URTJPREF3TENBaVkzSmxJam9nTVRRNE1UUXlOek00Tnl3Z0luTjJZMGxrSWpvZ0luTjVibU5RY204eElpd2dJbU56ZENJNklDSnZMV2MxTlU1elNHTldWU0lzSUNKemRtTkRiMlJsSWpvZ0luTjVibU5RY204aUxDQWlZMmxrSWpvZ0ltZHFaV295YmlJc0lDSjBlWEJsSWpvZ0luQmxjbk52Ym1Gc0lpd2dJbTl3ZEhNaU9pQjdJbVp2YkdSbGNsTmxZM0psZENJNklDSkJWMFJOVHpSRlUxTkJUbGhCV2xKVE1rMHlORlpCTmtkQlIxSlhWVFJRTlZZaUxDQWljMlZoZEhNaU9pQXhmWDBzSUNKbGVIQWlPaUF5TVRRMU9URTJPREF3ZlE9PSJ9
        ```

    {{< /admonition >}}

5. 防止被墙，添加中继服务器`sudo nano /etc/hosts` 添加`52.216.226.34 config.getsync.com`
6. key 分享
   1. [编程随想电子书分享](https://github.com/programthink/books)
   2. [书格同步](https://new.shuge.org/foryou/resilio_sync/) 1.5T 的古籍：BK6VORSOAPPEHB4D522QZERIZQETO3WY3

    {{< admonition type=warning title="不知道还能不能用的但在其他博客看到的" open=false >}}
    * 每周一书(往期) BB63I5PBPBFDELAPXI6NTF47IPNZQAAJZ
    * 每周一书(本周) B4TDKK7OLYBW7OAG3ILYLQPLFK3QSTAS4
    * kindle字典 BH5NCZJGQRYCREGGGXCQTI3FDFYJ27UFQ
    * 经济学人 BYRRPM52YK6Z6TETDQITFXBV647XLCNIO
    * 不知道是啥的 BCWHZRSLANR64CGPTXRE54ENNSIUE5SMO
    {{< /admonition >}}

## 参考

1. [为什么选择树莓派而不是NAS?从零建立树莓派4b的家庭多功能终端](https://www.bilibili.com/video/BV1aZ4y1g7ay)
2. [树莓派入门记录 | XEON CHOW](https://enit.xyz/tech/raspberry-pi-101/)
3. [树莓派3B+ 远程下载服务器（Aria2）_宁静致远's 博客-CSDN博客](https://blog.csdn.net/kxwinxp/article/details/80288006)
4. [用树莓派 Raspberry Pi 远程下载 (aria2)](https://li-aaron.github.io/2019/01/aira2-on-raspberry/)
5. [树莓派安装samba_sunye8910的博客-CSDN博客_树莓派samba](https://blog.csdn.net/sunye8910/article/details/81177298)
6. [树莓成长记2：树莓派4B挂载硬盘_jorondo的博客-CSDN博客](https://blog.csdn.net/jorondo/article/details/104504408)
7. [树莓派Raspbian系统格式化挂载硬盘 - 蓝曈魅 - 博客园](https://www.cnblogs.com/magicbowie/p/10896703.html)
8. [解决centos使用parted命令分区出现的警告Partition 1 does not start on physical sector boundary_u011785648的专栏-CSDN博客](https://blog.csdn.net/u011785648/article/details/106390074/)
9. [新买的硬盘放入移动硬盘盒无法识别怎么办解决-移动硬盘教程-U盘量产网](https://www.upantool.com/jiaocheng/disk/13511.html)
10. [树莓派安装 Resilio Sync｜迅猛式教程 | ziPeiJun's Blog](https://zipeijun.github.io/cjgtih9l4000bseabdb1c86r8/)
11. [Raspberry-Pi4安装Resilio-Sync | Zerone](https://miaospi.cn/post/Raspberry-pi%E5%AE%89%E8%A3%85Resilio-Sync/)
12. [[教程]在树莓派上装resilio | 大专栏](https://www.dazhuanlan.com/2020/01/04/5e1062d7783c2/)

