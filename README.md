# smallsteps-EJBCA
Smallstep CA + Smallstep Web UI
Smallstep là một hệ thống CA hiện đại, hỗ trợ giao diện web, REST API, ACME (cho auto TLS), SSH CA, dễ triển khai và bảo mật.

⚙️ Tính năng nổi bật:
Tạo CA nội bộ với root + intermediate

Hỗ trợ Web UI quản lý (qua Smallstep Dashboard)

Hỗ trợ chứng chỉ TLS, SSH

Có thể triển khai self-host đơn giản trên Ubuntu

Tích hợp được với Kubernetes, Docker, Hashicorp Vault...
#Hướng dẫn cài đặt Smallstep CA + Web UI
- Bước 1: Cài đặt Smallstep CA
curl -fsSL https://github.com/smallstep/cli/releases/latest/download/step-cli_linux_amd64.tar.gz | tar xzf - -C /usr/local/bin --strip-components=1
curl -fsSL https://github.com/smallstep/certificates/releases/latest/download/step-ca_linux_amd64.tar.gz | tar xzf - -C /usr/local/bin --strip-components=1
- Bước 2: Khởi tạo CA
step ca init
Trả lời các câu hỏi như:
Tên CA
Password bảo vệ khóa
Đường dẫn lưu CA data
Sau khi hoàn tất sẽ có thư mục chứa dữ liệu CA (mặc định: ~/.step).
- Bước 3: Khởi chạy CA server
step-ca $(step path)/config/ca.json
- Bước 4: Cài đặt Smallstep Web UI (Dashboard)
Cài đặt dashboard (dạng SPA React)
git clone https://github.com/smallstep/dashboard.git
cd dashboard
npm install
npm run build

Sau đó, bạn có thể:
Deploy frontend lên một webserver như Nginx
Backend API vẫn là Smallstep CA (step-ca)
_____________________________________
#Các bước triển khai EJBCA CE

- Bước 1: Cài Java (OpenJDK 11)
sudo apt update
sudo apt install openjdk-11-jdk -y

Kiểm tra:
java -version

- Bước 2: Cài MariaDB Server
sudo apt install mariadb-server -y
sudo mysql_secure_installation

Tạo user và DB:
CREATE DATABASE ejbca;
CREATE USER 'ejbcauser'@'localhost' IDENTIFIED BY '12345678';
GRANT ALL PRIVILEGES ON ejbca.* TO 'ejbcauser'@'localhost';
FLUSH PRIVILEGES;

- Bước 3: Tải & cấu hình WildFly
wget https://github.com/wildfly/wildfly/releases/download/26.1.3.Final/wildfly-26.1.3.Final.tar.gz
tar -xvzf wildfly-*.tar.gz
mv wildfly-* /opt/wildfly

Tạo người quản trị:
cd /opt/wildfly/bin
./add-user.sh  # chọn Management user

- Bước 4: Tải mã nguồn EJBCA CE
git clone https://github.com/keyfactor/ejbca-ce.git
cd ejbca-ce

- Bước 5: Cấu hình EJBCA (chỉnh sửa các file conf nếu cần)
Tạo file cấu hình database trong:
src/ejbca.properties

Thêm các dòng cấu hình:
database.name=ejbca
database.username=ejbcauser
database.password=StrongPass123!
database.driver=mysql
database.url=jdbc:mysql://localhost:3306/ejbca

- Bước 6: Build EJBCA
ant deployear
Sau khi build xong, file ejbca.ear sẽ được tạo ra tại dist/.

- Bước 7: Deploy lên WildFly
/opt/wildfly/bin/jboss-cli.sh --connect
deploy dist/ejbca.ear

- Bước 8: Truy cập giao diện Web
Admin GUI: http://localhost:8080/ejbca/adminweb
Public Web: http://localhost:8080/ejbca/
