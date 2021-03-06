# dnmp
docker搭建lnmp环境，php 7.2 + nginx latest + mysql 5.7 + redis 4

## 前言
  使用前，默认你已经安装了 [docker](https://www.jianshu.com/search?q=docker%E5%AE%89%E8%A3%85&page=1&type=note) 和 [docker-compose](https://www.jianshu.com/p/f323aa0416da)

## 先使用阿里云镜像加速器
 
 [阿里云镜像加速器](https://dev.aliyun.com)
 
 进入之后登录，然后进到 [ 管理中心 ]，找到镜像加速器，里面有教你怎么配置本地的加速器，不再赘述
  
 ##  克隆此包
  
     git clone  https://github.com/chichoyi/dnmp.git

### 修改docker-compose.yml

 这里会说明一些参数
 
    nginx:
      image: nginx:latest
      container_name: nginx
      hostname: nginx
      ports:
        - "80:80"
      volumes:
        - /data/www:/www  # /data/www 这个是宿主机的目录，可以修改指向你的代码文件夹，比如 /mnt/hgfs/www
        - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro  # 这里是配置文件，不多，可以打开看看
        - ./nginx/conf.d/:/etc/nginx/conf.d/:ro
      links:
        - php:PHP-FPM  # php容器的别名，nginx发送给php需要用到的，默认即可
      networks:
        - lnmp
      
    php:
      build: ./phpfpm  # Dockerfile 文件地址
      container_name: php
      hostname: php
      ports:
        - "9000：9000"
      links:
        - mysql:mysql
        - redis:redis
      volumes:
        - /data/www:/www  # /data/www 这个是宿主机的目录，可以修改指向你的代码文件夹，比如 /mnt/hgfs/www
        - ./phpfpm/conf/:/usr/local/etc/php/conf.d
      networks:
        - lnmp
        
        # 注意，nginx和php挂载出来的宿主机目录一定要一致(/data/www)，不然没法访问php文件
    
    mysql:
      image: mysql:5.7
      container_name: mysql
      hostname: mysql
      ports:
        - "3306:3306"
      environment:
        - MYSQL_ROOT_PASSWORD=root  #数据库的初始化密码，这里可以指定修改成你自己的
        networks:
          - lnmp
            
    redis:
      image: redis:4
      container_name: redis
      hostname: redis
      ports:
        - "6379:6379"
      networks:
        - lnmp
        
    networks:
      lnmp:
        driver: bridge
   
   
 ## 修改完之后执行命令
  
    cd dnmp
    
    docker-compose build
    
    # 修改代码目录
    vi docker-compose.yml
    # 把42行和54行的 /Users/chenyinshan/code 修改为你本机的代码目录

    cp nginx/demo.conf nginx/conf.d/defualt.conf

    # vi defualt.conf   修改你自己的虚拟域名和目录,容器的目录默认是/www,
    
    docker-compose up -d
    
    //如果没有目录的话请自己创建相关目录
    
 第一次执行需要花点时间下载镜像
 php连接mysql需要在本机hosts文件添加域名映射，例如：
 
    127.0.0.1 mysql
    127.0.0.1 nginx
    127.0.0.1 php
    127.0.0.1 redis
 
 这样就不用去查找mysql的容器的IP了
  
    # docker-compose up -d 的时候可能一些服务没有跑起来，
    # 比如mysql，可能你给的dnmp目录权限不够，要给充足权限
    
    sudo chmod -R 777 dnmp
    
## 命令参考

    # 你自己修改了docker-compose文件或Dockerfile文件的话，请执行
    docker-compose build
    
    # 开启 dnmp 服务
    docker-compose up -d
    
    # 重启 dnmp 服务
    docker-compose restart
    
    # 关闭 dnmp 服务
    docker-compose down
    
    # 如果修改或增加了 nginx 服务配置
    docker restart nginx
 
 ## tip
 
   使用的都是官方的镜像，如有疑问，欢迎提出指正,
 
 ## contact me
 
 email: chichoyi@163.com
   
