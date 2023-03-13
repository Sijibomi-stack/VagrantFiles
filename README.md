# VagrantFiles
ssh into every node and run this command

## step one
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
 net.bridge.bridge-nf-call-ip6tables = 1
 net.ipv4.ip_forward                 = 1
 net.bridge.bridge-nf-call-iptables = 1
 EOF
 
## step two
 sudo sysctl --system
 
 apt-get update && apt-get install -y \
 apt-transport-https ca-certificates curl software-properties-common gnupg2

## step three
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
 
## step four
 add-apt-repository \
 "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
 $(lsb_release -cs) \
 stable"

## step five 
apt-get update && apt-get install -y \
containerd.io=1.6.14 \
docker-ce=5:20.10.16~3-0~ubuntu-jammy - $(lsb_release -cs) \
docker-ce-cli=5:20.10.16~3-0~ubuntu-jammy - $(lsb_release -cs)

## step six
sudo apt-get update && sudo apt-get install \
ca-certificates \
curl \
gnupg \
lsb-release

## step seven
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

## step eight
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 
## step nine
  sudo apt-get update
 
## step ten
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
 
## step eleven -this is to test that runtime is working
 sudo service docker start
 sudo docker run hello-world
 
 
## step twelve
 cat > /etc/docker/daemon.json <<EOF
 {
   "exec-opts":["native.cgroupdriver=systemd"],
   "log-driver": "json-file",
   "log-opts":{ 
     "max-size": "100m"
	 },
	 "storage-driver": "overlay2"
  }
  EOF
  
## step thirteen
  mkdir -p /etc/systemd/system/docker.service.d
  
  
  
## step fourteen  
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

## step fifteen

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

## step sixteen
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

## step seventeen
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
 
 
## step eighteen  your IP address and Token will be different depending on your own cluster.

sudo apt-get install -y jq
local_ip="$(ip --json a s | jq -r '.[] | if .ifname == "eth1" then .addr_info[] | if .family == "inet" then .local else empty end else empty end')"
cat > /etc/default/kubelet << EOF
KUBELET_EXTRA_ARGS=--node-ip=$local_ip
EOF

## step nineteen to implemented only on master node

IPADDR="x.x.x.x" replace x with the IP address of your master node
NODENAME=$(hostname -s)
POD_CIDR="192.168.0.0/16"

sudo kubeadm init --apiserver-advertise-address=$IPADDR  --apiserver-cert-extra-sans=$IPADDR  --pod-network-cidr=$POD_CIDR --node-name $NODENAME --ignore-preflight-errors Swap

if step 19 is successful, you should have your similar output on your screen. copy them cos you will use kubeadm join to join each work node to the control plane
==============================================================================================
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
 kubeadm join 192.168.56.2:6443 --token 64y9ic.jpnepb3l2puac1b4 --discovery-token-ca-cert-hash sha256:1f3109e60ef2236c53d3745a0f12510d86b0e4edbbc8b937d78aa8e82ec826ba

==================================================================================================
