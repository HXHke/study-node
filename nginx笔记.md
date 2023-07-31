# nginx笔记

###什么是nginx

nginx是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/pop3/smtp服务

其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现比较好，

###正向代理和反向代理

* 正向代理

一个代理服务器帮客户端请求外部资源，再返回给客户端 例如VPN

* 反向代理

一个代理服务器代理多态服务器，外部访问服务器首先访问代理服务器，再跳往指定服务器

###负载均衡策略

1. 轮询

	依次循环的请求各个服务器

2. 加权轮询

	给指定服务器加权，这样就会有更多的请求访问到权数高的服务器

3.iphash

	iphash对客户端请求的ip进行hash操作，然后根据hash结果将同一个客户端ip请求分发给同一台服务器进行处理，可以解决session不共享的问题
	推荐使用redis做session共享

###动静分离

在我们软件开发中，有些请求是需要后台处理的，有些请求是不通过后台处理的（如：css、html、jpg、js等文件），这些不需要经过后台处理的文件称为静态文件。让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作。提高资源的响应的速度

###nginx常用命令

	cd /usr/local/nginx/sbin/
	./nginx 启动
	./nginx -s stop 停止
	./nginx -s quit 安全退出
	./nginx -s reload 重新加载配置文件

###配置nginx实现代理

![](https://img-blog.csdnimg.cn/20210719143750481.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MzEyOTg3,size_16,color_FFFFFF,t_70#pic_center)

配置源文件

	# 全局块
	------------------------------------------------------------------------------
	#user  nobody;
	worker_processes  1;
	
	#error_log  logs/error.log;
	#error_log  logs/error.log  notice;
	#error_log  logs/error.log  info;
	
	#pid        logs/nginx.pid;
	
	------------------------------------------------------------------------------
	
	# events块
	events {
	    worker_connections  1024;
	}
	
	# http块 
	http {
	------------------------------------------------------------------------------# http全局块
	    include       mime.types;
	    default_type  application/octet-stream;
	
	    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	    #                  '$status $body_bytes_sent "$http_referer" '
	    #                  '"$http_user_agent" "$http_x_forwarded_for"';
	
	    #access_log  logs/access.log  main;
	
	    sendfile        on;
	    #tcp_nopush     on;
	
	    #keepalive_timeout  0;
	    keepalive_timeout  65;
	
	    #gzip  on;        
	------------------------------------------------------------------------------    
	# server块
	server {
	# server全局块
	        listen       80;
	        server_name  localhost;
	
	        #charset koi8-r;
	
	        #access_log  logs/host.access.log  main;
	
	# location块
	        location / {
	            root   html;
	            index  index.html index.htm;
	        }
	
	        #error_page  404              /404.html;
	
	        # redirect server error pages to the static page /50x.html
	        #
	        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
	            root   html;
	        }
	
	        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
	        #
	        #location ~ \.php$ {
	        #    proxy_pass   http://127.0.0.1;
	        #}
	
	        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
	        #
	        #location ~ \.php$ {
	        #    root           html;
	        #    fastcgi_pass   127.0.0.1:9000;
	        #    fastcgi_index  index.php;
	        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
	        #    include        fastcgi_params;
	        #}
	
	        # deny access to .htaccess files, if Apache's document root
	        # concurs with nginx's one
	        #
	        #location ~ /\.ht {
	        #    deny  all;
	        #}
	}
		
	# 可以配置多个server块	
	
	}


####全局块
就是配置文件从头开始到events块之间的内容，主要设置的是影响nginx服务器整体运行的配置指令比如worker_process, 值越大，可以支持的并发处理量也越多，但是还是和服务器的硬件相关

####events块
events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。

上述例子就表示每个 work process 支持的最大连接数为 1024.
这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置

####http块
包括http全局块，以及多个server块

#####http全局块
http 全局块配置的指令包括文件引入、 MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

#####server块
* 这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。
* 每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机
* 而每个 server 块也分为全局 server 块，以及可以同时包含多个 location 块。

######server全局块
最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。

		#这一行表示这个server块监听的端口是80，只要有请求访问了80端口，此server块就处理请求
	  listen       80;
	  #  表示这个server块代表的虚拟主机的名字
	  server_name  localhost;

######location块
* 一个 server 块可以配置多个 location 块。
* 主要作用是根据请求地址路径的匹配，匹配成功进行特定的处理
* 这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

location详细文档[https://www.cnblogs.com/duhuo/p/8323812.html](https://www.cnblogs.com/duhuo/p/8323812.html)

server_name详细文档[https://blog.csdn.net/qq_40737025/article/details/85053164](https://blog.csdn.net/qq_40737025/article/details/85053164)

####代理示例

应用一

1. 首先启动服务器
2. 配置nginx
	1. 新建一个server块，在server全局块中监听80端口
	2. 在localion块中配置/路径请求代理到服务器的地址
	
				server {
			#	监听端口80 即当访问服务器的端口是80时，进入这个server块处理
			        listen       80;
			# server_name当配置了listen时不起作用        
			        server_name  localhost;
			
			# location后面代表访问路径 当是/ 请求时 代理到服务器的地址
			        location / {
			# 使用 proxy_pass（固定写法）后面跟要代理服务器地址            
			            proxy_pass http://192.168.80.102:8080;
			        }
			}    
		当访问http://192.168.80.102:80这个地址时，由于配置Nginx监听的是80端口，所以会进入这个server块进行处理，然后看你的访问路径，根据location块配置的不同路径进入对应的处理，由于配置了/请求，所以进入/的location处理，然后配置了proxy_pass，所以进行代理到指定的路径。
         
应用二

根据path目录，访问指定的服务器

1. 启动多台服务器
2. 配置nginx
	
			server {
		# 监听9001端口
		        listen       9001;
		# 进行路径匹配，匹配到edu代理到8081
		        location ~/edu/ {
		            proxy_pass http://192.168.80.102:8081;
		        }
		# 进行路径匹配，匹配到vod代理到8082
		        location ~/vod/ {
		            proxy_pass http://192.168.80.102:8082;
		        }
		}

####负载均衡示例

简单来说就是使用分布式的场景，将原先的一台服务器做成一个集群，然后将请求分发到各个服务器上，但是，如何将请求每次转发到不同的服务器呢，Nginx就可以做到。原来我们都是直接访问服务器，现在我们可以使用Nginx进行反向代理，然后我们访问Nginx，由Nginx将我们的请求分发到不同的服务器上，以实现负载均衡

1. 启动多台服务器
2. 配置nginx

		# 在http块中的全局块中配置
		# upstream固定写法 后面的myserver可以自定义
		upstream myserver{
		    server 192.168.80.102:8081;
		    server 192.168.80.102:8082;
		}
		
		# server配置
		    server {
		      # 监听80端口
		        listen 80;   
		    	#location块
		        location / {
		# 反向代理到上面的两台服务器 写上自定义的名称
		        proxy_pass http://myserver;
		        }
		    }

规则

1. 轮询（默认）

	每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除

2. weight权重

	weight 代表权重默认为 1,权重越高被分配的客户端越多

		upstream myserver { 
			server 192.168.80.102:8081 weight=1 ;
			server 192.168.80.102:8082 weight=2 ;
		}
		server {  
		    listen       80;  
		    location / {
		    proxy_pass http://myserver; 
		}

3. iphash

	每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决session问题

		#配置负载均衡的服务器和端口
		upstream myserver { 
			server 192.168.80.102:8081;
			server 192.168.80.102:8082;
		    ip_hash;
		}
		server {  
		    listen       80;  
		    location / {
		    proxy_pass http://myserver; 
		   }
		}

4. fair

	按后端服务器的响应时间来分配请求，响应时间短的优先分配。

		#配置负载均衡的服务器和端口
		upstream myserver {   
			server 192.168.80.102:8081;
			server 192.168.80.102:8082;
		    fair;
		}
		server {  
		    listen       80;   
		    location / {
		    proxy_pass http://myserver; 
		    }    
		}

###配置动静分离

* 将静态资源 css html js等和动态资源(jsp servlet)进行分开部署，我们可以将静态资源直接部署在专门的服务器上，也可以直接放在反向代理服务器上(Nginx)所在在的服务器上 然后动态资源还是部署在服务器上，如tomcat。
* 然后请求来的时候，静态资源从专门的静态资源服务器获取，动态资源还是转发到后端服务器上

![](https://img-blog.csdnimg.cn/20210719143822670.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MzEyOTg3,size_16,color_FFFFFF,t_70#pic_center)

配置

准备工作：在Linux的根目录下/的staticResource目录下创建两个文件夹，分别是www和image，在www目录下创建一个okc.html,在image目录下放一张ttt.jpg

实现效果，访问http://192.168.80.102:80/www/okc.html和http://192.168.80.102:80/image/ttt.img时可以成功访问资源


	server {
	        listen       80;
	    # 当访问路径带了www时，进入这个location处理，去/staticResource目录下对应的www目录     去找okc.html
	 #  即最终实现访问到这个路径
	  #  http://192.168.80.102:80/staticResource/www/okc.html
	        location /www/{
	            root   /staticResource/;
	            index  index.html index.htm;
	        }
	    # 跟上面一样
	        location /image/{
	            root  /staticResource/;
	      }   
	}  

root与alias区别与访问路径

* alias 实际访问文件路径不会拼接URL中的路径

		location ^~ /sta/ {  
		   alias /usr/local/nginx/html/static/;  
		}
请求：http://test.com/sta/sta1.html
实际访问：/usr/local/nginx/html/static/sta1.html 文件


* root 实际访问文件路径会拼接URL中的路径

	location ^~ /tea/ {  
	   root /usr/local/nginx/html/;  
	}

请求：http://test.com/tea/tea1.html
实际访问：/usr/local/nginx/html/tea/tea1.html 文件

###高可用集群

* 作为备用服务器，当主服务器宕掉后，配置的备用服务器自动切换，
* keepalived提供虚拟ip，对外我们访问的是虚拟ip，绑定了主备的ip

![](https://img-blog.csdnimg.cn/20210719143831670.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MzEyOTg3,size_16,color_FFFFFF,t_70#pic_center)