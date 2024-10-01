# Các lệnh cơ bản của DOCKER

**1. Image**

- `docker images`: Liệt kê các images đang có trên máy.
- `docker pull <image-name>`: Tải image về máy.
- `docker rmi <image-id>`: Xóa image khỏi máy.
- `docker pull <image-name>:<tag>`: Tải image về máy với tag cụ thể.
- `docker push <image-name>:<tag>`: Đẩy image lên registry.
- `docker prune`: Xóa tất cả các image không sử dụng.

- Sort Hand: `docker pull` = `docker image pull`, `docker push`, `docker rmi` = `docker image rm`, `docker prune` = `docker image prune`.

**2. Container**

- `docker ps`: Liệt kê các container đang chạy.
- `docker ps -a`: Liệt kê tất cả các container.
- `docker run <image-name>`: Chạy một container từ image.
- `docker stop <container-id>`: Dừng container.
- `docker container prune`: Xóa tất cả các container đang dừng.
- `docker exec <container-id> <command>`: Chạy một lệnh trong container đang chạy.

## Chu kỳ sống của một Container (Lifecycle)

- `docker run`: Tạo container từ image. (trạng thái **UP**)
- `docker stop`: Dừng container. (trạng thái **EXITED**)
- `docker start`: Khởi động container đã dừng. (trạng thái **UP**)
- `docker rm | docker prune`: Xóa container. (trạng thái **DEAD**)

**1. Container Lifecycle in Depth: PID1**

- PID1: Process ID 1
- PID1 là một quy trình đặc biệt (tiến trình chủ đạo) trong container, nó chịu trách nhiệm quản lý các tiến trình con trong container.
- Khi dùng `docker stop` thì PID1 dừng chuyển sang trạng thái **SIGTERM**, container sẽ dừng ngay lập tức.
- Khi PID1 bị dừng, container sẽ chuyển sang trạng thái **EXITED (0)** .
- Khi dùng `docker kill` hoặc quá 10s thì PID1 dừng chuyển sang trạng thái **SIGKILL**, container sẽ dừng ngay lập tức **EXITED (137)**.

- `docker run alpine`: Container sẽ chạy lệnh `sh` và PID1 sẽ chạy `sh`.
- `docker run -it alpine`: Container sẽ chạy lệnh `sh` và PID1 sẽ chạy `sh` (khi cần sự tương tác với người dùng).
- `Ctrl + D`: Dừng container và PID1 chuyển sang trạng thái **EXITED (0)**.

- **Note**: Khi tạo một container, PID1 sẽ chạy một lệnh cụ thể, nếu lệnh đó kết thúc thì container sẽ dừng.

**2. Chế độ Container**

- Attach Mode: `docker run -it alpine sh`. `Ctrl + D` để dừng container.
- Detach Mode: `docker run -d alpine sleep 1000`. `Ctrl + P + Q`để thoát khỏi container (container vẫn chạy ngầm). Nếu muốn vào container thì dùng `docker attach <container-id>`.
- `docker run -d <image>`: Chạy container ở chế độ detach.

## Execute Command in Container

- `docker exec <container-id> <command>`: Chạy một lệnh trong container đang chạy.
- `docker exec -it <container-id> sh | bash`: Tạo một terminal trong container.

**1. Port Mapping**

- `docker run -p <host-port>:<container-port> <image>`: Map port từ host vào container.

**2. Logs trace**

- `docker logs -f <container-id>`: Xem logs của container đang chạy.

**3. Volume - bind mount**

- Volume: Là một thư mục (ảo) hoặc file nằm ngoài container, được mount vào container.
- Thao tác này gọi là bind mount.

- Trình tự:
  - `docker volume create <volume-name>`: Tạo volume.
  - `docker run -v <volume-name>:<container-path> <image>`: Mount volume vào container.

# Cách tạo Dockerfile

**1. Cách tạo image**

- `docker build -t <image-name> .`: Tạo image từ Dockerfile.

**2. Các lệnh dockerfile**

- FROM <images>: Image cơ sở.
- RUN <command>: Chạy lệnh.
- WORKDIR <directory>: Thư mục làm việc.
- COPY <src> <dest>: Copy file từ host vào container.
- ADD <src/URL> <dest>: Tương tự như copy nhưng hỗ trợ URL và có thể tự giải nén.
- EXPOSE <port>: Mở port.
- CMD <command argument1 argument2...>: Lệnh mặc định khi chạy container (Hook command) (Shell form).
- CMD [command argument1 argument2...]: Lệnh mặc định khi chạy container (Hook command) (Exec form - Recommend).

**3. Quy trình chạy Dockerfile**

- Image được tạo ra từ nhiều layer. Mỗi lệnh trong Dockerfile tạo ra một layer.
- Khi building image, Docker sẽ duyệt từng lệnh trong Dockerfile và tạo ra một layer tương ứng.
- Qua mỗi câu lệnh, Docker sẽ tạo một container tạm thời từ layer image trước đó và sao lưu sang layer mới. Xong câu lệnh tiếp theo, container tạm thời sẽ bị xóa và tạo container mới từ layer mới cho đến khi hoàn thành Dockerfile.

**4. Push image lên registry**

- `docker login`: Đăng nhập vào registry.
- `docker push <image-name>:<tag>`: Đẩy image lên registry.
