# Cài đặt Kubernetes sử dụng kubeadm
Sử dụng Centos7
IP các node:
vi /etc/hosts
10.4.18.23	k8s-master
10.4.18.31	k8s-worker01
10.4.18.37	k8s-worker02

scp /etc/hosts root@k8s-woker01:/etc/hosts
scp /etc/hosts root@k8s-woker02:/etc/hosts

Làm trên tất cả các node:

yum update -y

Tạo Repository

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

Tắt firewall

systemctl stop firewalld
systemctl disable firewalld
Tắt Selinux

sudo setenforce 0

Cài đặt docker, kubelet, kubeadm, kubectl

yum install -y kubelet kubeadm kubectl
systemctl enable docker && sudo systemctl start docker
systemctl enable kubelet && sudo systemctl start kubelet

Cấu hình IPtables

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

Trên master node:

kubeadm init --pod-network-cidr 10.244.0.0/16

Trên các worker node:

kubeadm join --token a2dc82.7e936a7ba007f01e 10.0.0.7:6443 --discovery-token-ca-cert-hash sha256:30aca9f9c04f829a13c925224b34c47df0a784e9ba94e132a983658a70ee2914

Trên master node:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Cấu hình Pod Network

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml

Kiểm tra các Pod:

kubectl get pods --all-namespaces

Kiểm tra các node:

kubectl get nodes

Cài đặt Dashboard:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

kubectl proxy

Sửa cấu hình Expose port cho Dashboard

kubectl -n kube-system edit service kubernetes-dashboard

Sửa từ type: ClusterIP thành type: NodePort

Truy cập Dashboard:

Kiểm tra cổng được Expose cho Dashboard

kubectl -n kube-system get service kubernetes-dashboard

https://10.4.18.23:31108

Login bằng token

Cách lấy Dashboard token:

kubectl -n kube-system get secret

kubectl describe secret kubernetes-dashboard-token-lw8t7

Copy token dán vào giao diện truy cập Dashboard bấm Sign in

