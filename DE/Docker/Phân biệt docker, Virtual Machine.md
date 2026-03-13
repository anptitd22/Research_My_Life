![alt text](images/Docker/Compare_docker_VM.png)

**Docker**
Containers sẽ dùng thẳng OS kernel trên máy dùng
Ví dụ:
- Cài 2 hdh ubuntu, archlinux thì khi `uname -a` đều cho ra kernel ubuntu (hdh máy chủ)

**VM**
Sẽ được tạo, quản lý cấp phát tài nguyên hardware nhờ hypervisor, khi đó cài các hdh trên các VM này sẽ tự tạo OS kernel riêng 
![alt text](images/VM/VM_architecture.png)