########### Create cluster with anthos service mesh 

Fisrt you need to enable anthos api 

Go to apis and servie  =====> enable apis and service  ===> search for anthos and enable it

############### Cluster creation 

Go to create cluster ---> standred cluster
add the nodes which are requireds

Go to features and enable the anthos mesh 

Now create the cluster 

Wait until both cluster and anthos are created         # you can check in notification section


######## Deploy ingress Gateway for cluster traffic  

    kubectl create ns gateway
    kubectl label namespace gateway istio-injection=enabled
    kubectl create -f istio-ingressgateway -n gateway    
    kubectl get svc -n gateway




#############3 Deploy a simple deployment 

        kubectl label namespace default istio-injection=enabled
        kubectl create -f deployment.yml
        kubectl create -f Gateway.yml
        kubectl create -f Virtual-service.yml

If deployment will not injected then restart the deployment 
        
        kubectl rollout restart deploy 

########### testing 

    kubectl get svc -n  gateway 

Get the external IP and open in browser 

        http://104.197.234.34

