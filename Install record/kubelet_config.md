# The address for the info server to serve on

KUBELET_ADDRESS="--address=0.0.0.0"

# The port for the info server to serve on

KUBELET_PORT="--port=10250"

# You may leave this blank to use the actual hostname

KUBELET_HOSTNAME="--hostname-override=kube-minion" #01, 02

# Location of the api-server

KUBELET_API_SERVER="--api-servers=http://kube-master:8080"

# Add your own!

KUBELET_ARGS=""