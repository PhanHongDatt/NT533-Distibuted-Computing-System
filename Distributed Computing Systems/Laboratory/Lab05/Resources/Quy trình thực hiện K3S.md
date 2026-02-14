Tài liệu tham khảo: https://www.digitalocean.com/community/tutorials/how-to-setup-k3s-kubernetes-cluster-on-ubuntu#step-1-mdash-installing-k3s
Phiên bản Ubuntu sử dụng: Ubuntu 24.04
Docker, Puty
Đầu tiên tiến hành cài đặt K3S trên Ubuntu
Command: 
curl -sfL https://get.k3s.io | sh -
![[Pasted image 20251210143435.png]]

Sau khi cài đặt xong tiến hành kiểm tra trạng thái của K3S bằng câu lệnh 
systemctl status k3s
![[Pasted image 20251210143540.png]]

Tiếp theo tiến hành kiểm tra trạng thái của Kubernetes Object
![[Pasted image 20251210143632.png]]

Tuy nhiên nếu chạy câu lệnh trên mà không kèm với quyền sudo thì sẽ xảy ra tình trạng lỗi. Chính vì bậy mà để khắc phục tình trạng này trong quá trình làm lab cần cấp quyền thực thi

Command: sudo chmod 644 /etc/rancher/k3s/k3s.yaml

Tiếp theo, tiến hành cấu hình lại Kubernetes
Thực hiện 2 câu lệnh sau
curl -sfL https://get.k3s.io  | INSTALL_K3S_EXEC="--disable=traefik" sh -
sudo kubectl get all -n kube-system
![[Pasted image 20251210144005.png]]

![[Pasted image 20251210144151.png]]


# Tiến hành cài đặt K3SUP
k3sup:
curl -sLS https://get.k3sup.dev | sh
sudo install k3sup /usr/local/bin/
Sau khi cài đặt xong có thể kiểm tra bằng câu lệnh k3sup --help
![[Pasted image 20251210144830.png]]

# Tiến hành cài đặt Arkade 
curl -sLS https://get.arkade.dev | sudo sh
sau khi thực hiện cài đặt xong tiến hành kiểm tra version bằng câu lệnh 
arkade version

![[Pasted image 20251210145117.png]]

## Tiến hành gen key
![[Pasted image 20251210152927.png]]


---
Tiến hành SSH vô Agent
![[Pasted image 20251210160523.png]]

