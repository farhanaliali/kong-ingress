######### Nginx ingress 

		helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
		helm repo update
		helm show values ingress-nginx/ingress-nginx > nginx_values.yml
	
add the GKE internal LB annotation

vim nginx_values.yml
  
      service:
                 annotations: 
							networking.gke.io/load-balancer-type: "Internal"	  
							
save and quit 

       helm install nginx-ingress ingress-nginx/ingress-nginx -f nginx_values.yml
		helm list

################### deployment and ingress rule for 2 microservices 
git clone 

kubectl create -f nginx-deploy-internal-apps.yml

    apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: hello-deployment
	spec:
	  selector:
	    matchLabels:
	      app: hello
	  template:
	    metadata:
	      labels:
	        app: hello
	    spec:
	      containers:
	      - name: hello
	        image: gcr.io/google-samples/hello-app:2.0
	        ports:
	        - containerPort: 8080

	---
	apiVersion: v1
	kind: Service
	metadata:
	  name: hello-service
	spec:
	  type: ClusterIP
	  selector:
	    app: hello
	  ports:
	  - port: 8000 
	    targetPort: 8080

	---
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: nginx
	spec:
	  selector:
	    matchLabels:
	      app: nginx
	  template:
	    metadata:
	      labels:
	        app: nginx
	    spec:
	      containers:
	      - name: nginx
	        image: nginx
	        ports:
	        - containerPort: 80

	---
	apiVersion: v1
	kind: Service
	metadata:
	  name: nginxservice
	spec:
	  type: ClusterIP
	  selector:
	    app: nginx
	  ports:
	  - port: 80 
	    targetPort: 80

	---
	apiVersion: networking.k8s.io/v1
	kind: Ingress
	metadata:
	  name: awesome-ingress
	  annotations:
	       nginx.ingress.kubernetes.io/rewrite-target: /
	       kubernetes.io/ingress.class: "nginx"

	spec:
	  defaultBackend:
	    service:
	      name: hello-service
	      port:
	        number: 8000
	  rules:
	  - http:
	      paths:      
	      - path: /hello
	        pathType: Prefix
	        backend:
	          service:
	            name: hello-service
	            port:
	              number: 8000
	      - path: /nginx
	        pathType: Prefix
	        backend:
	          service:
	            name: nginx
	            port:
	              number: 80



########### verify the deploymets and ingress rules

	kubectl get pods
	kubectl get ing
	kubectl describe  ing awesome-ingress 

verify the endpoints also 

your nginx ingress and loadbalacer is created now you can access it form internal VPC

########### Deploying third service 

	kubectl create -f nginx-deploy-internal-thirdapp.yml


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
		---
		apiVersion: networking.k8s.io/v1
		kind: Ingress
		metadata:
		  name: third-svs
		  annotations:
		       nginx.ingress.kubernetes.io/rewrite-target: /
		       kubernetes.io/ingress.class: "nginx"
		spec:
		  rules:
		  - http:
		      paths:
		      - path: /whereami
		        pathType: Prefix
		        backend:
		          service:
		            name: whereami-svc
		            port:
		              number: 8000


############### testing the ingress rules 

	kubectl run curl -it  --image=curlimages/curl -- sh

	curl {yourLoadbalacer url}


############ clean up the resourece 

	
	kubectl delete -f nginx-deploy-internal-thirdapp.yml
	kubectl delete -f nginx-deploy-internal-apps.yml
	kubectl delete pod curl 
	helm uninstall nginx-ingress

delete the cluster 

