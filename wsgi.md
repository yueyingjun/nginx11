# 项目部署
## 1. 选择服务器
## 2. 安装python环境
- pip升级
    ```
    python -m pip install -U[upgrade] pip
    ```
## 3. 创建虚拟环境
- python -m venv myvenv
- 安装相应的包
    ```
    1. pip freeze > list.txt
    2. pip install -r list.txt
    ```
- 安装 ssh
    ```
    sudo apt-get update
    sudo apt-get install openssh-server
    ```
- 安装mysql
    ```
    sudo apt-get install mysql-server mysql-client
    ```
- 安装redis
    ```
    sudo apt-get install redis-server
    ```
- 修改配置文件
    ```
    DEBUG = False
    ALLOW_HOSTS=['*',]表示可以访问服务器的ip
    ```

## 4. 上传代码
- scp source username@ipaddress:/home/..

## 5. 导出，导入数据库
- 导出数据库
    ```
    mysqldump -h localhost -u root -p mydb > output
    ```
- 导入数据库
    ```
    1. 创建相应的数据库
    2. source datafile
    ```
- mysql 用户授权
    ```
    GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;

    FLUSH   PRIVILEGES;
    ```

## 6. 启动服务器

    ```
    python manager.py runserver
    ```

django manager.py runserver


## 7. UWSGI

- WSGI：Web服务器网关接口，英文为Python Web Server Gateway Interface，缩写为WSGI，是Python应用程序或框架和Web服务器之间的一种接口，被广泛接受
- uWSGI实现了WSGI的所有接口，是功能完善的服务器
- uWSGI代码完全用C编写
- 安装uwsgi
    ```
    pip install uwsgi
    ```
- 如果安装失败,请安装以下依赖
    ```
    sudo apt-get install libpython3.x-dev
    ```
- 配置uwsgi(在项目的根目录新建 uwsgi.ini 文件)
    ```
    [uwsgi]
    socket=ipaddress:port（和nginx服务器配合时，使用 socket）
    http=ipaddress:port（直接做web服务器，使用http）
    chdir=项目根目录
    wsgi-file=项目中wsgi.py文件的目录，相对于项目根目录
    processes=4
    threads=2
    master=True
    pidfile=uwsgi.pid
    daemonize=uswgi.log
    ```
- 启动：uwsgi --ini uwsgi.ini
- 停止：uwsgi --stop uwsgi.pid
- 重启：uwsgi --reload uwsgi.pid

## 8. nginx
- 负载均衡,反向代理的服务器
- 安装
    ```
    sudo apt-get install nginx
    ```
    + 查看版本：sudo nginx -v
    + 启动：sudo nginx
    + 停止：sudo nginx -s stop
    + 重启：sudo nginx -s reload
- 修改配置文件
    + /etc/nginx/nginx.conf
    + /etc/nginx/sites-enabled/default
    ```
    location / {
    include uwsgi_params;将所有的参数转到uwsgi下
    uwsgi_pass uwsgi服务器的ip与端口;
    }
    ```
    + 修改uwsgi.ini文件，启动socket，禁用http
    + nginx 代理其他设置
    ```
     large_client_header_buffers 4 16k;
     client_max_body_size 300m;
     client_body_buffer_size 128k;
     proxy_connect_timeout 600;
     proxy_read_timeout 600;
     proxy_send_timeout 600;
     proxy_buffer_size 64k;
     proxy_buffers   4 32k;
     proxy_busy_buffers_size 64k;
     proxy_temp_file_write_size 64k;
     root /var/www/project/static;
     index index.html index.htm;
    ```
- 静态文件设置
    + 设置静态文件位置
    ```
    location /static {
    alias /var/www/project/static/;
    }
    ```
    + 在指定位置创建目录结构，并修改权限
    ```
    sudo chmod -R 777 /var/www/project
    ```
    + 修改settings.py文件
    ```
    STATIC_ROOT='/var/www/project/static/'
    STATIC_URL='/static/
    ```
    + 采集所有静态文件到static_root指定的目录
    ```
    python manage.py collectstatic
    ```
    + 重启uwsgi和nginx



