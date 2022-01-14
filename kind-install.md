# Kind Install

https://kind.sigs.k8s.io/

## Pré Requisito

Ter o docker instalado

    docker info

## Instalação

    # kind
    KIND_URL="https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64" &&\
    command -v wget && CMD_GET_KIND="wget $KIND_URL -O /usr/local/bin/kind" || CMD_GET_KIND="curl -Lk $KIND_URL -o /usr/local/bin/kind" &&\
    sudo CMD_GET_KIND &&\
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
      kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"    
      extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
    - role: worker
    - role: worker
    #- role: worker
    #  image: kindest/node:v1.19.9
    EOF
    
    kind create cluster --config cluster-kind.yaml
    
    # kubectl
    K8S_VERSION=$(docker ps -a | grep control-plane | head -1 | awk '{print $2}' | awk -F: '{print $2}') &&\
    KUBECTL_URL=https://storage.googleapis.com/kubernetes-release/release/$K8S_VERSION/bin/linux/amd64/kubectl &&\
    command -v wget && CMD_GET_KCTL="wget $KUBECTL_URL -O /usr/local/bin/kubectl" || CMD_GET_KCTL="curl -Lk $KUBECTL_URL -o /usr/local/bin/kubectl" &&\
    sudo $CMD_GET_KCTL &&\
    sudo chmod +x /usr/local/bin/kubectl
    
## Remoção

    kind delete cluster --name k8s-cluster
