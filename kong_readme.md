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

		helm install kong-ingress-demo . -f values.yaml
		
### LB ip  and port 

        HOST=$(kubectl get svc --namespace default kong-ingress-demo-kong-proxy -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

        PORT=$(kubectl get svc --namespace default kong-ingress-demo-kong-proxy -o jsonpath='{.spec.ports[0].port}')

        export PROXY_IP=${HOST}:${PORT}

        curl $PROXY_IP

######### Deployment 
Create a simple deploymnet 

        kubectl create  deploy demo-app --image=nginx:1.19-alpine 
        kubectl expose deploy demo-app --type=ClusterIP --port=80

############ Creata a ingress rule 
vim ingress.yml

            apiVersion: networking.k8s.io/v1
            kind: Ingress
            metadata:
              name: demo-app
              annotations:
                konghq.com/strip-path: "true"
                kubernetes.io/ingress.class: kong
            spec:
              rules:
              - http:
                  paths:
                  - path: /
                    pathType: Prefix
                    backend:
                      service:
                        name: demo-app
                        port:
                          number: 80

Save and quit 

      kubectl create -f ingress.yml 


################ internal LB 
List  internal LB 
        
        gcloud compute forwarding-rules list --filter="loadBalancingScheme=INTERNAL"  --regions us-central1

Desribe the internal LB 
        
        gcloud compute backend-services describe a7353ff8be54f48009d069011b95f48c --region us-central1

