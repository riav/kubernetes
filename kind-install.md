# Kind Install

https://kind.sigs.k8s.io/

## Pré Requisito

Ter o docker instalado

    docker info

## Instalação

    # kind
    sudo curl -Lo /usr/local/bin/kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64 &&\
    sudo chmod +x /usr/local/bin/kind &&\
    source <(kind completion bash) &&\
    echo 'source <(kind completion bash)' >> ~/.bashrc
    
    # Instalação cluster
    
    cat << EOF > cluster-kind.yaml
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    name: k8s-cluster
    #networking:
    #  podSubnet: "10.244.0.0/16"
    #  serviceSubnet: "10.96.0.0/12"
    nodes:
    - role: control-plane
    - role: worker
    - role: worker
    #- role: worker
    #  image: kindest/node:v1.19.9
    EOF
    
    kind create cluster --config cluster-kind.yaml
    
    # kubectl
    K8S_VERSION=$(kind get clusters) &&\
    sudo wget https://storage.googleapis.com/kubernetes-release/release/$K8S_VERSION/bin/linux/amd64/kubectl \
    -O /usr/local/bin/kubectl &&\
    sudo chmod +x /usr/local/bin/kubectl
