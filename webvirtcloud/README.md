- Phần này thì mình cũng không có gì để giới thiệu nhiều, mình sẽ đi vào cấu hình luôn.

#### 1 install packet cần thiết.

- Chú ý: phần cấu hình này sẽ được cấu hình trên 1 máy ảo  đang chạy trên chính host mà ta vừa cấu hình trước đó

- Cài đặt 1 VM đặt tên là webvirtcloud


        http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm 
        
        yum -y install python-virtualenv python-devel libvirt-devel glibc gcc nginx supervisor python-lxml git python-libguestfs python-pip libvirt-python libxml2-python python-websockify
        
        pip install numpy
        
- Tạo thư mục
        
       mkdir /srv && cd /srv
       
       git clone https://github.com/retspen/webvirtcloud && cd webvirtcloud
       
       cp webvirtcloud/settings.py.template webvirtcloud/settings.py
       
- Thêm secret key vào file settings.py
        
        vi webvirtcloud/settings.py
        
    - Thêm bất kỳ kí tự nào trong dấu '' đều được, sau đó lưu lại.
    
        SECRET_KEY = ''
           
           
#### Bắt đầu cài đặt webvircloud
        
        virtualenv venv
        source venv/bin/activate
        venv/bin/pip install -r conf/requirements.txt
        cp conf/nginx/webvirtcloud.conf /etc/nginx/conf.d/
        venv/bin/python manage.py migrate
##### Cấu hình supervisor
        
        vi /etc/supervisord.conf
        
   - Thêm đoạn code sau vào cuối file.
   
            [program:webvirtcloud]
            command=/srv/webvirtcloud/venv/bin/gunicorn webvirtcloud.wsgi:application -c /srv/webvirtcloud/gunicorn.conf.py
            directory=/srv/webvirtcloud
            user=nginx
            autostart=true
            autorestart=true
            redirect_stderr=true
            
            [program:novncd]
            command=/srv/webvirtcloud/venv/bin/python /srv/webvirtcloud/console/novncd
            directory=/srv/webvirtcloud
            user=nginx
            autostart=true
            autorestart=true
            redirect_stderr=true


##### Edit lại file nginx.conf.

            vi /etc/nginx.conf

   - comment lại toàn bộ đoạn code sau.
      
            #    server {
            #        listen       80 default_server;
            #        listen       [::]:80 default_server;
            #        server_name  _;
            #        root         /usr/share/nginx/html;
            #
            #        # Load configuration files for the default server block.
            #        include /etc/nginx/default.d/*.conf;
            #
            #        location / {
            #        }
            #
            #        error_page 404 /404.html;
            #            location = /40x.html {
            #        }
            #
            #        error_page 500 502 503 504 /50x.html;
            #            location = /50x.html {
            #        }
            #    }
            
##### Edit lại file webvirtcloud.conf

            vi /etc/nginx/conf.d/webvirtcloud.conf   
            
   - Xóa toàn bộ code có trong file và thêm đoạn code sau vào.
   
            upstream gunicorn_server {
            #server unix:/srv/webvirtcloud/venv/wvcloud.socket fail_timeout=0;
            server 127.0.0.1:8000 fail_timeout=0;
            }
            server {
                listen 80;
            
                server_name servername.domain.com;
                access_log /var/log/nginx/webvirtcloud-access_log; 
            
                location /static/ {
                    root /srv/webvirtcloud;
                    expires max;
                }
            
                location / {
                    proxy_pass http://gunicorn_server;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
                    proxy_set_header Host $host:$server_port;
                    proxy_set_header X-Forwarded-Proto $remote_addr;
                    proxy_connect_timeout 600;
                    proxy_read_timeout 600;
                    proxy_send_timeout 600;
                    client_max_body_size 1024M;
                }
            }   
            

##### Phân quyền cho thư mục /srv/webvirtcloud

        chown -R nginx:nginx /srv/webvirtcloud
        
##### start và enable supervisor và nginx

        systemctl restart nginx && systemctl restart supervisord
        systemctl enable nginx && systemctl enable supervisord
     
     
##### Check status supervisord
    
        supervisorctl status

##### cài đặt packet hỗ trợ hết nối bảo mật
    
   - **Cài đặt cả trên webvirtcloud và host**
   
        yum -y install cyrus-sasl-gssapi.i686 cyrus-sasl-gssapi.x86_64 cyrus-sasl-md5.i686 cyrus-sasl-md5.x86_64 perl-Authen-DigestMD5.noarch -y
        
##### Cấu hình kết nối trên host

     vi /etc/sasl2/libvirt.conf
     
   - Tìm đến mech_list: 
   
        - thay vào đó là **digest-md5**
        
   - Tìm đến **sasldb_path: /etc/libvirt/passwd.db**
   
        - bỏ dấu # ở đầu đi
        
    
    vi /etc/libvirt/libvirtd.con
    
    listen_tls = 0
    listen_tcp = 1
    auth_tcp="sasl"

