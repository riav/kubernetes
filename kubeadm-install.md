# KUBE-ADM-INSTALL
## Instalação do CentOS 7
Essa instalação do CentOs7 está baseada na ISO [CentOS-7-x86_64-Minimal-1810.iso](http://mirror.ufscar.br/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso)

**Os procedimentos abaixo devem ser realizados em todos os nodes do cluster.**

### Opções de instalação do CentOS7
* Idioma: English
* Timezone: America/Maceio
* Kdump: Disable
* Security Profile: Disable
* Teclado: Português/No Dead Keys
* Disco: Automático

**O /var/lib precisa está em um volume LVM para suportar ajuste de tamanho, ou ter uma partição acima de 100GB.**

### Desligar o swap
    sed -i 's/.*swap/#&/' /etc/fstab &&\
    swapoff -a
    
### Ajustar Selinux
    sed -i 's/SELINUX=.*/SELINUX=permissive/' /etc/selinux/config &&\
    setenforce 0

### Ajuste no sshd_config
#### Remoção da checagem do reverso ao fazer login

    sed -i 's/GSSAPIAuthentication yes/GSSAPIAuthentication no/' /etc/ssh/sshd_config

### Atualização de pacotes
#### Remoção do s pacotes firewalld postfix NetworkManager, instalação dos install bash-completion iptables-services yum-utils nfs-utils e atualização do sistema.
    yum remove firewalld postfix NetworkManager* -y &&\
    yum install bash-completion iptables-services yum-utils nfs-utils -y &&\
    yum update -y
    
#### Se o host for um VM
    yum install -y open-vm-tools
    
### NTP
    yum install ntp -y &&\
    sed -i "s/^server/#server/" /etc/ntp.conf &&\
    sed -i "/server 0.centos.*/i server a.ntp.br iburst" /etc/ntp.conf &&\
    systemctl enable ntpdate ntpd &&\
    systemctl start ntpdate ntpd
    
### Instalação do Docker
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo &&\
    yum install docker-ce -y &&\
    systemctl enable docker &&\
    systemctl start docker
    
### Configurando o cgroupdrive para systemd
    mkdir /etc/docker
    
    # Set up the Docker daemon
    cat > /etc/docker/daemon.json <<EOF
    {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {"max-size": "100m"},
      "storage-driver": "overlay2",
      "storage-opts": ["overlay2.override_kernel_check=true"]
    }
    EOF
    
    mkdir -p /etc/systemd/system/docker.service.d
    
    # Restart Docker
    systemctl daemon-reload
    systemctl restart docker
    
    cat > /etc/sysconfig/kubelet <<EOF
    KUBELET_EXTRA_ARGS=--cgroup-driver=systemd
    EOF
    
### Instalação do Kubernetes

#### Adiconando repositório do k8s
    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOF

#### Instalar binarios do k8s
    yum install -y kubelet kubectl kubeadm &&\
    systemctl enable kubelet

#### Autocomplete do kubernetes
    source <(kubectl completion bash)

#### Ajuste de parametros no kernel
    echo -e 'net.bridge.bridge-nf-call-ip6tables = 1\nnet.bridge.bridge-nf-call-iptables = 1\nfs.inotify.max_user_watches = 81920' >> /etc/sysctl.d/k8s.conf
    
### Reiniciei o host após toda a execução dos passos:
    reboot

## Instalação do cluster kubernetes

### Iniciando o cluster k8s
    kubeadm init --apiserver-advertise-address $(hostname -i)

#### Copiar o JOIN para inserir os nodes workes

#### Recuperando o join default

    PUB_KEY=$(openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null \
    | openssl dgst -sha256 -hex | sed 's/^.* //') &&\
    TOKEN=$(kubeadm token list | grep default-node-token | awk '{print $1}') &&\
    echo "kubeadm join $(hostname -i):6443 --token $TOKEN --discovery-token-ca-cert-hash sha256:$PUB_KEY"

#### Criando novo join caso o default se perca com o tempo

    kubeadm token create --print-join-command

### Ajustando configuração
    mkdir -p $HOME/.kube &&\
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config &&\
    chown $(id -u):$(id -g) $HOME/.kube/config
    
### Instalando PODs de network
   kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

### Verificar os nodes
    kubectl get pods -n kube-system
