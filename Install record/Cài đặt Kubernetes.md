#Cài đặt Kubernetes

#Các node:
- kube-master: 10.4.18.31
- kube-minion01: 10.4.18.37
- kube-minion02: 10.3.18.39

#Tạo repo
vi /etc/yum.repos.d/virt7-docker-common-release

#Cài đặt trên tất cả các host:
yum -y install --enablerepo=virt7-docker-common-release kubernetes etcd

#Sửa hostfile:
echo "10.4.18.31	kube-master
	  10.4.18.37	kube-minion01
	  10.4.18.39	kube-minion02 
	  10.4.18.40	kube-minion03" >> /etc/hosts

#Sửa file /etc/kubernetes/config

#Tắt firewall trên các node
systemctl disable iptables-services firewalld
systemctl stop iptables-services firewalld

Trên kube-master
#Sửa file /etc/etcd/etcd.conf

#Sửa file /etc/kubernetes/apiserver

#Start service
for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
	systemctl restart $SERVICES
	systemctl enable $SERVICES
	systemctl status $SERVICES
done

Trên kube-minion01 và 02
#Sửa file /etc/kubernetes/kubelet

#Start service:
for SERVICES in kube-proxy kubelet docker; do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES
done

#Kiểm tra các node:
kubectl get nodes