# Sample Backend Golang dengan Echo dan Load Balancer Nginx

## 1. Deskripsi
Proyek ini adalah backend API yang dibuat menggunakan **Golang** dan **Echo Framework**. API ini memiliki dua endpoint utama:

- **GET /user** → Mengembalikan data user dalam format JSON.
- **POST /user/create** → Menerima JSON payload dari Postman atau klien lain dan mengembalikan respons.

Untuk meningkatkan ketersediaan dan performa, kita menggunakan **Nginx sebagai Load Balancer** untuk mendistribusikan request ke beberapa instance backend.

---

## 2. Instalasi

### **A. Instalasi di Ubuntu**

#### **1. Install Nginx**
```sh
sudo apt install nginx -y
```

#### **2. Install Echo Framework**
```sh
go get -u github.com/labstack/echo/v4
```

---

### **B. Instalasi di macOS**

#### **1. Install Nginx**
```sh
brew install nginx
```

#### **2. Install Echo Framework**
```sh
go get -u github.com/labstack/echo/v4
```

---

## 3. Menjalankan Backend Golang

### **A. Menjalankan Backend Secara Lokal**

Buka terminal dan jalankan perintah berikut untuk menjalankan server:

```sh
PORT=8081 go run main.go
```

Untuk menjalankan beberapa instance backend:

```sh
PORT=8082 go run main.go &
PORT=8083 go run main.go &
```

Server akan berjalan di `http://localhost:8081/user` dan `http://localhost:8081/user/create`.

---

## 4. Konfigurasi Load Balancer dengan Nginx

Edit file konfigurasi Nginx:

```sh
sudo nano /usr/local/etc/nginx/nginx.conf  # macOS
sudo nano /etc/nginx/nginx.conf   # Ubuntu
```

Tambahkan konfigurasi berikut di dalam blok `http`:

```nginx
http {
    upstream golang_backend {
        server 127.0.0.1:8081;
        server 127.0.0.1:8082;
        server 127.0.0.1:8083;
    }

    server {
        listen 8080;
        server_name localhost;

        location / {
            proxy_pass http://golang_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### **Restart Nginx**
```sh
sudo nginx -s reload  # Jika sudah berjalan
sudo systemctl restart nginx  # Untuk Ubuntu
brew services restart nginx  # Untuk macOS
```

---

## 5. Testing API dengan Curl

### **A. GET Request**
```sh
curl -X GET http://localhost:8080/user
```
- **Response:**
  ```json
  {
      "id": 1,
      "name": "John Doe",
      "email": "johndoe@example.com",
      "age": 30
  }
  ```

### **B. POST Request**
```sh
curl -X POST http://localhost:8080/user/create \
     -H "Content-Type: application/json" \
     -d '{"id": 2, "name": "Alice", "email": "alice@example.com", "age": 25}'
```
- **Response:**
  ```json
  {
      "message": "User created successfully",
      "user": {
          "id": 2,
          "name": "Alice",
          "email": "alice@example.com",
          "age": 25
      }
  }
  ```

---

## 6. Troubleshooting

### **A. Cek Status Backend**
```sh
lsof -i :8081
lsof -i :8082
lsof -i :8083
```
Jika tidak ada proses yang berjalan, jalankan ulang backend.

### **B. Cek Log Error Nginx**
```sh
cat /usr/local/var/log/nginx/error.log  # macOS
cat /var/log/nginx/error.log  # Ubuntu
```

### **C. Restart Nginx dan Backend**
```sh
sudo nginx -s stop && sudo nginx
brew services restart nginx  # Untuk macOS
```

