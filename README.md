# MỤC LỤC

## [1. Khái niệm ảo hóa](#aohoa)

### [1.1 OpenVZ](#openVZ)
### [1.2 XEN](#XEN)
### [1.3 VMWare](#VMWare)
### [1.4 KVM](#KVM)
### [2 Nội dung chính](#noidung)

### <a name="aohoa"></a>1.Khái niệm ảo hóa là gì?
- Ảo hóa với nhiều tên gọi khác nhau như VPS, máy chủ ảo,… là máy chủ được tạo ra bằng cách phân chia máy chủ vật lý thành nhiều máy chủ nhỏ khác nhau. Chúng mang đầy đủ tính chất, chức năng của một máy chủ riêng biệt, được chạy dưới dạng chia sẻ tài nguyên từ máy chủ ban đầu. Tùy thuộc vào nhu cầu sử dụng mà các doanh nghiệp lựa chọn dử dụng máy chủ ảo có các tính chất riêng.

- Một số công nghệ áo hóa phổ biến hiện nay OpenVZ, XEN, VMWare, KVM.

### <a name="openVZ"></a>1.1 OpenVZ
        
   ![technology](https://user-images.githubusercontent.com/19284401/55128967-fd8e4c80-5147-11e9-8cea-e4b19e2be622.jpg)

    
   - OpenVZ: Là một hệ thống cấp công nghệ ảo hóa hoạt động dựa trên nhân Linux. 
    
   - OpenVZ không thực sự ảo hóa, nó sử dụng chung 1 nhân Linux đã được sửa đổi và do đó chỉ có thể chạy duy nhất hệ điều hành Linux,

   - Ứu điểm: Do không có nhân riêng nên nó rất nhanh và hiệu quả.

   **- Nhược điểm:** 
   
   - Nhược điểm của nó khi tất cả các máy chủ phải sử dụng chung 1 nhân duy nhất.
    
   - OpenVZ là việc cấp phát bộ nhớ không được tách biệt, nghĩa là bộ nhớ được cấp phát cho 1 máy chủ VPS này lại có thể bị sử dụng bởi VPS khác trong trường hợp VPS kia yêu cầu
    
   - OpenVZ sử dụng hệ thống file dùng chung, vì thế mối VPS thực chất chỉ là 1 Thư mục được change root.
     
        
### <a name="XEN"></a>1.2 XEN
    
   ![xendiag](https://user-images.githubusercontent.com/19284401/55128904-cddf4480-5147-11e9-88e3-6588bb23075e.png)

   - Là công nghệ ảo hóa thực sự cho phép chạy cùng lúc nhiều máy chủ ảo VPS trên 1 máy chủ vật lý. Công nghệ ảo hóa XEN cho phép mỗi máy chủ ảo chạy nhân riêng của nó, do đó VPS có thể cài được cả Linux hay Windows Operating system, mỗi VPS có hệ thống File System riêng và hoạt động như 1 máy chủ độc lập.
        
   - Tài nguyên cung cấp cho máy chủ VPS XEN cũng độc lập, nghĩa là mỗi máy chủ XEN được cấp 1 lượng RAM, CPU và Disk riêng
        
   - Nhược điểm:
        
       - XEN yêu cầu tài nguyên vật lý đầy đủ cho mỗi VPS, do đó nhà cung cấp dịch vụ cũng phải tăng cường tài nguyên vật lý trên máy chủ thật
            
### <a name="VMWare"></a>1.3 VMWare 

![VMware-ESXi](https://user-images.githubusercontent.com/19284401/55128984-0d0d9580-5148-11e9-9ffa-55fe4f9d8524.jpg)

    
   - Công nghệ ảo hóa VMWare do công ty VMWare phát triển, nó hỗ trợ ảo hóa từ mức phần cứng. Công nghệ này thường áp dụng cho các công ty lớn như ngân hàng, và ít được sử dụng cho các VPS thương mại trên thị trường hiện nay.
        
### <a name="KVM"></a>1.4 KVM (Kernel-based Virtual Machine)
    
![1_zL5mLsbMAWGplkluTUaqtw](https://user-images.githubusercontent.com/19284401/55129371-704bf780-5149-11e9-891a-4f2abb7e6d90.png)
    
   - KVM là công nghệ ảo hóa mới cho phép ảo hóa thực sự trên nền tảng phần cứng. Do đó máy chủ KVM giống như XEN được cung cấp riêng tài nguyên để sử dụng, tránh việc tranh chấp tài nguyên với máy chủ khác trên cùng node. Máy chủ gốc được cài đặt Linux, nhưng KVM hỗ trợ tạo máy chủ ảo có thể chạy cả Linux, Windows. Nó cũng hỗ trợ cả x86 và x86-64 system.
   
   
### <a name="noidung"></a>2 Nội Dung Chính
- Sau khi dạo qua một vòng về các công nghê ảo hóa phổ biến hiện nay. Giờ chúng ta sẽ cùng đi vào vấn đề chính trong phần này đó là tìm hiểu về  **KVM (Kernel-based Virtual Machine)**:

- Tìm hiểu về KVM mình chia làm 3 phần để sẽ giới thiệu cho mọi người (đây là hệ thông thực tế bên mình đang chạy).

    1. Cài đặt, cầu hình và quản lý KVM bằng commdline và giao diện đồ họa.
    
        - Mình sẽ tập trung vào vấn đề cài đặt và cấu hình.

    2. Cài đặt và cấu hình OVS(open virtual switch).

    3. Cấu hình web để quản lý KVM trên giao diện web.

**ok let's go!!!!!**

        
            
        