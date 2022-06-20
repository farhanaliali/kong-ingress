############### isntallation of istio
Download the istio CLI

		curl -L https://istio.io/downloadIstio | sh -

To configure the istioctl client tool for your workstation,
add the /home/farhanali/Notepad/kubernetes/yamls/service_mesh/istio/istio-1.14.1/bin directory to your environment path variable with:
	 export PATH="$PATH:/home/farhanali/Notepad/kubernetes/yamls/service_mesh/istio/istio-1.14.1/bin"
	 
################ PreCheck

	 istioctl x precheck

######### isntallation
  
     cd istio-1.14.1 
	 export PATH=$PWD/bin:$PATH
	  istioctl install --set profile=demo -y
	
######## Starting Istio 

		kubectl label namespace default istio-injection=enabled	
		
######## default istio example  deployment 

		 kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
			kubectl get svc
			kubectl get pod
			
wait until all the pods in running 
###### verfiy the deployment

		kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"

##########Open the application to outside traffic

			kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
			istioctl analyze
			kubectl get svc istio-ingressgateway -n istio-system
			http://(laodbalacerip)/productpage    ###  34.123.129.206/productpage 
			
your service is deployed with service mesh 
########### Cleanup basic example
	
	kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml 
	kubectl delete -f samples/bookinfo/networking/bookinfo-gateway.yaml
	
######## Deploying basic whereiam deployment 

			kubectl create -f istio-basic-deployment.yml  
			
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: whereami
            spec:
              selector:
                matchLabels:
                  app: whereami
              template:
                metadata:
                  labels:
                    app: whereami
                spec:
                  containers:
                  - name: whereami
                    image: gcr.io/google-samples/whereami:v1.2.6
                    ports:
                    - containerPort: 8080
            ---
            apiVersion: v1
            kind: Service
            metadata:
              name: whereami-svc
            spec:
              type: ClusterIP
              selector:
                app: whereami
              ports:
              - port: 8000 
                targetPort: 8080

############# Gateway 

        kubectl create -f Gateway.yml
        kubectl get svc istio-ingressgateway -n istio-system

    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: whereami-gateway
    spec:
      selector:
        istio: ingressgateway # use Istio default gateway implementation
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "*"

    
  
######  Virtual Service 

    kubectl create -f Virtual-service.yml

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: frontend-ingress
    spec:
      hosts:
      - "*"
      gateways:
      - whereami-gateway
      http:
      - route:
        - destination:
            host: whereami-svc
            port:
              number: 8000

############# Verfiy the Gateway and Virtual service 

    kubectl get svc istio-ingressgateway -n istio-system

get the External IP 

      curl 34.123.129.206   ## Replace with you External IP 
or open in browser 

       http://34.123.129.206/       ## Replace with you External IP

    

######### View the dashboard   
 Go to istio installation folder 
 
        kubectl apply -f samples/addons
        kubectl rollout status deployment/kiali -n istio-system

        istioctl dashboard kiali

####### Next steps



These tasks are a great place for beginners to further evaluate Istio’s features 

    Request routing
    Fault injection
    Traffic shifting
    Querying metrics
    Visualizing metrics
    Accessing external services
    Visualizing your mesh
    
    Before you customize Istio for production use, see these resources:
    
    Deployment models
    Deployment best practices
    Pod requirements
    General installation instructions


