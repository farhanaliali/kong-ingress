########## Creating GKE Cluster 
Go to the Google Kubernetes Engine page in the Cloud console.
Click add_box Create.

In the Autopilot section, click Configure.

Enter the Name for your cluster.

Select a region for your cluster.

Choose a public or private cluster.

Click Create.

#########kubeconfig 

Get cluster details 

		gcloud container clusters get-credentials kong-demo --region us-central1 
		
Verfiy cluster details 
		kubectl cluster-info
		

######### deploy Kong Ingress 

##Update User Permissions
Create a rolebinding for kong ingress

            echo -n "
            kind: ClusterRoleBinding
            apiVersion: rbac.authorization.k8s.io/v1
            metadata:
              name: cluster-admin-user
            roleRef:
              apiGroup: rbac.authorization.k8s.io
              kind: ClusterRole
              name: cluster-admin
            subjects:
            - kind: User
              name: silentluck9@gmail.com # usually the Google account

              namespace: kube-system" | kubectl apply -f -


############ Clone kong helm chart


Clone the kong helm chart repo 

		git clone https://github.com/Kong/charts.git 
		 cd charts/kong/
		 
Edit values.yml 

and add Annotation in 

                proxy:
					 annotations:
					         networking.gke.io/load-balancer-type: "Internal"
							 
Save and quit 

Now install the helm chart with values.yml file 

        helm dependency build
        helm repo add kong https://charts.konghq.com
        helm update 
		helm install kong-ingress-demo . -f values.yaml
		
### LB ip  and port 

        HOST=$(kubectl get svc --namespace default kong-ingress-demo-kong-proxy -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

        PORT=$(kubectl get svc --namespace default kong-ingress-demo-kong-proxy -o jsonpath='{.spec.ports[0].port}')

        export PROXY_IP=${HOST}:${PORT}

        curl $PROXY_IP

######### Deployment 
Create a simple deploymnet 
    
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
        konghq.com/strip-path: "true"
        kubernetes.io/ingress.class: kong
    
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
            pathType: ImplementationSpecific
            backend:
              service:
                name: hello-service
                port:
                  number: 8000
          - path: /nginx
            pathType: ImplementationSpecific
            backend:
              service:
                name: nginxservice
                port:
                  number: 80
    
Save and quit 

      kubectl create -f ingress.yml 


################ internal LB 
List  internal LB 
        
        gcloud compute forwarding-rules list --filter="loadBalancingScheme=INTERNAL"  --regions us-central1

Desribe the internal LB 
        
        gcloud compute backend-services describe a7353ff8be54f48009d069011b95f48c --region us-central1

