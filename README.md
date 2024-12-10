# nginx-quic-libressl-brotli

Скопировано с [nginx-quic-libressl-src](https://aur.archlinux.org/packages/nginx-quic-libressl-src) и изменено с последних версий на 1.27.2, поскольку brotli версия требует её.

Сначала скачиваем этот репозиторий, делаем `makepkg -si` и получаем nginx-quic-libressl-brotli, но пока ещё без самого brotli.

Для установки brotli `paru -S nginx-mainline-mod-brotli`, он установит и создаст .so файлы.

Документация по [brotli](https://github.com/google/ngx_brotli)

После установки используем `mkdir /etc/nginx/modules/` и далее `sudo cp /usr/lib/nginx/modules/ngx_http_brotli_* /etc/nginx/modules/`

После этого мы будем иметь две .so в папке модулей, которые прописываем в шапку /etc/nginx/nginx.conf
```
load_module modules/ngx_http_brotli_filter_module.so;
load_module modules/ngx_http_brotli_static_module.so;
```

Далее в http прописываем включение:
```
brotli on;
brotli_comp_level 6;
brotli_static on;
brotli_types application/atom+xml application/javascript application/json application/vnd.api+json application/rss+xml
             application/vnd.ms-fontobject application/x-font-opentype application/x-font-truetype
             application/x-font-ttf application/x-javascript application/xhtml+xml application/xml
             font/eot font/opentype font/otf font/truetype image/svg+xml image/vnd.microsoft.icon
             image/x-icon image/x-win-bitmap text/css text/javascript text/plain text/xml;
gzip on;
gzip_comp_level 6;
```

Таким образом он будет работать только на необходимые типы данных;

Но ещё лучше использовать [zstd](https://nginx-extras.getpagespeed.com/modules/zstd/); пишем `paru -S nginx-mainline-mod-zstd`. После чего `sudo cp /usr/lib/nginx/modules/ngx_http_zstd_* /etc/nginx/modules/`. Дальше в nginx.conf:
```
load_module modules/ngx_http_zstd_filter_module.so;
load_module modules/ngx_http_zstd_static_module.so;
```
А также в http:
```
# Используем только brotli + zstd
gzip off;

brotli on;
brotli_comp_level 4;
brotli_static on;
brotli_types application/javascript application/json application/x-javascript
             text/css text/javascript text/plain text/xml image/svg+xml;

zstd on;
zstd_min_length 256; # no less than 256 bytes
zstd_comp_level 3;   # set the level to 3
zstd_types application/vnd.api+json application/vnd.ms-fontobject
           application/x-font-opentype application/x-font-truetype
           application/x-font-ttf application/xhtml+xml application/xml
           font/eot font/opentype font/otf font/truetype
           image/vnd.microsoft.icon image/x-icon image/x-win-bitmap;
```
