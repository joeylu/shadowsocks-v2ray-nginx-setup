# shadowsocks-v2ray-nginx-setup
setup shadowsocks server over https 


首先用以下脚本架设shadowsock 和 v2ray-plugin

Shadowsocks + v2ray-plugin 服务端
已经有大佬写好了一键脚本, 直接运行命令即可, 请参考https://github.com/M3chD09/shadowsocks-with-v2ray-plugin-install
安装完成后配置文件的目录/etc/shadowsocks-libev/config.json

    {
        "server":"0.0.0.0",
        "server_port":443,
        "password":"password",
        "timeout":300,
        "method":"aes-256-gcm",
        "plugin":"v2ray-plugin",
        "plugin_opts":"server;cert=/path/cert.pem;key=/path/key.pem;host=yourhost;loglevel=none"
    }
默认配置文件是没有经过nginx转发, 直接连接SS服务端, 开启443端口, 所以需要证书. 如果经过nginx转发需要简单修改一下配置文件, 端口是nginx转发端口, 不再需要证书, 因为证书已经配置在nginx上了. 可能有路径.

    {
        "server":"127.0.0.1",
        "server_port":10001, //端口需要与nginx转发端口一致
        "password":"password", //ss密码
        "timeout":300,
        "method":"aes-256-gcm",//ss加密方式
        "plugin":"v2ray-plugin",
        "plugin_opts":"server;path=/sspath;loglevel=none" //path需要与nginx配置路径一致
    }
重新启动ss service shadowsocks restart

该脚本通过certbot已经安装好域名的证书

不过为了让接下来安装的nginx无脑安装，我们接下来还要再reinstall一次
    sudo add-apt-repository ppa:certbot/certbot
    sudo apt-get update
    sudo apt-get install python-certbot-nginx nginx

然后 sudo nano /etc/nginx/conf.d/shadowsocks.conf 建立一个新的配置档

    server {
        listen       80;
        server_name  youhost.com; # 换成你自己的域名

    location /sspath { # v2ray插件配置的path
        proxy_pass                  http://127.0.0.1:10001; # shadowsocks服务port
        proxy_redirect              off;
        proxy_http_version          1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host       $http_host;
        }
    }

    sudo certbot --nginx --agree-tos --no-eff-email --email xxx@xxx.com

运行时会问你要不要reinstall ssl，选择yes
运行时会问你要不要redirect，选择2 （要）

最后 sudo nginx -s reload


服务器配置完成

安卓客户端安装方式
https://www.xpath.org/blog/0015661425086431d7c969c88854e5294d6b8ee4efc3c13000

windows客户端安装方式
https://lala.im/6434.html


原文
https://www.wervps.com/we/2891.html


