##版本区分插件是利用nginx服务来映射项目方便快捷替换项目版本测试

###1.nginx的配置


```

server {

        listen 80;
        server_name  localhost;
        index index.php index.html;
        #设置你的项目名称
        set $project "version";

        #HTTP版本配置
        if ( $http_server_code_version = "" ) { 
                set $http_server_code_version "master";
        }
        
        location /server-version {
                try_files $uri $uri/ /index.php;
        }
        
        #设置项目目录
        set $rootPath "/www/project/${project}/${http_server_code_version}";

        #php配置
        location ~ \.php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            #SERVER_CODE_VERSION 是php用来区分你的版本号的必须配置
            fastcgi_param  SERVER_CODE_VERSION $http_server_code_version;
            include        fastcgi_params;
        }

        root $rootPath;
                
        #之后的配置根据项目自定义
        location / {
                  # Redirect everything that isn't a real file to index.php
                try_files $uri $uri/ /index.php?$args;
        }

    }

```

###2.使用方式
* 在http header 头部中设置 server-code-version 来控制版本（默认是master版本）
* 通过在你的 http://域名/server-version 来设定你需要的版本这个方式是此控件主要操作方式