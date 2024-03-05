# Vargant là gì? 
- Vagrant là một công cụ phần mềm mã nguồn mở
- Xây dựng và phát triển tương tác với các nền tảng ảo hóa như VirtualBox, KVM, Hyper-V, Docker, VMware, AWS và được viết bằng ngôn ngữ lập trình Ruby nhưng có hỗ trợ phát triển bằng một số loại ngôn ngữ khác

# Lợi ích gì khi biết cách dùng Vargant?
- Vagrant giúp cho việc triển khai các server trên các nền tảng ảo hóa được đơn giản, tối ưu, tập trung và nhanh chóng hơn. 
- Tại sao vậy? Khi bạn sử dụng triển khai thủ công các server sẽ rất tốn thời gian và chỉ triển khai được từng server
- Nếu như dự án lớn bạn cần phải tạo đến hàng chục server, hơn thế việc triển khai thủ công có thể bạn sẽ quên mất và lại phải xem lại cách làm hoặc sai bước nào đó là lo lắng, cũng như việc sử dụng code để tạo hạ tầng (Infrastructure as Code) sẽ giúp chúng ta có những file config tập chung sau này muốn tạo hạ tầng tương tự chỉ việc cầm file config qua và chạy sẽ tiết kiệm, nhẹ nhàng hơn rất nhiều.

Ref: https://developer.hashicorp.com/vagrant/intro

# Cài đặt Vargant
- Trên Ubuntu
~~~bash
sudo apt update && sudo apt upgrade
~~~
~~~bash
sudo apt-get -y install vagrant
~~~
~~~bash
vagrant --version
~~~
- Trên Window/MacOS: 
https://www.vagrantup.com/downloads.html

# Cài dặt VirtualBox 
~~~bash
sudo apt update && sudo apt upgrade
~~~
~~~bash
sudo apt install virtualbox
~~~
~~~bash
vboxmanage --version
~~~

# Sử dụng Vagrant tạo máy ảo cơ bản
- Tạo thư mục lưu trữ những config
~~~bash
mkdir /home/vagrant-config && chmod -R 777 /home/vagrant-config/ cd /home/vagrant-config
~~~
- Tạo Vagrantfile (file cấu hình tạo máy ảo)
~~~bash
vi Vagrantfile
~~~
```shell
Vagrant.configure(2) do |config|                 # khai báo máy ảo
  
  config.vm.box = 'ubuntu/trusty64'              # Sử dụng Box ubuntu/trusty64 tạo máy ảo được lấy từ vagrant cloud
  config.vm.provider "virtualbox" do |vb|        # Máy ảo dùng virtualbox và cấu hình tài nguyên
     vb.name = "server-1"                        # tên máy ảo được tạo ra
     vb.cpus = 2                                 # cấp 2 core CPU
     vb.memory = "4096"                          # cấp 4GB bộ nhớ
  end                                            # kết thúc cấu hình tài nguyên 

end                                              #  kết thúc cấu hình tạo máy ảo
```
- `vagrant up` tạo máy ảo
- `vagrant ssh` truy cập vào máy ảo bằng ssh, password mặc định là `vagrant`

# Các lệnh Vagrant cơ bản
- `vagrant init` tạo Vagrantfile mới.

- `vagrant up` thực hiện tạo máy ảo.

- `vagrant ssh` truy cập vào máy ảo.

- `vagrant halt` dừng máy ảo (shutdown).

- `vagrant reload` chạy lại máy ảo có cập nhật lại cấu hình từ Vagrantfile (khi thay đổi cấu hình máy).

- `vagrant destroy` xóa máy ảo.

# Dùng Vagrant tạo nhiều máy ảo
- Tạo thư mục lưu trữ những config
~~~bash
mkdir /home/Vagrant-config && chmod -R 777 /home/Vagrant-config && cd /home/Vagrant-config
~~~
- Tạo Vagrantfile (file cấu hình tạo máy ảo)
~~~bash
vi Vagrantfile
~~~

## Solution 1: Tạo nhiều máy ảo cơ bản
~~~
Vagrant.configure("2") do |config|
  config.vm.define "web1" do |web1|
    web1.vm.provider "virtualbox" do |vb_web1|
      vb_web1.memory = 8192
      vb_web1.cpus = 2
    end

    web1.vm.box = "ubuntu/trusty64"
    web1.vm.hostname = "webserver-1-projectname"
    web1.vm.network "private_network", ip: "10.32.4.130"

  end

  config.vm.define "web2" do |web2|
    web2.vm.provider "virtualbox" do |vb_web2|
      vb_web2.memory = 8192
      vb_web2.cpus = 2
    end

    web2.vm.box = "ubuntu/trusty64"
    web2.vm.hostname = "webserver-2-projectname"
    web2.vm.network "private_network", ip: "10.32.4.131"

  end

  config.vm.define "postgresql" do |postgresql|
    postgresql.vm.provider "virtualbox" do |vb_postgresql|
      vb_postgresql.memory = 8192
      vb_postgresql.cpus = 2
    end

    postgresql.vm.box = "ubuntu/trusty64"
    postgresql.vm.hostname = "database-projectname"
    postgresql.vm.network "private_network", ip: "10.32.4.132"

  end

end
~~~
~~~bash
vagrant up
~~~

## Solution 2: Tạo nhiều máy ảo bằng vòng lặp
~~~bash
Vagrant.configure("2") do |config|
  vm_name=['webserver-1-projectname','webserver-2-projectname','database-projectname']
  vm_name.each do |i|
      config.vm.define "#{i}" do |node|
      node.vm.box = "ubuntu/trusty64"
    end
  end
end
~~~
~~~bash
vagrant up
~~~

# Tạo máy ảo kèm những ứng dụng cần thiết
~~~bash
Vagrant.configure("2") do |web|
  web.vm.box = "ubuntu/trusty64"
  web.vm.network "private_network", ip: "10.32.4.130"
  web.vm.hostname = "webserver.xtl"
  web.vm.synced_folder '.', '/var/www/public/'
  web.vm.provider "virtualbox" do |vb_web|
     vb_web.name = "webserver.xtl"
     vb_web.cpus = 2
     vb_web.memory = "8192"
  end
  #web.vm.provision "shell", path: "install-docker.sh" #install docker từ file .sh (nếu cần thiết)
  web.vm.provision "shell", inline: <<-SHELL
    # cài đặt Nginx
    apt update
    apt -y install nginx
    systemctl start nginx
    ufw allow 'Nginx Full'

    # Đặt mật khẩu tài khoản root là 'elroydev' và cho phép đăng nhập ssh
    echo "elroydev" | passwd --stdin root
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
    systemctl reload sshd

    # Tạo file cấu hình nginx để chạy web
    echo '
      server {
	listen 80 default_server;
	listen [::]:80 default_server;
	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
    }' > /etc/nginx/conf.d/webserver.conf
    systemctl reload nginx
  SHELL
              
end
~~~

- Tại thư mục hiện tại (tức thư mục ‘.’) tạo và viết vào file `vi index.html`
```
Welcome webserver
```
- `vagrant reload` áp dụng lại cấu hình



