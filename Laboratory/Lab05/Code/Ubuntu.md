
''' 
sudo apt update && sudo apt upgrade


// cài đặt kubectl
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" [cite: 65, 66]



![[Pasted image 20251121015912.png]]

chmod +x ./kubectl 
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
![[Pasted image 20251121020009.png]]

curl -sLS https://get.k3sup.dev | sh
sudo install k3sup /usr/local/bin/
![[Pasted image 20251121020124.png]]

Sau đó tiến hành cài arkake , tuy nhiên DNS bị sai nên cần đổi cách cài

curl -SL https://github.com/alexellis/arkade/releases/latest/download/arkade -o arkade

![[Pasted image 20251121020548.png]]

Cấp quyền thực thi và di chuyển vào thư mục

![[Pasted image 20251121020713.png]]
Check lại

![[Pasted image 20251121020737.png]]


sudo apt install -y rpi-imager
rpi-imager


# Re do

Môi trường thực hiện thay thế cho Rasp Pi 4
Máy host: Window 11 chạy WSL
Máy ảo thay thế PI: Ubuntu 22.04
+ Server : 192.168.228.144
+ Agent: 192.168.228.142

## Bước 1 Cài đặt Kubectl
```
sudo apt update
sudo snap install kubectl --clasic 
kubectl version --client 
```
![[Pasted image 20251217091907.png]]

## Bước 2 Cài K3UP
```
curl -sLS https://get.k3sup.dev | sh
sudo install k3sup /usr/local/bin/
k3sup version
```
![[Pasted image 20251217091923.png]]

## Bướ 3 Cài Arkade 
```
curl -sLS https://github.com/alexellis/arkade/releases/latest/download/arkade -o arkade
# Cấp quyền và di chuyển vào thư mục hệ thống
chmod +x arkade
sudo mv arkade /usr/local/bin/
arkade version
```
![[Pasted image 20251217093553.png]]

## Bước 4 Tạo keygen
```
ssh-genkey
```
![[Pasted image 20251217093924.png]]

## Bước 5: Tiến hành copy Key qua các máy ảo
```
#Trên máy Server
ssh-copy-id -o StrictHostKeyChecking=no hongdat@192.168.228.144
```
![[Pasted image 20251217094224.png]]

```
#Trên máy Agent
ssh-copy-id -o StrictHostKeyChecking=no hongdat@192.168.228.142
```

![[Pasted image 20251217094427.png]]


Kiểm tra bằng cách kết nối SSH đến, kết quả thực hiện thành công

![[Pasted image 20251217094535.png]]

## Bước 6: Tiến hành cài K3S Server
```
export SERVER=192.168.228.144

k3sup install \
  --ip $SERVER \
  --user hongdat \
  --ssh-key ~/.ssh/id_ed25519 \
  --k3s-extra-args '--disable traefik'
```
Tuy nhiên để tránh lỗi thì cần thiết lập không yêu cầu mật khẩu sudo cho máy Master, cần tiến hành SSH vào master
```
echo "hongdat ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/hongdat
```

![[Pasted image 20251217100010.png]]


Kiểm tra trạng thái của Node
![[Pasted image 20251217100149.png]]
## Bước 8 Join Agent
```
export SERVER=192.168.228.144
export AGENT=192.168.228.145

k3sup join \
  --ip $AGENT \
  --server-ip $SERVER \
  --user hongdat \
  --ssh-key ~/.ssh/id_ed25519
```

![[Pasted image 20251217120346.png]]


Kiểm tra 
```
kubectl get node
```

![[Pasted image 20251217120541.png]]

## Bước 10 Cài Kubernetes Dashboard
arkade install kubernetes-dashboard
![[Pasted image 20251217131606.png]]

Kiểm tra dashboard có chạy chưa
![[Pasted image 20251217131638.png]]

Tạo user admin và kiểm tra trạng thái

![[Pasted image 20251217131701.png]]

Tạo token để đăng nhập
kubectl -n kubernetes-dashboard create token admin-user
![[Pasted image 20251217131944.png]]


![[Pasted image 20251217132847.png]]