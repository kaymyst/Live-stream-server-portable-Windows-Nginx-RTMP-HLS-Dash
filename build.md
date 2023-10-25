Building nginx on windows

# requirements:
- Microsoft Visual C compiler. Microsoft Visual Studio 2022 with c++.
- [MSYS2](https://www.msys2.org/)
- Perl, if you want to build OpenSSL® and nginx with SSL support.  [Strawberry Perl](http://strawberryperl.com/).
- [PCRE](http://www.pcre.org/), [zlib](https://zlib.net/) and [OpenSSL](http://www.openssl.org/) libraries sources.

# 1 start msys with dev environment
- open VS2022 x64 Native Tools Command Prompt, 
- enter C:\msys64 folder

```
msys2_shell.cmd -use-full-path
```


```
mv /usr/bin/perl /usr/bin/perl.msys 
mv /usr/bin/link /usr/bin/link.msys
```
Then create a symlink to Strawberry Perl's executable in its place:

```
ln -s /c/strawberry/perl/bin/perl.exe /usr/bin/perl
```
# 2 get repos

git clone https://github.com/nginx/nginx

cd nginx

git switch branches/stable-1.24

git clone https://github.com/kaymyst/nginx-rtmp-module

# 3 - Fix nginx-rtmp-module 

the following is already been done in kaymyst repo so do not do this

nginx-rtmp-module/ngx_rtmp_core_module.c(611): warning C4456: declaration of 'sa' hides previous local declaration
nginx-rtmp-module/ngx_rtmp_core_module.c(506): note: see declaration of 'sa'

Both lines 611 and 506 are the same declaration, so when I compiled it, I commented out line 611

```
struct sockaddr  *sa;
```
I also had to do the modification suggested in [#1564 (comment)](https://github.com/arut/nginx-rtmp-module/pull/1564#issuecomment-1246535801)

Two instances of (1 << h.type) to (ngx_uint_t)(1 << h.type)

```
ngx_rtmp_flv_module.c
nginx-rtmp-module/ngx_rtmp_flv_module.c(508): error C2220: the following warning is treated as an er
ror
nginx-rtmp-module/ngx_rtmp_flv_module.c(508): warning C4334: '<<': result of 32-bit shift implicitly
 converted to 64 bits (was 64-bit shift intended?)
nginx-rtmp-module/ngx_rtmp_flv_module.c(521): warning C4334: '<<': result of 32-bit shift implicitly
 converted to 64 bits (was 64-bit shift intended?)
```

# 4 unzip libs
```
mkdir objs
mkdir objs/lib
cd objs/lib
tar -xzf ../../pcre2-10.42.tar.gz
tar -xzf ../../zlib-1.3.tar.gz
tar -xzf ../../openssl-3.0.11.tar.gz
```

# 5 configue
```
auto/configure \
    --with-cc=cl \
    --with-debug \
    --prefix= \
    --conf-path=conf/nginx.conf \
    --pid-path=logs/nginx.pid \
    --http-log-path=logs/access.log \
    --error-log-path=logs/error.log \
    --sbin-path=nginx.exe \
    --http-client-body-temp-path=temp/client_body_temp \
    --http-proxy-temp-path=temp/proxy_temp \
    --http-fastcgi-temp-path=temp/fastcgi_temp \
    --http-scgi-temp-path=temp/scgi_temp \
    --http-uwsgi-temp-path=temp/uwsgi_temp \
    --with-cc-opt=-DFD_SETSIZE=1024 \
    --with-pcre=objs/lib/pcre2-10.42 \
    --with-zlib=objs/lib/zlib-1.3 \
    --with-openssl=objs/lib/openssl-3.0.11 \
    --with-openssl-opt=no-asm \
    --with-http_ssl_module \
	  --with-http_flv_module \
    --with-http_gzip_static_module \
    --with-http_mp4_module \
    --with-http_secure_link_module \
    --with-http_slice_module \
    --with-http_v2_module \
    --with-mail \
    --with-mail_ssl_module \
    --with-stream \
    --with-stream_ssl_module \
    --add-module=nginx-rtmp-module
```

# 6 build
run the following
```
nmake
```
file will be in objs/nginx.exe
