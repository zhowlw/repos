- 下载介质（如果不行，手动下载并上传至服务器即可）
[Privoxy下载](https://www.privoxy.org/sf-download-mirror/Sources/)
```
# wget http://www.privoxy.org/sf-download-mirror/Sources/3.0.26%20%28stable%29/privoxy-3.0.26-stable-src.tar.gz
# tar -zxvf privoxy-3.0.26-stable-src.tar.gz
```
- centos7 下需要安装相关的开发工具包
```
# yum groupinstall "Development Tools"
```
- 安装privoxy(编译安装)
```
1. 创建privoxy用户(不建议使用root用户启动)
# sudo useradd privoxy -r -s /usr/sbin/nologin

2.编译
# autoheader
# autoconf
# ./configure
gcc -c -pipe -O2   -pthread -Wall   actions.c -o actions.o
gcc -c -pipe -O2   -pthread -Wall   cgi.c -o cgi.o
gcc -c -pipe -O2   -pthread -Wall   cgiedit.c -o cgiedit.o
gcc -c -pipe -O2   -pthread -Wall   cgisimple.c -o cgisimple.o
gcc -c -pipe -O2   -pthread -Wall   deanimate.c -o deanimate.o
gcc -c -pipe -O2   -pthread -Wall   encode.c -o encode.o
gcc -c -pipe -O2   -pthread -Wall   errlog.c -o errlog.o
gcc -c -pipe -O2   -pthread -Wall   filters.c -o filters.o
filters.c: In function ‘match_sockaddr’:
filters.c:215:42: warning: ‘address_port’ may be used uninitialized in this function [-Wmaybe-uninitialized]
    if (*netmask_port && *network_port != *address_port)
                                          ^
filters.c:182:17: warning: ‘addr_len’ may be used uninitialized in this function [-Wmaybe-uninitialized]
    unsigned int addr_len;
                 ^
filters.c:215:25: warning: ‘network_port’ may be used uninitialized in this function [-Wmaybe-uninitialized]
    if (*netmask_port && *network_port != *address_port)
                         ^
filters.c:215:8: warning: ‘netmask_port’ may be used uninitialized in this function [-Wmaybe-uninitialized]
    if (*netmask_port && *network_port != *address_port)
        ^
filters.c:210:20: warning: ‘netmask_addr’ may be used uninitialized in this function [-Wmaybe-uninitialized]
       netmask_addr += 12;
                    ^
gcc -c -pipe -O2   -pthread -Wall   gateway.c -o gateway.o
gcc -c -pipe -O2   -pthread -Wall   jbsockets.c -o jbsockets.o
gcc -c -pipe -O2   -pthread -Wall   jcc.c -o jcc.o
gcc -c -pipe -O2   -pthread -Wall   list.c -o list.o
gcc -c -pipe -O2   -pthread -Wall   loadcfg.c -o loadcfg.o
gcc -c -pipe -O2   -pthread -Wall   loaders.c -o loaders.o
gcc -c -pipe -O2   -pthread -Wall   miscutil.c -o miscutil.o
gcc -c -pipe -O2   -pthread -Wall   parsers.c -o parsers.o
gcc -c -pipe -O2   -pthread -Wall   ssplit.c -o ssplit.o
gcc -c -pipe -O2   -pthread -Wall   urlmatch.c -o urlmatch.o
gcc -c -pipe -O2   -pthread -Wall   client-tags.c -o client-tags.o
gcc -c -pipe -O2   -pthread -Wall   pcrs.c -o pcrs.o
gcc  -pthread -o privoxy actions.o cgi.o cgiedit.o cgisimple.o deanimate.o encode.o errlog.o filters.o gateway.o jbsockets.o jcc.o list.o loadcfg.o loaders.o miscutil.o parsers.o ssplit.o urlmatch.o client-tags.o  pcrs.o   -lnsl  -lpcre -lpcreposix   
grep -v '^#MASTER#' default.action.master > default.action
3. 安装privoxy
# make install
Creating directories, and preparing Privoxy 3.0.26 stable installation
chmod 0755 ./mkinstalldirs
mkdir /usr/local/etc/privoxy
mkdir /usr/local/etc/privoxy/templates
mkdir /var/log/privoxy
Installing privoxy executable to /usr/local/sbin
/usr/bin/install -c -m 0755  privoxy /usr/local/sbin
mkdir /usr/local/share/doc
mkdir /usr/local/share/doc/privoxy
mkdir /usr/local/share/doc/privoxy/user-manual
mkdir /usr/local/share/doc/privoxy/faq
mkdir /usr/local/share/doc/privoxy/developer-manual
mkdir /usr/local/share/doc/privoxy/man-page
mkdir /usr/local/share/doc/privoxy/images
Installing FAQ, Manual, and other docs to /usr/local/share/doc/privoxy
Installing man page to /usr/local/share/man/man1/privoxy.1
/usr/bin/install -c -m 0644 privoxy.1  /usr/local/share/man/man1/privoxy.1
Rewriting config for this installation
sed 's+^confdir \.+confdir /usr/local/etc/privoxy+' config | \
sed 's+^logdir \.+logdir /var/log/privoxy+' >config.tmp
mv config.updated config
Installing templates to /usr/local/etc/privoxy/templates
Warning: Setting group owner to privoxy
Installing configuration files to /usr/local/etc/privoxy
Installing fresh default.action
Installing fresh default.filter
Creating logfiles in /var/log/privoxy
Installing generic init script to /etc/init.d/privoxy
Please check that the PATHs are correct, and edit if needed.
rm -f config.base config.tmp
Privoxy 3.0.26 stable installation succeeded!
The Privoxy configuration files have been installed in /usr/local/etc/privoxy
```
- 更改并启动privoxy服务
```
1. 更改配置文件(default 8118)
# vi /usr/local/etc/privoxy/config
127.0.0.1:8118
forward-socks5t   /               127.0.0.1:1080 (ssr的local_port)
2. 启动服务
# systemctl daemon-reload && systemctl start privoxy
● privoxy.service - LSB: Start privoxy at boot time
   Loaded: loaded (/etc/rc.d/init.d/privoxy; bad; vendor preset: disabled)
   Active: active (running) since Sat 2019-11-23 21:56:22 CST; 6s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 10602 ExecStart=/etc/rc.d/init.d/privoxy start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/privoxy.service
           └─10616 privoxy --pidfile /var/run/privoxy.pid --user privoxy /usr/local/etc/privoxy/config

Nov 23 21:56:21 k8s-master03 systemd[1]: Starting LSB: Start privoxy at boot time...
Nov 23 21:56:22 k8s-master03 privoxy[10602]: Starting Privoxy, OK.
Nov 23 21:56:22 k8s-master03 systemd[1]: Started LSB: Start privoxy at boot time.
# netstat -anltp | grep 8118
tcp        0      0 127.0.0.1:8118          0.0.0.0:*               LISTEN      29542/privoxy  
3. 测试internet联通性
```
- curl测试
curl google.com
- 我自己搞了个k8s-repo,所以我直接yum makecache
# yum makecache
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
docker-ce-edge                                                                                                                                | 3.5 kB  00:00:00     
https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64/repodata/repomd.xml: [Errno 12] Timeout on https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64/repodata/repomd.xml: (28, 'Operation timed out after 30001 milliseconds with 0 out of 0 bytes received')
Trying other mirror.
kubernetes/signature                                                                                                                          |  454 B  00:00:00     
Retrieving key from https://packages.cloud.google.com/yum/doc/yum-key.gpg
Importing GPG key 0xA7317B0F:
 Userid     : "Google Cloud Packages Automatic Signing Key <gc-team@google.com>"
 Fingerprint: d0bc 747f d8ca f711 7500 d6fa 3746 c208 a731 7b0f
 From       : https://packages.cloud.google.com/yum/doc/yum-key.gpg
Is this ok [y/N]: y
Retrieving key from https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
kubernetes/signature                                                                                                                          | 1.4 kB  00:00:15 !!! 
(1/7): docker-ce-edge/x86_64/updateinfo                                                                                                       |   55 B  00:00:02     
(2/7): docker-ce-edge/x86_64/primary_db                                                                                                       |  41 kB  00:00:02     
(3/7): docker-ce-edge/x86_64/filelists_db                                                                                                     |  20 kB  00:00:07     
(4/7): docker-ce-edge/x86_64/other_db                                                                                                         |  80 kB  00:00:04     
(5/7): kubernetes/filelists                                                                                                                   |  20 kB  00:00:06     
(6/7): kubernetes/primary                                                                                                                     |  59 kB  00:00:06     
(7/7): kubernetes/other                                                                                                                       |  39 kB  00:00:02     
kubernetes                                                                                                                                                   430/430
kubernetes                                                                                                                                                   430/430
kubernetes                                                                                                                                                   430/430
Metadata Cache Created
```
