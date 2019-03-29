
#### Cài đặt dhcp

        yum -y install dhcp

Cấu hình dhcp.

- Nếu bạn chưa biết cấu hình và cũng ko chắc chắn về cách cấu hình của mình thì hãy copy file mẫu rồi sừa lại.

       cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf
      
     - Nó hỏi có copy đè ko? y enter
     
- Bạn chưa nắm đc cách cấu hình thì các bạn chỉ cần quan tâm đến các phần sau của file

        option domain-name "xxxxx";
        option domain-name-servers 8.8.8.8;
        
        
        subnet 10.0.1.0 netmask 255.255.255.0 {
        range 10.0.1.20 10.0.1.100;
        option routers 10.0.1.1;
        }
        
    - **option domain-name**: đặt tên cho domain, các bạn đặt cái gì cũng được.
     
    - **option domain-name-servers**: đây là DNS server sẽ phân giải tên miền 
       
 ![Selection_021](https://user-images.githubusercontent.com/19284401/55212748-77473880-5223-11e9-883a-2cda5c465caa.png)
 
#### Restart và enable dhcp

       systemctl start dhcpd && systemctl enable dhcpd 


#### Tài liệu tham khảo

1; https://www.server-world.info/en/note?os=CentOS_7&p=dhcp&f=1 
       