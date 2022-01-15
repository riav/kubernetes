# Ingress - NGINX

## Instalação

    # Cloud
    kubectl apply -f https://raw.githubusercontent.com/riav/kubernetes/master/ingress/nginx/controller-v0.49.3-cloud-deploy.yaml
    
    # Baremetal - NodePort
    https://raw.githubusercontent.com/riav/kubernetes/master/ingress/nginx/controller-v0.49.3-baremetal-nodeport-deploy.yaml
    
    # Baremetal - HostNetwork
    https://raw.githubusercontent.com/riav/kubernetes/master/ingress/nginx/controller-v0.49.3-baremetal-host-deploy.yaml
    
## Pós instalação

    POD_NAME=$(kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --field-selector=status.phase=Running -o name) &&\
    kubectl exec $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version
    
    kubectl get all -n ingress-nginx -o wide
