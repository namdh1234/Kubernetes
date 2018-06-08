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

kubeadm join --token a2dc82.7e936a7ba007f01e 10.4.18.23:6443 --discovery-token-ca-cert-hash sha256:30aca9f9c04f829a13c925224b34c47df0a784e9ba94e132a983658a70ee2914

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

Tạo token:

Tạo CreateServiceAccount.yaml với nội dung như sau:

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system

kubectl create -f CreateServiceAccount.yaml

Tạo CreateClusterRoleBinding.yaml với nội dung như sau:

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
[root@k8s-master k8s-user]# cat CreateClusterRoleBinding.yaml 
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system

kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

Output như sau:

Name:         admin-user-token-p49vc
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=admin-user
              kubernetes.io/service-account.uid=85820950-6ae3-11e8-a006-005056b1763e

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXA0OXZjIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI4NTgyMDk1MC02YWUzLTExZTgtYTAwNi0wMDUwNTZiMTc2M2UiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.NRtm1OREcs3zS3Is_ueSmgt39DQdEtdlA6ptFUywJXlo1l6rKkoKcHn0WG4wh-gzPJHxjyuwVxAQGewA3ZGMzOivi76GhvG5cu21dirzQNF3uXEIa-Vba9Ci65JTddP1mJQZb6p485vddpW1WPqRFRaQkicJx7_5-UalyakykADnlaQSaTAQi3D7xn7fhR_rK6iEsiFFteUnnfpzFIeMmAko-yG0fUEv_RBYQqDrKmxrG2idLtrv75OXAero7cA_QzBk2K0qilIFsw17Xbitd57Aj8MhrJjZOqaELRXNQGV5TZadtXmQSgqZvuxv82jxedXkExBowcqbSPx-GgVOKw

Copy token dán vào giao diện truy cập Dashboard bấm Sign in

