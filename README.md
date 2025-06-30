# 🛡️ smallsteps-EJBCA

## 🔐 Smallstep CA + Smallstep Web UI

**Smallstep** là một hệ thống CA hiện đại, hỗ trợ giao diện web, REST API, ACME (cho auto TLS), SSH CA, dễ triển khai và bảo mật.

### ⚙️ Tính năng nổi bật:
- Tạo CA nội bộ với root + intermediate
- Hỗ trợ Web UI quản lý (qua Smallstep Dashboard)
- Hỗ trợ cấp chứng chỉ TLS, SSH
- Triển khai self-host đơn giản trên Ubuntu
- Tích hợp với Kubernetes, Docker, Hashicorp Vault...

---

## 🚀 Hướng dẫn cài đặt Smallstep CA + Web UI

### 🔧 Bước 1: Cài đặt Smallstep CLI và CA
```bash
curl -fsSL https://github.com/smallstep/cli/releases/latest/download/step-cli_linux_amd64.tar.gz | tar xzf - -C /usr/local/bin --strip-components=1
curl -fsSL https://github.com/smallstep/certificates/releases/latest/download/step-ca_linux_amd64.tar.gz | tar xzf - -C /usr/local/bin --strip-components=1
```

### 🧪 Bước 2: Khởi tạo CA
```bash
step ca init
```

Trả lời các câu hỏi:
- Tên CA
- Mật khẩu bảo vệ khóa
- Thư mục lưu CA data (mặc định: `~/.step`)

### ▶️ Bước 3: Khởi chạy CA Server
```bash
step-ca $(step path)/config/ca.json
```

### 🌐 Bước 4: Cài đặt Web UI (Smallstep Dashboard)
```bash
git clone https://github.com/smallstep/dashboard.git
cd dashboard
npm install
npm run build
```

Sau đó bạn có thể:
- Deploy frontend lên một web server (ví dụ: Nginx)
- Backend API vẫn là `step-ca`

---

## 🏛️ Các bước triển khai EJBCA Community Edition (CE)

EJBCA là một hệ thống CA đầy đủ tính năng cho môi trường doanh nghiệp. Phiên bản Community là mã nguồn mở, miễn phí và có thể triển khai nội bộ.

### ☕ Bước 1: Cài Java (OpenJDK 11)
```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version
```

### 🛢️ Bước 2: Cài MariaDB Server
```bash
sudo apt install mariadb-server -y
sudo mysql_secure_installation
```

Tạo user và database:
```sql
CREATE DATABASE ejbca;
CREATE USER 'ejbcauser'@'localhost' IDENTIFIED BY '12345678';
GRANT ALL PRIVILEGES ON ejbca.* TO 'ejbcauser'@'localhost';
FLUSH PRIVILEGES;
```

### 🧰 Bước 3: Tải & cấu hình WildFly
```bash
wget https://github.com/wildfly/wildfly/releases/download/26.1.3.Final/wildfly-26.1.3.Final.tar.gz
tar -xvzf wildfly-*.tar.gz
sudo mv wildfly-* /opt/wildfly
```

Tạo người dùng quản trị:
```bash
cd /opt/wildfly/bin
./add-user.sh  # Chọn Management user
```

### 📦 Bước 4: Tải mã nguồn EJBCA CE
```bash
git clone https://github.com/keyfactor/ejbca-ce.git
cd ejbca-ce
```

### 🛠️ Bước 5: Cấu hình kết nối database

Tạo file `src/ejbca.properties` với nội dung:
```properties
database.name=ejbca
database.username=ejbcauser
database.password=StrongPass123!
database.driver=mysql
database.url=jdbc:mysql://localhost:3306/ejbca
```

### 🏗️ Bước 6: Build EJBCA
```bash
ant deployear
```

Sau khi build xong, file `ejbca.ear` sẽ nằm tại thư mục `dist/`.

### 📤 Bước 7: Deploy lên WildFly
```bash
/opt/wildfly/bin/jboss-cli.sh --connect
deploy dist/ejbca.ear
```

### 🌐 Bước 8: Truy cập giao diện Web
- **Admin GUI**: http://localhost:8080/ejbca/adminweb  
- **Public Web**: http://localhost:8080/ejbca/

---

## 📌 Ghi chú
- Với cả Smallstep và EJBCA, bạn nên backup dữ liệu định kỳ.
- Có thể kết hợp CA nội bộ với các hệ thống như VPN, Intranet, HTTPS nội bộ, xác thực client.
- Khuyến nghị cấu hình HTTPS hoặc reverse proxy bảo vệ truy cập CA server.

---

## 📚 Tham khảo

- 🔗 [Smallstep Docs](https://smallstep.com/docs/)
- 🔗 [Smallstep Dashboard GitHub](https://github.com/smallstep/dashboard)
- 🔗 [EJBCA CE GitHub](https://github.com/keyfactor/ejbca-ce)
- 🔗 [EJBCA Official Docs](https://docs.keyfactor.com/ejbca/)

---

## 🧑‍💻 License
Dự án này sử dụng phần mềm mã nguồn mở từ Smallstep và EJBCA Community Edition. Tuân theo license của từng bên.
