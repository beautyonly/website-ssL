website-ssl.sh：低门槛跨入Https大门
一、简介
网站https化已是大势所趋，个人blog也都可以把https玩儿起来！

Jerry Qu大神研究的很深，我把自己的操作步骤整合了一下，做成一个小工具，也许对大家有用！

二、使用方式
1、下载
# 下载工具
curl -so website-ssl.sh https://github.com/beautyonly/website-ssL.git/website-ssl.sh
chmod 0755 website-ssl.sh
没错，就这么下载了就能用了！当然，github源文件的下载，你也可以用你熟悉的任何方式！

**注意：**此工具会使用到openssl命令，请务必保证你的机器上已安装此工具！

2、配置
2.1 wsl.cnf.sh的配置
首次执行website-ssl.sh的时候，工具会自动在当前目录下创建配置文件：wsl.cnf.sh

./website-ssl.sh
得到结果：

您的配置文件「wsl.cnf.sh」配置不正确或还未进行配置，请检查！
你可用任意编辑工具打开wsl.cnf.sh文件，针对头部的如下几个配置项进行按需配置：

# ************************ 配置区域 START ******************************
# 你的ssl主目录位置
ssl_dir="/home/work/www/ssl"
# nginx中配置的，给 Let's Encrypt 验证用的
challenges_dir="/home/work/www/challenges/"
# 按照你的需求进行配置，多个域名用空格分开
websites="your-baidu.com www.your-baidu.com"
# ************************ 配置区域 END ********************************
2.2 nginx conf文件的配置
本工具是用Let's Encrypt来实现的https，所以证书的申请需要一个域名验证的过程； 也就是需要对目标站点的Nginx增加一个location，形如：

# CA认证
location ^~ /.well-known/acme-challenge/ {
    # 注：这里的$challenges_dir请替换成你自己的真实目录，如：/home/work/www/challenges/
    alias $challenges_dir;
    try_files $uri =404;
}
3、使用
# 直接执行脚本，获取帮助信息
./website-ssl.sh
结果：

    网站ssl自动化工具（v1.0）使用方法:

    usage: ./website-ssl.sh -v | csr | pem | nginx | renew | crontab | upgrade
    -v        查看工具的版本号
    csr       根据域名配置生成csr证书文件（For pem）
    pem       生成 Let's Encrypt 认可的pem证书文件
    nginx     获取nginx配置文件Demo
    renew     更新ssl证书文件
    crontab   自动更新pem证书文件的crontab任务
    upgrade   升级「website-ssl.sh」工具到最新版
4、实际使用案例
step1：创建pem文件
./website-ssl.sh pem
**注：**这一步会自动为我们创建domain.key文件和ssl-encrypt.pem文件

step2：获取nginx配置的Demo
./website-ssl.sh nginx
**注：**如果自己知道怎么配置nginx，这一步都可以忽略

step3：配置自己的nginx conf文件
核心就是配置一下这个：

server {
    listen 443 ssl;
    server_name  your-website.com;
    
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES
    ssl_prefer_server_ciphers on;
    ssl_certificate /home/work/www/ssl/ssl-encrypt.pem;
    ssl_certificate_key /home/work/www/ssl/domain.key;
    ssl_session_timeout 5m;
    
    ...
}
step4：重新载入nginx配置文件，https完美启用
service nginx reload
5、csr文件强制更新
此种情况，只针对「需要走https的域名有增减」的情况，我们可以手动执行命令来更新csr文件：

./website-ssl.sh csr
只更新csr文件是没用的，还需要再次更新pem文件：

./website-ssl.sh pem
6、ssl证书有效期问题
由Let's Encrypt机构颁发的证书，默认只有90天的有效期，所以我们需要有一个证书更新的机制：

./website-ssl.sh renew
此命令会重新生成签名证书，并重启nginx，使得站点的https寿命延续

下面是baidufe.com站点的证书自动更新日志：

[root@www-baidufe-com ssl]# ./website-ssl.sh renew
Parsing account key...
Parsing CSR...
Registering account...
Already registered!
Verifying baidufe.com...
baidufe.com verified!
Verifying static.baidufe.com...
static.baidufe.com verified!
Verifying www.baidufe.com...
www.baidufe.com verified!
Signing certificate...
Certificate signed!

ssl-encrypt.pem文件创建成功！
Reloading nginx!
ssl证书自动更新成功！Nginx已重启！
当然，我们完全可以不用手动来做这件事情，用crontab，省事又省心：

./website-ssl.sh crontab
把输入的内容，添加到root账号下的crontab列表中，即可：


# Let’s Encrypt 签发的证书只有90天有效期，可以设置为每个月自动更新
0 0 1 * * cd /home/work/www/ssl/ && ./website-ssl.sh renew >/dev/null 2>&1
到此，你可以开开心心的用了！

7、工具升级方法
但凡是个工具，都可能会有bug、或者新功能迭代，等等，所以，咱们可以通过如下方式，将工具升级到最新版本：

./website-ssl.sh upgrade
当然，要查看工具的版本号，也是有命令的：

./website-ssl.sh -v
#或者
./website-ssl.sh version
三、疑难杂症
如果使用bash website-ssl.sh出现shell脚本执行报语法错，那就试试这样：

chmod 0755 website-ssl.sh
./website-ssl.sh
四、意见反馈
Mail: beautytao@protonmail.com
