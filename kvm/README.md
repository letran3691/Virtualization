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

 Giờ ta sẽ chuyển sang cấu hình OVS(Open visual switch)
 
<a href="https://github.com/letran3691/AoHoa/tree/master/OVS#1" rel="nofollow">  Cài đặt OVS</a>

 
 
       
Tài liệu tham khảm:

 1: https://github.com/hocchudong/thuctap012017/blob/master/TamNT/Virtualization/docs/KVM/1.Tim_hieu_KVM.md#1.2



