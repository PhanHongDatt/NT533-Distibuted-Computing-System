
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
