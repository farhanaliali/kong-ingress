######## Traffic Management 

Install istio   ### For installation  go istio_readme.md

#######  Inject the namespace 	

    kubectl label namespace default istio-injection=enabled	

######## default istio example deployment
Go istio installation folder 

	  kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml


####### ######### View the dashboard
Go to istio installation folder

    kubectl apply -f samples/addons
    kubectl rollout status deployment/kiali -n istio-system

Open in other terminal 

    istioctl dashboard kiali

Open and explore the kiali

######################################## Cleanup 

    kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml
    kubectl delete -f samples/bookinfo/networking/bookinfo-gateway.yaml





##################### Creation custom applocation with v1 and v2

    kubectl label namespace default istio-injection=enabled

    cd ../Traffic\ Management
    kubectl create -f deployment.yml
    kubectl create -f Gateway_and_vs.yml
    kubectl get svc istio-ingressgateway -n istio-system

Get the external IP 
open the browser and  http://35.225.174.22  # replace with externalIP
refresh 5 to 10 time  you both version 
Currently traffic is spliting round-robin

You can also explore this  to kaili 

############ Traffice split on persentage 

NOw delete the virtual serivce 
     
     kubectl delete -f Gateway_and_vs.yml

Now we split traffic 90 persent to v1 and 10 persent to v2

     kubectl create -f vs_with_destinationrule.yml


open refersh the browser approx 10 to 20 time 
mostly you land on v1 page 
2 to  3 time you land on v2

you can adjust the traffice persent age in vs_with_destinationrule.yml



############### 


