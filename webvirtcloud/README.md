## Mục Lục

--------------

### [1 Install packet](#1)

### [2 Install webvirtcloud](#2)

  - [2.1 Cấu hình suppervisord](#2.1)
  - [2.2 Cấu hình nginx](#2.2)
  - [2.3 Edit Webvirtcloud](#2.3)
  - [2.4 Phân quyền && restart](#2.4)
  - [2.5 Bảo mât](#2.5)

### [3 Tính năng cơ bản](#3)


- Phần này thì mình cũng không có gì để giới thiệu, mình sẽ đi vào cấu hình luôn.

#### <a name="1"><a/> 1 install packet cần thiết.

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
           
           
#### <a name="2"><a/>2 cài đặt webvircloud
        
        virtualenv venv
        source venv/bin/activate
        venv/bin/pip install -r conf/requirements.txt
        cp conf/nginx/webvirtcloud.conf /etc/nginx/conf.d/
        venv/bin/python manage.py migrate
        
##### <a name="2.1"><a/>2.1 Cấu hình supervisor
        
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


#### <a name="2.3"><a/>2.3 Edit lại file nginx.conf.

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
            
#### <a name="2.3"><a/>2.3 Edit lại file webvirtcloud.conf

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
            
#### <a name="2.4"><a/>2.4 Phân quyền && restart

   #### Phân quyền cho thư mục /srv/webvirtcloud

        chown -R nginx:nginx /srv/webvirtcloud
        
   #### start và enable supervisor và nginx

        systemctl restart nginx && systemctl restart supervisord
        systemctl enable nginx && systemctl enable supervisord
     
     
   #### Check status supervisord
    
        supervisorctl status


#### <a name="2.5"><a/>2.5 Bảo mật

   #### Cài đặt packet hỗ trợ hết nối bảo mật
    
   - **Cài đặt cả trên webvirtcloud và host**
   
            yum -y install cyrus-sasl-gssapi.i686 cyrus-sasl-gssapi.x86_64 cyrus-sasl-md5.i686 cyrus-sasl-md5.x86_64 perl-Authen-DigestMD5.noarch -y
        
   #### Cấu hình kết nối trên host

     vi /etc/sasl2/libvirt.conf
     
   - Tìm đến mech_list: 
   
        - Thay vào đó là **digest-md5**
        
  ![Selection_015](https://user-images.githubusercontent.com/19284401/55211854-57fadc00-5220-11e9-9836-fe4eea2ac5de.png)
        
   - Tìm đến **sasldb_path: /etc/libvirt/passwd.db**
   
      - bỏ dấu # ở đầu đi
      
  ![Selection_016](https://user-images.githubusercontent.com/19284401/55211855-57fadc00-5220-11e9-9145-7dfbc4c0e44f.png)
        
   - Cấu hình kết nối tcp
       
            vi /etc/libvirt/libvirtd.conf
    
            listen_tls = 0
            listen_tcp = 1
            auth_tcp="sasl"
    
   ![Selection_017](https://user-images.githubusercontent.com/19284401/55211856-58937280-5220-11e9-88f9-2f896a96aa8e.png)
   ![Selection_018](https://user-images.githubusercontent.com/19284401/55211857-58937280-5220-11e9-8c63-7e9573ad2aa6.png)
   
    
   -  Edit file libvirtd
    
            vi /etc/sysconfig/libvirtd
    
   - Bỏ commnent
   
         LIBVIRTD_ARGS="--listen"
         
   ![Selection_019](https://user-images.githubusercontent.com/19284401/55211858-58937280-5220-11e9-96c1-bd1df5f407e5.png)
        
        
- Restart libvirt
        
        systemctl restart libvirtd
        
   #### <a name="user"><a/>Tạo Tài khoản        
          
    - Tạo user kết nối tcp
        
            saslpasswd2 -a libvirt username
            
    - Import user to file password.db
    
            sasldblistusers2 -f /etc/libvirt/passwd.db
            
    - Test connect tcp

        virsh -c qemu+tcp://IP_host/system nodeinfo        
      
     ![Selection_020](https://user-images.githubusercontent.com/19284401/55212093-161e6580-5221-11e9-9c4a-f59a7e809bee.png)
     
    - Như vậy là đã kết nối tcp thành công.
    
          
   - Ok ta đã cấu hình webvirtcloud
    
   - Truy cập vào trang webvirtcloud
    
        http://ip_cua_webvirtcloud
    
   - Giờ ta sẽ đi cấu hình dhcp server cho hệ thống VM.
    


<a href="https://github.com/letran3691/AoHoa/tree/master/dhcp" rel="nofollow"> Cấu hình DHCP Server <a/>


   #### Vậy là ta đã cấu hình xong 1 hệ thống ảo hóa vừa và nhỏ cho 1 doanh nghiệp.

   - Giờ chúng ta sẽ tìm hiểu qua 1 về giao diện của webvirtcloud.
    
   - Như ở <a href="https://github.com/letran3691/Virtualization/tree/master/kvm#1.3" rel="nofollow"> **Topo** <a/>  các bạn cũng đã thấy toàn bộ VM trên 2 host KVM và KVM1  muốn ra được internet thì phải đi qua pfsense. Sẽ có 2 cách để các bạn truy cập vào webvirtcloud
    
   - 1 là cấu hình vpn trên pfsense để truy cập trực tiếp bằng ip của webvirtcloud.
    
   - 2 là NAT port trên pfsense và rồi truy vào webvirtcloud thông qua port NAT.
    
   ![Selection_001](https://user-images.githubusercontent.com/19284401/55217453-1f63fe00-5232-11e9-9f41-bbce0d25a8ac.png)

   - Như các bạn đã thấy ỏ đây mình đã NAT webvirtcloud
    
        - Ở đâu mình có NAT 2 port cho webvirtcloud 8088 là port web còn port 6080 là port novnc có tác dụng truy cập vào VM trực tiếp từ giao diện web.
        
#### <a name="3"><a/>3 Tính năng cơ bản. 
               
- Truy cập nào.

     - login 
        
     ![Selection_002](https://user-images.githubusercontent.com/19284401/55217880-29d2c780-5233-11e9-907d-aff44c459f69.png)
     
    - Tạo kết nối giữa webvirtcloud với các host KVM.
    
    ![Selection_003](https://user-images.githubusercontent.com/19284401/55218088-a82f6980-5233-11e9-9ada-910b006d678b.png)
   
    - Nhập các thông tin cần thiết
    
    ![Selection_004](https://user-images.githubusercontent.com/19284401/55218090-a82f6980-5233-11e9-84ff-ef804c115cb7.png)

     - Label: đăt tên cho host
     - FQDN / IP: nhập ip của host mà bạn muốn kết nội.
     - user - pasword bạn nhập tài khoàn mà các bạn đã tạo ở bước [Tạo tài khoản](#user)
     
- Như <a href="https://github.com/letran3691/Virtualization/tree/master/kvm#1.3" rel="nofollow">Topo<a/> ban đầu và trong hình các bạn cũng thấy hiện tại mình đang có 2 host KVM.
     
     ![Selection_005](https://user-images.githubusercontent.com/19284401/55218442-b631ba00-5234-11e9-817b-c51bcba99e8c.png)
     
- Tạo mới VM từ giao diện webvirtcloud.

    ![image](https://user-images.githubusercontent.com/19284401/55277581-8a95f900-5334-11e9-8ff8-28ff429204c8.png)
    
    - Chọn host sẽ tạo VM
    
     ![image](https://user-images.githubusercontent.com/19284401/55277587-9aadd880-5334-11e9-975a-dab2d2ec396d.png)
     
    - Các cách nào VM
    
    ![image](https://user-images.githubusercontent.com/19284401/55277620-f4160780-5334-11e9-8c4b-38fed649bc00.png)

     - **Flavor**: tạo từ 1 cấu hình cho có sẵn hoặc bạn cũng có thể tự tạo cho mình 1 Flavor mới.
     
     - **Custom**: Tự tạo VM theo cách của bạn.
     
        - Ở phần này các bạn chú ý của mình 2 phần đó là **HDD** và **NETWORK** 
        
            - HDD: nó sẽ list ra cho các bạn nhưng disk đã có sẵn trong host. Tạo tạo ra 1 VM mẫu sau đó clone disk của VM đó ra, đến bước này các bạn chỉ việc chọn disk vừa clone ra vậy là xog ko cần phải cài đặt gì thêm cả
            
            - NETWORK: nó sẽ list ra các network mà các bạn đã tạo trước đi để các bạn chọn.
            
            ![image](https://user-images.githubusercontent.com/19284401/55277689-6f77b900-5335-11e9-819a-885f631fbeda.png)
        
     - **Template** cũng khá giống với **Custom**
     
     - **XML** các bạn tham khảo ở phần XML của các VM có sẵn nhé.
     
     
    
     
- click vào **instances** các bạn sẽ thấy toàn bộ các VM đang có trên các host
     
     ![Selection_007](https://user-images.githubusercontent.com/19284401/55219744-49b8ba00-5238-11e9-90d2-a8ab01e2f5ad.png)
     
- Menu của 1 instance   
    
    ![Selection_008](https://user-images.githubusercontent.com/19284401/55219958-b8961300-5238-11e9-91be-2ada951f471b.png)
    
   - Tab **Power**
   
        - stop hoặc start 1 VM
        
   - Tab **Access**

        - dùng để kết nối noVNC   
        
   ![Selection_009](https://user-images.githubusercontent.com/19284401/55220297-8638e580-5239-11e9-9beb-709e96c2a43d.png)
   
     - Sau khi click vào console sẽ có 1 của sổ web, hiện lên để các bạn có thể đăng nhập vào VM, và thao tác với VM như bình thường.
     
    ![Selection_010](https://user-images.githubusercontent.com/19284401/55220299-86d17c00-5239-11e9-8a4f-a540db9ebd6a.png)
    
   - Tab **resize**
   
    ![Selection_011](https://user-images.githubusercontent.com/19284401/55220575-2858cd80-523a-11e9-85d9-1005b6bfa894.png)
    
    - đây là 1 tab cực hay. mình ko biết các công nghệ ảo hóa khác có làm được điều này ko.
        
        - Đó là thay đổi phần cứng như RAM vs CPU của VM khi nó vẫn đang chạy 
        
        -  để thực hiện được việc nào các bạn cần chú ý 
            
            - Mục **Maximum Alocaltion** phải cao hơn Mục **Current alocaltion**. điều này cần có sự tính toán từ trước là VM này có cần nâng cấp về mặt phần cứng hay ko?
            
    - Tab **Snapshot**
    
    ![Selection_013](https://user-images.githubusercontent.com/19284401/55221842-07de4280-523d-11e9-89c5-13a0036b0a82.png)
    
     - Tạo và revert snapshot cho VM. nếu bị VM đang chạy thì nó sẽ ẩn tính năng này đi. khi VM tắt thì tình năng này mới đc bật.
    
    ![Selection_015](https://user-images.githubusercontent.com/19284401/55222060-7fac6d00-523d-11e9-9929-5607a9f4fc85.png)
    ![Selection_016](https://user-images.githubusercontent.com/19284401/55222061-80450380-523d-11e9-9058-c39e3dae8558.png)
   
   - Tab **Setting**
   
   ![Selection_017](https://user-images.githubusercontent.com/19284401/55222203-e29e0400-523d-11e9-8434-81cb72ff3a2c.png)
   
     - đây là tab có nhiều tính năng nhất. Chúng ta sẽ đi qua 1 vài tab quan trọng. còn lại các bạn tìm hiểu dần dần.
  
        - Tab **Disk**
        
           ![image](https://user-images.githubusercontent.com/19284401/55277317-579e3600-5331-11e9-8d3c-f2d323f572f7.png)
           
           - Tab này có tác dụng để mount các disk có sẵn trong host 
           
           ![image](https://user-images.githubusercontent.com/19284401/55277342-a350df80-5331-11e9-948e-aac7bee64b1e.png)
                
             - Mặc định tab này bị ẩn khi VM đang chạy, tuy nhiên mình đã sửa lại code web để có thể mount trực tiếp khi VM đang chạy.

           - Tab **Network** 
           
            ![image](https://user-images.githubusercontent.com/19284401/55277414-7a7d1a00-5332-11e9-919d-cdadd78d0e3e.png)
            
             - Tab này sẽ hiện thị thông tin về NIC của VM như MAC, IP, Tên Network mà VM đang kết nối, vị trí của port mà VM đang cắm trên OVS.
             
             - Bạn cũng có thể thêm NIC cho VM tại đây (VM phải được shutdown)
             
           - Tab **Migrate** 
           
            ![image](https://user-images.githubusercontent.com/19284401/55277439-d34cb280-5332-11e9-9a2e-d3950202cde5.png)
            
             - Cài tên nói lên tất cả. Nó sẽ chuyển VM từ host này sang host khác trong khi VM vẫn đang chạy.
             
             - Chú ý: Nếu VM của bạn đang có 1 snapshot bạn sẽ nhận đc 1 thông báo lỗi là ko thể Migrate với 1 Snapshot
             
           - Tab **XML**
           
             ![image](https://user-images.githubusercontent.com/19284401/55277535-ef048880-5333-11e9-88c0-a5d76996238e.png)
             
             - Đây là Tab lưu nội dung câu hình của 1 VM, các bạn cũng có thể sửa đổi cấu hình trực tiếp trên file này(VM phải đặc shutdown)
             
- **Chú ý**: 1 số tính năng mặc định **webvirtcloud** ko có, mình đã sửa lại code để thêm 1 file tính năng đó nhé.          
             
             
- Vậy là chúng ta đã tìm hiểu xong công cụ quản lý KVM trên web. 

- Chúc các bạn thành công
            

#### Tài liệu tham khảo

   1)  https://github.com/retspen/webvirtcloud