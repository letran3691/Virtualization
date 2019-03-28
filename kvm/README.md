### MỤC LỤC
-------------

##### [1. Giới thiệu KVM](#1)
  - [1.1 Tính năng nổi bật](#1.1)
  - [1.2 Cấu trức của KVM](#1.2)
#### [2. Cài đặt KVM Centos7](#2)



------------

### <a name="1"></a>1. Giới thiệu KVM

KVM ra đời phiên bản đầu tiên vào năm 2007 bởi công ty Qumranet tại Isarel, KVM được tích hợp sẵn vào nhân của hệ điều hành Linux bắt đầu từ phiên bản 2.6.20. Năm 2008, RedHat đã mua lại Qumranet và bắt đầu phát triển, phổ biến KVM Hypervisor.

KVM (Kernel-based virtual machine) là giải pháp ảo hóa cho hệ thống linux trên nền tảng phần cứng x86 có các module mở rộng hỗ trợ ảo hóa (Intel VT-x hoặc AMD-V).

Về bản chất, KVM không thực sự là một hypervisor có chức năng giải lập phần cứng để chạy các máy ảo. Chính xác KVM chỉ là một module của kernel linux hỗ trợ cơ chế mapping các chỉ dẫn trên CPU ảo (của guest VM) sang chỉ dẫn trên CPU vật lý (của máy chủ chứa VM). Hoặc có thể hình dung KVM giống như một driver cho hypervisor để sử dụng được tính năng ảo hóa của các vi xử lý như Intel VT-x hay AMD-V, mục tiêu là tăng hiệu suất cho guest VM.

KVM hiện nay được thiết kế để giao tiếp với các hạt nhân thông qua một kernel module có thể nạp được. Hỗ trợ một loạt các hệ thống điều hành máy guest như: Linux, BSD, Solaris, Windows, Haiku, ReactOS và hệ điều hành nghiên cứu AROS. Sử dụng kết hợp với QEMU, KVM có thể chạy Mac OS X.

Trong kiến trúc của KVM, Virtual machine được thực hiện như là quy trình xử lý thông thường của Linux, được lập lịch hoạt động như các scheduler tiểu chuẩn của Linux. Trên thực tế, mỗi CPU ảo hoạt động như một tiến trình xử lý của Linux. Điều này cho phép KVM được hưởng lợi từ tất cả các tính năng của nhân Linux.

### <a name="1.1"></a>1.1  Vài tính năng nổi bật
    
   **Storage**

   - KVM sử dụng khả năng lưu trữ hỗ trợ bởi Linux để lưu trữ các images máy ảo, bao gồm các local dish với chuẩn IDE, SCSI và SATA, Network Attached Storage (NAS) bao gồm NFS và SAMBA/CIFS, hoặc SAN được hỗ trợ iSCSI và Fibre Chanel.
      
   - KVM cũng hỗ trợ các image của máy ảo trên hệ thống chia sẻ tập tin như GFS2 cho phép các image máy ảo được chia sẻ giữa các host hoặc các logical volumn
      
   - Định dạng image tự nhiên của KVM là QCOW2 – hỗ trợ việc snapshot cho phép snapshot nhiều mức, nén và mã hóa dữ liệu.
  
  **Live migration**
  
   - KVM hỗ trợ live migration cung cấp khả năng di chuyển ác máy ảo đang chạy giữa các host vật lý mà không làm gián đoạn dịch vụ. Khả năng live migration là trong suốt với người dùng, các máy ảo vẫn duy trì trạng thái bật, kết nối mạng vẫn đảm bảo và các ứng dụng của người dùng vẫn tiếp tục duy trì trong khi máy ảo được đưa sang một host vật lý mới.
   
=> Khi QEMU/KVM kết hợp nhau thì tạo thành type-1 hypervisor.

 - QEMU cần KVM để boost performance và ngược lại KVM cần QEMU (modified version) để cung cấp giải pháp full virtualization hoàn chỉnh.

 - Do KVM kết hợp QEMU nên các máy ảo và mạng ảo có file cấu hình xml sẽ được lưu lại tại thư mục /etc/libvirt/qemu/


   
   
### <a name="1.2"></a>1.2  Cấu trúc của KVM

   ![687474703a2f2f696d6775722e636f6d2f777341356846372e6a7067](https://user-images.githubusercontent.com/19284401/55133059-f7ec3300-5156-11e9-9efd-0f7abee2dbf5.png)
   
   - **User-facing tools**: Là các công cụ quản lý máy ảo hỗ trợ KVM Các công cụ có giao diện đồ họa (như virt-manager) hoặc giao diện dòng lệnh như (virsh) và virt-tool (Các công cụ này được quản lý bởi thư viện libvirt).
   
   - **Management layer**: Lớp này là thư viện libvirt cung cấp API để các công cụ quản lý máy ảo hoặc các hypervisor tương tác với KVM thực hiện các thao tác quản lý tài nguyên ảo hóa, bởi vì KVM chỉ là một module của nhân hỗ trợ cơ chế mapping các chỉ dẫn của CPU ảo để thực hiện trên CPU thật, nên tự thân KVM không hề có khả năng giả lập và quản lý tài nguyên ảo hóa. Mà phải dùng nhờ các công nghệ hypervisor khác, thường là QEMU.

   - **Virtual machine**: Chính là các máy ảo người dùng tạo ra. Thông thường, nếu không sử dụng các công cụ như virsh hay virt-manager, KVM sẽ sử được sử dụng phối hợp với một hypervisor khác điển hình là QEMU.

   - **Kernel support**: Chính là KVM, cung cấp một module làm hạt nhân cho hạ tầng ảo hóa (kvm.ko) và một module kernel đặc biệt chỉ hỗ trợ các vi xử lý VT-x hoặc AMD-V (kvm-intel.ko hoặc kvm-amd.ko) để nâng cao hiệu suất ảo hóa.
   
    
### <a name="2"> </a>2 Cài đặt KVM   

-  Trước khi bắt đầu cài đặt các bạn cần tắt firewall
        
        systemctl stop firewall
        systemctl disable firewall

- Cài đặt eple-release
        
        yum -y install epel-release
        
- Cài đặt các packet cần thiết.

        yum -y install qemu-kvm libvirt virt-install libguestfs libvirt-client virt-manager virt-top virt-viewer virt-who
      
    - Ở đây mình không cài linux-bridge mà thay, vào đó mình sẽ cài OVS(open visual switch). mình sẽ nhắc đến ở phần sau
    
            yum install "@X Window System" xorg-x11-xauth xorg-x11-fonts-* xorg-x11-utils -y
            
    - Muốn sử dụng **virt-manager** cần có packet này để sử dụng khi quản lý bằng giao diện độ họa.
    
- Start và enable libvirtd

       systemctl start libvirtd
       systemctl enable libvirtd
       
Như vậy là đã cấu hình xong KVM.

- Giờ ta sẽ chuyển sang cấu hình OVS(Open visual switch)
 
     <a href="https://github.com/letran3691/AoHoa/tree/master/OVS#1" rel="nofollow">  Cài đặt OVS</a>
     
     
- Giờ ta sẽ đi cấu hình interface trên host

    - Tạo ra 1 interface ảo tên ifcfg-br0
    
            vi /etc/sysconfig/network-scripts/ifcfg-br0
            
       - Nội dung
            
             DEVICE=br0
             ONBOOT=yes
             TYPE=Bridge
             BOOTPROTO=dhcp
             DELAY=0
              
          - **DEVICE**: tên interface
          - **ONBOOT**: active interface
          - **TYPE**: Kiểu kết nối
          - **BOOTPROTO**: ip sẽ  cấp như thế nào       
    
    - Sửa file interface vật lý
            
            vi /etc/sysconfig/network-scripts/ifcfg-em1
            
        - Nội dung
                
                DEVICE=em1
                ONBOOT=yes
                NETBOOT=yes
                UUID=
                BOOTPROTO=dhcp
                HWADDR=""
                TYPE=Ethernetem1
                BRIDGE=br0

- Như vậy ta đã  cấu hình xong  bridge từ **br0** **em1**
    - Kiểm lại bằng lệnh 
    
          ovs-vsctl show
          
![Selection_027](https://user-images.githubusercontent.com/19284401/55142441-3857ab00-516f-11e9-8af5-40d51e524694.png)

- Như các bạn thấy ở trong hình thì interface **em1** bây giờ đã thành port của **br0**

- Các bạn hãy hiểu đơn giản thế này, **br0** giờ nó đã là 1 con swich. và interface **em1** đã được cắm vào swithc **br0**

- Giờ ra sẽ tạo network cho KVM

     - Như đã nói ở phần tính năng thì các file cấu hình VM và network sẽ có dạng XML 
     
        - Nội dung của 1 file network sẽ như sau:
        
            - Đây là cấu trúc của 1 file **NETWORK ROUTE** 
        
                    <network>
                      <name>route</name>
                      <forward dev='ens33' mode='route'>
                        <interface dev='ens33'/>
                      </forward>
                      <bridge name='virbr1' stp='on' delay='0'/>
                      <ip address='192.168.1.225' netmask='255.255.255.224'>
                        <dhcp>
                          <range start='192.168.1.240' end='192.168.1.254'/>
                        </dhcp>
                      </ip>
                    </network>
                    
                - network **route** sẽ được route ra ngoài bằng interface **ens33**
                - **forward dev**: là tên card mạng vật lý của host 
                - **mode**: là cở chế của card mạng
                - **bridge name** : là tên network 
                - **ip address** đâu là ip của interface
                - **range start**: dải ip dhcp cấp
               
            - Đây là cấu trúc của 1 file **NETWORK BRIDGE**
            
                    <network>
                      <name>ovs-br0</name>
                      <forward mode='bridge'/>
                      <bridge name='br0'/>
                      <virtualport type='openvswitch'/>
                      <portgroup name='no-vlan' default='yes'>
                      </portgroup>
                      <portgroup name='vlan-100'>
                         <vlan>
                           <tag id='100'/>
                       </vlan>
                      </portgroup>
                        <portgroup name='vlan-200'>
                           <vlan>
                             <tag id='200'/>
                          </vlan>
                      </portgroup>
                    </network>
                  
                 - file này mình có tạo thêm các vlan
                 - toàn bộ network **ovs-br0** sẽ đc bridge ra ngoài bằng interface **br0**
                  
            - Đây là cấu trúc của file **NETWORK NAT**
            
                    <network>
                      <name>ovs-br0</name>
                      <forward mode="nat" dev="br1"/>
                      <virtualport type='openvswitch'/>
                      <ip address='192.168.122.1' netmask='255.255.255.0'>
                        <dhcp>
                          <range start='192.168.122.2' end='192.168.122.254'/>
                        </dhcp>
                      </ip>
                    </network>
                    
                - Network **ovs-br0** sẽ được NAT ra ngoài qua interface **br1**
                
- Tùy mục địch sử dụng mà chọn kiểu network cho hợp lý.

- Ở đâu mình sẽ tạo ra 2 network:

    - file thứ nhất tên br0.xml
    
        - nội dung file thứ nhất
        
                <network>
                  <name>br0</name>
                  <forward mode='bridge'/>
                  <bridge name='br0'/>
                  <virtualport type='openvswitch'/>
                </network>
            
        - Tạo ra 1 network tên br0 với chế độ bridge
        - file này lát mình sẽ cấu hình để nó bridge với mạng thật thông qua interface vật lý
        
    - file thứ 2 tên localbr.xml
        
                <network>
                  <name>localbr</name>
                  <forward mode='bridge'/>
                  <bridge name='localbr'/>
                  <virtualport type='openvswitch'/>
                  <portgroup name='no-vlan' default='yes'>
                  </portgroup>
                  <portgroup name='vlan-100'>
                     <vlan>
                       <tag id='100'/>
                   </vlan>
                  </portgroup>
                    <portgroup name='vlan-200'>
                       <vlan>
                         <tag id='200'/>
                      </vlan>
                  </portgroup>
                </network>

        - Tạo ra 1 network tên localbr với chế độ bridge
        - network này sẽ dùng để quản lý và cấp pháp trong LAN
        - file này như các bạn đã thấy thì sẽ ko có DHCP. thay vào đó mình sẽ dựng riêng 1 con DHCP SERVER

- Sau khi có 2 file trên giờ chúng ta sẽ đi định nghĩ 2 network này để KVM sử dụng nó.

![Selection_025](https://user-images.githubusercontent.com/19284401/55140051-c466d400-5169-11e9-8db4-8414f6acca29.png)

- Định nghĩ file br0.xml

        virsh net-define /root/br0.xml
        virsh net-start br0
        virsh net-autostart br0
        
- Định nghĩ file localbr.xml

        virsh net-define /root/localbr.xml
        virsh net-start localbr
        virsh net-autostart localbr
        
     - **virsh net-define**: virsh sẻ đọc cấu hình của file đội tạo ra network theo file đó
     - **virsh net-start**: active network
     - **virsh net-autostart**: trường hợp nào nó host bị tắt, khi host bật lên thì nó sẽ khơi động cùng KVM
     
- Sau khi tạo network xong ban kiểm tra lại bằng lệnh 
        
        virsh net-list
        
![Selection_026](https://user-images.githubusercontent.com/19284401/55140579-03495980-516b-11e9-9da1-d2da1cf382df.png)

ok vây là chúng ta đã cấu hình xong <a href="https://github.com/letran3691/AoHoa/tree/master/kvm" rel="nofollow">KVM</a> và <a href="https://github.com/letran3691/AoHoa/tree/master/OVS" rel="nofollow"> OVS</a>

- Giờ chúng ta sẽ cài đặt mới 1 VM.
    - Trước hết sẽ tạo 1 thư mục chưa images thay vì thư mục mặc định của KVM
    
            mkdir -p /var/kvm/images
            
    - cài đặt VM bằng command
    
            virt-install \
            --name centos7 \
            --ram 2048 \
            --disk path=/var/kvm/images/centos7.img,size=30 \
            --vcpus 2 \
            --os-type linux \
            --os-variant rhel7 \
            --network bridge=br0 \
            --graphics none \
            --console pty,target_type=serial \
            --location 'đường dẫn đền file iso' \
            --extra-args 'console=ttyS0,115200n8 serial'

      - name : Tên VM
      - ram: RAM cho VM
      - disk path: đường dẫn đền nơi lưu trữ VM    
      - size : dung lương ổ cứng của VM
      - vcpus: số core cpu
      - os-type: kiểu os (linux hoặc windows)
      
       Tham khảo thêm <a href="http://www.owl.homeip.net/manuals/services/kvm/ibm_kvm/virt-install-options.htm" rel="nofollow">  tại đây </a>
       
- Nếu như các bạn cấu hình đúng các option và run cả lệnh trên thì sẽ được như hình dưới.     
       
![Selection_028](https://user-images.githubusercontent.com/19284401/55144744-114fa800-5174-11e9-9dff-9559e3a1f64a.png)
![Selection_029](https://user-images.githubusercontent.com/19284401/55144754-144a9880-5174-11e9-979c-1fe000373427.png)

- Một vài lệnh cơ bản để thao tác với KVM
        
    - list ra cách VM đang chạy
    
            virsh list 
    
   ![Selection_030](https://user-images.githubusercontent.com/19284401/55144958-72777b80-5174-11e9-94ef-1e947415ba1d.png)
   
   - list ra tất cả các VM có trong host cả chạy lẫn ko chạy
    
            virsh list --all
   ![Selection_031](https://user-images.githubusercontent.com/19284401/55144961-75726c00-5174-11e9-8d59-cbcbbcbb7135.png)
   
   - run và shutdown VM
   
            virsh shutdown tênVM
            virsh start tênVM
            
   - Tạo snapshot cho VM
   
            virsh snapshot-create-as --name tênsnap tênVM
            
   - list snapshot của VM
            
            virsh snapshot-list  tênVM   
   
   - revert lại VM
            
            virsh snapshot-revert tênVM tênsnap
            
   - **Chú ý: Tạo snapshot và revert thì phải shutdown VM**
   
            
   - Truy cập vào VM từ console của host
            
            virsh console tênVM
            
   ![Selection_032](https://user-images.githubusercontent.com/19284401/55145492-5de7b300-5175-11e9-96bf-528f4923a296.png)
   
   - Để thoát của VM trong console bạn dùng tổ hợp phím **Ctr+]**
   
   

            
- Nếu các bạn cảm thấy khó năng khi cài bằng lệnh thì chúng ta sẽ quản lý bằng đồ họa.  

- Quản lý KVM bằng virt-manager (giao diện đồ họa).

       
       

 
       
Tài liệu tham khảm:

 1: https://github.com/hocchudong/thuctap012017/blob/master/TamNT/Virtualization/docs/KVM/1.Tim_hieu_KVM.md#1.2



