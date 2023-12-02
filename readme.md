<img src="https://avatars.githubusercontent.com/u/56885001?s=200&v=4" alt="logo" width="130" height="130" align="right"/>

[![](https://img.shields.io/badge/TgChat-@UnOfficialV2board讨论-blue.svg)](https://t.me/unofficialV2board)

## 本分支支持的后端
 
 - [修改版XrayR](https://github.com/wyx2685/XrayR)
 - [修改版V2bX](https://github.com/wyx2685/V2bX)
 - [V2bX](https://github.com/InazumaV/V2bX)

## 原版迁移步骤

按以下步骤进行面板文件迁移：

    git remote set-url origin https://github.com/wyx2685/v2board  
    git checkout master  
    ./update.sh  


按以下步骤刷新设置缓存，重启队列:

    php artisan config:clear
    php artisan config:cache
    php artisan horizon:terminate



## aapanel部署指南
> 本文将教你如何使用aapanel进行部署
### 安装
1. 安装aaPanel 

如果是Centos系统
```
yum install -y wget && wget -O install.sh http://www.aapanel.com/script/install_6.0_en.sh && bash install.sh aapanel
```
如果是Ubuntu/Deepin系统
```
wget -O install.sh http://www.aapanel.com/script/install-ubuntu_6.0_en.sh && sudo bash install.sh aapanel
``` 
如果是Debian 系统
```
wget -O install.sh http://www.aapanel.com/script/install-ubuntu_6.0_en.sh && bash install.sh aapanel
```

安装完成后我们登陆 aaPanel 进行环境的安装。
2. 选择使用LNMP的环境安装方式勾选如下信息  
☑️ Nginx 任意版本  
☑️ MySQL 5.7  
☑️ PHP 8.1 （如果没看到8.1先不选，去App Store安装）
选择 Fast 快速编译后进行安装。

3. 安装扩展 
> aaPanel 面板 > App Store > 找到PHP 8.1点击Setting > Install extentions选择以下扩展进行安装
- redis
- fileinfo
- swoole4
- readline
- event

4. 解除被禁止函数
> aaPanel 面板 > App Store > 找到PHP 8.1点击Setting > Disabled functions 将以下函数从列表中删除
- putenv
- proc_open
- pcntl_alarm
- pcntl_signal

5. 添加站点  
>aaPanel 面板 > Website > Add site。  
>>在 Domain 填入你指向服务器的域名  
>>在 Database 选择MySQL  
>>在 PHP Verison 选择PHP-81 

6. 安装 Xborad  
>通过SSH登录到服务器后访问站点路径如：/www/wwwroot/你的站点域名。
>以下命令都需要在站点目录进行执行。
```
# 删除目录下文件
chattr -i .user.ini
rm -rf .htaccess 404.html index.html .user.ini
```
> 执行命令从 Github 克隆到当前目录。
```
git clone https://github.com/ZHQSM/v2board.git ./
```
> 执行命令安装依赖包以及V2board
```
sh init.sh
```
> 根据提示完成安装
7. 配置站点目录及伪静态
> 添加完成后编辑添加的站点 > Site directory > Running directory 选择 /public 保存。  
> 添加完成后编辑添加的站点 > URL rewrite 填入伪静态信息。
```
location /downloads {
}

location / {  
    try_files $uri $uri/ /index.php$is_args$query_string;  
}

location ~ .*\.(js|css)?$
{
    expires      1h;
    error_log off;
    access_log /dev/null; 
}
```
8. 配置守护进程
>Xboard的系统强依赖队列服务，正常使用XBoard必须启动队列服务。下面以aaPanel中supervisor服务来守护队列服务作为演示。  
- 1️⃣. aaPanel 面板 > App Store > Tools  
- 2️⃣. 找到Supervisor进行安装，安装完成后点击设置 > Add Daemon按照如下填写
- - 在 Name 填写 Xboard  
- - 在 Run User 选择 www  
- - 在 Run Dir 选择 站点目录 在 Start Command 填写 php artisan horizon 在 Processes 填写 1  

>填写后点击Confirm添加即可运行。

9. 配置定时任务#
aaPanel 面板 > Cron。
- 在 Type of Task 选择 Shell Script
- 在 Name of Task 填写 v2board
- 在 Period 选择 N Minutes 1 Minute
- 在 Script content 填写 php /www/wwwroot/路径/artisan schedule:run

根据上述信息添加每1分钟执行一次的定时任务。


### 开启webman
> 在上述安装的基础上开启webman提高性能

1. 配置php.ini
> 通过SSH登录到服务器后访问站点路径如：/www/wwwroot/你的站点域名。
```
cp /www/server/php/81/etc/php.ini cli-php.ini

sed -i 's/^disable_functions[[:space:]]*=[[:space:]]*.*/disable_functions=header,header_remove,headers_sent,http_response_code,setcookie,session_create_id,session_id,session_name,session_save_path,session_status,session_start,session_write_close,session_regenerate_id,set_time_limit/g' cli-php.ini

```
2. 添加守护进程
>下面以aaPanel中supervisor服务来守护队列服务作为演示。  
- 1️⃣. aaPanel 面板 > App Store > Tools 
- 2️⃣. 找到Supervisor进行安装，安装完成后点击设置 > Add Daemon按照如下填写
- - 在 Name 填写 webman
- - 在 Run User 选择 www  
- - 在 Run Dir 选择 站点目录 在 Start Command 填写 ```/www/server/php/81/bin/php -c cli-php.ini webman.php start``` 在 Processes 填写 1  
>填写后点击Confirm添加即可运行。

3. 修改伪静态
> 站点设置 > URL Rewrite(伪静态) 填入一下内容

```
location ~* \.(jpg|jpeg|png|gif|js|css|svg|woff2|woff|ttf|eot|wasm|json|ico)$ {

}
location ~ .* {
        proxy_pass http://127.0.0.1:7010;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Real-PORT $remote_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header Server-Protocol $server_protocol;
        proxy_set_header Server-Name $server_name;
        proxy_set_header Server-Addr $server_addr;
        proxy_set_header Server-Port $server_port;
  }
```

> 在此你的webman已经成功部署了

### 更新

1. 更新代码
> 通过SSH登录到服务器后访问站点路径如：/www/wwwroot/你的站点域名。
```
sh update.sh
```
2. 重启webman 守护进程(如果启用了webman)
- 1️⃣. aaPanel 面板 > App Store > Tools 
- 2️⃣. 找到Supervisor点击设置，找到名为webman的守护进程点击重启即可


### 注意
启用webman后做的任何代码修改都需要重启生效
