---
layout: post
title: "nextcloud and remote debugging"
categories: nextcloud php-fpm xdebug
output:
  html_document:
    highlight: pygments
published: true
comments: false
tags: [nextcloud, php-fpm, debug, dev env]    
excerpt: How to set up a dev environement with nextcloud debugging
---
<div class="social-media-list">
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a>
<script type="IN/Share" data-url="{{ site.url }}{{ page.url }}"></script>
<div class="fb-share-button" data-href="{{ site.url }}{{ page.url }}" data-layout="button" data-size="small"><a target="_blank" href="https://www.facebook.com/sharer/sharer.php?u={{ site.url }}{{ page.url }}" class="fb-xfbml-parse-ignore">Partager</a></div>
</div>

How to set up a dev environement with nextcloud debugging
based on https://xdebug.org/docs/install
https://www.theaveragedev.com/xdebug-and-php-fpm-configuration/

Dev workstation side (windows10)
Get Visual Studio Code (1.34.0) and its extension "PHP Debug" (1.13.0)


Server side (WSL Debian 9)

php-fpm 7.0 installed with 
```
gui@SAGIS-09:$ sudo apt-get install php7.0-fpm
```
activate notice logs :
```
gui@SAGIS-09:$ sudo vi /etc/php/7.0/fpm/php-fpm.conf
```

error_log = /var/log/php7.0-fpm.log
log_level = notice


and dev tools 
```
gui@SAGIS-09:$ sudo apt-get install php7.0-dev
```



clone the Xdebug source , and build it
```
gui@SAGIS-09:/usr/src/$ sudo git clone https://github.com/xdebug/xdebug.git 
gui@SAGIS-09:/usr/src/$ cd xdebug/
gui@SAGIS-09:/usr/src/$ sudo ./rebuild.sh 
gui@SAGIS-09:/usr/src/$ sudo make install
```
You will have the library xdebug.so inside a directory

Configure php-fpm to get xdebug listening on server side
```
gui@SAGIS-09:/$ sudo vi /etc/php/7.0/fpm/php.ini
```
add config at the end / 9009 is used because nginx/apache httpserver could use the default port 9000 with fpm
```
[xdebug]
zend_extension="/usr/lib/php/20151012/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_host="localhost"
xdebug.remote_handler=dbgp
xdebug.remote_port=9009
xdebug.remote_autostart=1
```

Dev workstation side (windows10)
Open the configurations on DEBUG / PHP (the launch.json file)
```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for XDebug",
            "type": "php",
            "request": "launch",
            "port": 9009,
            // server -> local
            "pathMappings": 
            {
                "/var/www/html": "${workspaceRoot}/www",
//                "/app": "${workspaceRoot}/app",
                "/home/gui/nc": "${workspaceRoot}",
                "/apps": "${workspaceRoot}/apps",
                "/custom_apps": "${workspaceRoot}/custom_apps"                
            }
        },
        {
            "name": "Launch currently open script",
            "type": "php",
            "request": "launch",
            "program": "${file}",
            "cwd": "${fileDirname}",
            "port": 9009
        }
    ]
}
```