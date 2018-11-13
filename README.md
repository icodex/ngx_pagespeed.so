# ngx_pagespeed.so
nginx dynamic module ngx_pagespeed.so

In NGINX 1.9.11 onwards a new way of loading modules dynamically has been introduced.
There are some ngx_pagespeed.so, which have been compiled, you can download one and load it dynamically for your nginx.

## About Pagespeed

ngx_pagespeed is an nginx moudle created by Google to help Make the Web Faster by rewriting web pages to reduce latency and bandwidth. More at [pagespeed/ngx_pagespeed](https://github.com/pagespeed/ngx_pagespeed)

## Installation

```cmd
$ mkdir ~/tmp
$ cd ~/tmp
$ git clone https://github.com/icodex/ngx_pagespeed.so.git
$ nginx_version=`echo $(nginx -v 2>&1) | cut -c22-`
$ sudo mkdir /usr/share/nginx/modules
$ cp ~/tmp/ubuntu_x86_64/nginx-stable-${nginx_version}/ngx_pagespeed.so /usr/share/nginx/modules/
```

### Load Module

edit `/etc/nginx/nginx.conf`, e.g.

```conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;

load_module modules/ngx_pagespeed.so;

events {
        worker_connections 768;
        # multi_accept on;
}

http {
    ...
}
```


### Test 

run `sudo nginx -t`, if you get output as follow, that means it works

```cmd
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

## Usage

```cmd
$ sudo mkdir /var/cache/ngx_pagespeed -p
$ sudo chown www-data:www-data -R /var/cache/ngx_pagespeed
```

edit `/etc/nginx/sites-enabled/default`, e.g.

```conf
server {
    listen 80;
    
    pagespeed on;
    pagespeed FileCachePath /var/cache/ngx_pagespeed;

    root /var/www/html;
    ...
}

```

## How to compile your ngx_pagespeed module manually

```cmd
#[check nginx's site(http://nginx.org/en/download.html) for the latest version]
NGINX_VERSION=1.14.1
#[check the release notes(https://www.modpagespeed.com/doc/release_notes) for the latest version]
NPS_VERSION=1.13.35.2-stable

mkdir -p /build/nginx-6hl6ty/nginx-${NGINX_VERSION}/debian/modules

# ngx_pagespeed
cd /build/nginx-6hl6ty
wget --no-check-certificate https://github.com/apache/incubator-pagespeed-ngx/archive/v${NPS_VERSION}.zip
unzip v${NPS_VERSION}.zip
nps_dir=$(find . -name "*pagespeed-ngx-${NPS_VERSION}" -type d)
cd "$nps_dir"
NPS_RELEASE_NUMBER=${NPS_VERSION/beta/}
NPS_RELEASE_NUMBER=${NPS_VERSION/stable/}
psol_url=https://dl.google.com/dl/page-speed/psol/${NPS_RELEASE_NUMBER}.tar.gz
[ -e scripts/format_binary_url.sh ] && psol_url=$(scripts/format_binary_url.sh PSOL_BINARY_URL)
wget --no-check-certificate ${psol_url}
tar -xzvf $(basename ${psol_url})  # extracts to psol/
# copy to module path
cp -a /build/nginx-6hl6ty/${nps_dir} /build/nginx-6hl6ty/nginx-${NGINX_VERSION}/debian/modules/pagespeed

# nginx
cd /build/nginx-6hl6ty
wget --no-check-certificate http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
tar -xvzf nginx-${NGINX_VERSION}.tar.gz
cd nginx-${NGINX_VERSION}/
./configure --with-cc-opt='-g -O2 -fdebug-prefix-map=/build/nginx-6hl6ty/nginx-${NGINX_VERSION}=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-compat --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_geoip_module=dynamic --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-stream_ssl_preread_module --with-mail=dynamic --with-mail_ssl_module --add-dynamic-module=/build/nginx-6hl6ty/nginx-${NGINX_VERSION}/debian/modules/pagespeed
make
cp objs/ngx_pagespeed.so /usr/lib/nginx/modules/
```
