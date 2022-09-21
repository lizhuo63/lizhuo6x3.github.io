# 配置解读

```bash
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

#=========== 以上的 暂不关心 ============#

#静态资源服务 
upstream static_server_pool {
    #监听91端口的服务
    server 127.0.0.1:91 weight=10;
}

		# 80端口的 '至尊' 服务器
    server {
        listen       80;
        server_name  localhost;
        ssi on;
        ssi_silent_errors on;

        #通过 '/' 映射到 index[应用主页]
        location / {
            alias G:/myProject/;
            index index.html;
        }  
        # 走名为 http://static_server_pool 的代理通道
        location /static/company/ {
           proxy_pass http://static_server_pool;
        }
        location /static/teacher/ {
           proxy_pass  http://static_server_pool;
        }

        #静态资源，包括系统所需要的图片，js、css等静态资源
        location /static/js/ { 
            alias   G:/myProject/js/;
        } 
        location /static/img/ {
           alias  G:/myProject/static/img/;
        }
        location /static/css/ {
           alias  G:/myProject/static/css/;
        }
        location /static/plugins/{
           alias  G:/myProject/static/plugins/;
           #处理跨域的资源请求
           add_header Access-Control-Allow-Origin *;
           add_header Access-Control-Allow-Credentials true;
           add_header Access-Control-Allow-Methods GET;
        }
    }

    #静态页面资源
    server {
        listen       91;
        server_name  localhost;

         location /static/company/ {
           alias  G:/myProject/static/company/;
        }

        location /static/teacher/ {
           alias  G:/myProject/static/teacher/;
        }
        #通过 '/static/stat/' 映射到 'G:/myProject/static/stat/' 目录，匹配具体的文件名进行访问
        location /static/stat/ {
           alias  G:/myProject/static/stat/;
        }
    }
}
```

