# Kind Install

https://kind.sigs.k8s.io/

## Instalação

    sudo wget https://storage.googleapis.com/kubernetes-release/release/v1.21.1/bin/linux/amd64/kubectl \
    -O /usr/local/bin/kubectl &&\
    sudo chmod +x /usr/local/bin/kubectl

    sudo curl -Lo /usr/local/bin/kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64 &&\
    sudo chmod +x /usr/local/bin/kind &&\
    source <(kind completion bash) &&\
    echo 'source <(kind completion bash)' >> ~/.bashrc
