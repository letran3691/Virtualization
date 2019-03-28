### MỤC LỤC

#### [Giới thiệu](#1) 

#### [Cài đặt ](#2) 



#### <a name="1"></a>1 Giới Thiệu

OpenvSwitch (OVS) là một dự án về chuyển mạch ảo đa lớp (multilayer). Mục đích chính của OpenvSwitch là cung cấp lớp chuyển mạch cho môi trường ảo hóa phần cứng, trong khi hỗ trợ nhiều giao thức và tiêu chuẩn được sử dụng trong hệ thống chuyển mạch thông thường. OpenvSwitch hỗ trợ nhiều công nghệ ảo hóa dựa trên nền tảng Linux như Xen/XenServer, KVM, và VirtualBox.


![0a60d30a-5c3e-4cc8-aa9a-a1f91e2546c0](https://user-images.githubusercontent.com/19284401/55133603-d68c4680-5158-11e9-89fe-c5083349b603.png)

- OpenvSwitch hỗ trợ các tính năng sau: - VLAN tagging & 802.1q trunking - Standard Spanning Tree Protocol (802.1D) - LACP - Port Mirroring (SPAN/RSPAN) - Tunneling Protocols - QoS

- Các thành phần chính của OpenvSwitch:
     
    - **ovs-vswitchd**: thực hiện chuyển đổi các luồng chuyển mạch.
    
    - **ovsdb-server**: là một lightweight database server, cho phép ovs-vswitchd thực hiện các truy vấn đến cấu hình.

    - **ovs-dpctl**: công cụ để cấu hình các switch kernel module.

    - **ovs-vsctl**: tiện ích để truy vấn và cập nhật cấu hình ovs-vswitchd.

    - **ovs-appctl**: tiện ích gửi command để chạy OpenvSwitch.
    
    
#### <a name="2"><a/>2 Cài đặt

- cài đặt repo hộ trợ OVS

        yum -y localinstall https://repos.fedorapeople.org/repos/openstack/openstack-ocata/rdo-release-ocata-3.noarch.rpm
        
- Cài đặt ovs

        yum install openvswitch -y
        
- Enable và start ovs
    
        systemctl enable openvswitch.service
        systemctl start openvswitch.service

- Disable và stop NetworkManager
    
        systemctl disable NetworkManager.service 
        systemctl stop NetworkManager.service
        
- Enabel và restart network service
    
        systemctl enable network.service
        systemctl restart network.service
        
- Kiểm tra trạng thái của network service vs NetworkManager

       
        systemctl is-enabled network.service  
        systemctl is-enabled NetworkManager.service      
 
 ![Selection_024](https://user-images.githubusercontent.com/19284401/55134210-b8bfe100-515a-11e9-952f-fbaebdbadf72.png)
 
 
 - Một vài lệnh cơ bản với OVS
    
    - show switch ảo
            
             ovs-vsctl show   
     
![Selection_033](https://user-images.githubusercontent.com/19284401/55146997-2af2ee80-5178-11e9-949c-31f3639fa293.png)
    
   - Mỗi 1 port ở đây sẽ tương ứng với 1 interface của VM đang cắm vào
   
   - show ra các thông tin liên quan đên interface cắm trên swithc br0
    
            ovs-ofctl show br0
            
![Selection_034](https://user-images.githubusercontent.com/19284401/55147336-c8e6b900-5178-11e9-9fd2-ac879fbf02fc.png)

Tìm hiểu thêm <a href="http://www.openvswitch.org/support/dist-docs/"> tại đây. <a/>




Tài liệu tham khảo:

1:  https://viblo.asia/p/tong-quan-ve-sdn-va-openvswitch-m68Z0N865kG