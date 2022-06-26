###  Cloud Armor with https LB

##  Creating cloud armor polices with gcloud 

Creating policies

		gcloud compute security-policies create app-user-policy     --description "policy for external users"  
adding rule 

		gcloud compute security-policies rules update 2147483647 \     
			--security-policy app-user-policy \           
			--action "deny-404"

adding rule 

		gcloud compute security-policies rules create 1000 \
			--security-policy app-user-policy \
			--description "allow traffic from 119.160.97.28/32" \
			--src-ip-ranges "119.160.97.28/32" \
			--action "allow"



The following BackendConfig manifest specifies a connection draining timeout of 60 seconds:

    kubectl create -f backendConfig.yaml


    apiVersion: cloud.google.com/v1
    kind: BackendConfig
    metadata:
      name: my-backendconfig
    spec:
      securityPolicy:
        name: app-user-policy
      timeoutSec: 40
      connectionDraining:
        drainingTimeoutSec: 60



############## Create a simple deploymnet with ingress 
Add this annotations in service

annotations:

                 cloud.google.com/backend-config: '{"ports": {"80":"my-backendconfig"}}'
                 
                 cloud.google.com/neg: '{"ingress": true}'

kubectl create -f cloudarmor.yml 

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
          annotations:
                 cloud.google.com/backend-config: '{"ports": {"80":"my-backendconfig"}}'
                 cloud.google.com/neg: '{"ingress": true}' 
        spec:
          type: ClusterIP
          selector:
            app: whereami
          ports:
          - port: 8000 
            targetPort: 8080
        ---
        ##ingress.yaml
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          namespace: default
          name: my-ingress

        spec:
          defaultBackend:
            service:
              name: whereami-svc
              port:
                number: 8000


###### Validating backend service properties

First, describe the my-ingress resource and filter for the annotation that lists the backend services associated with the Ingress. For example:


    kubectl describe ingress my-ingress | grep ingress.kubernetes.io/backends

You should see output similar to the following:

    ingress.kubernetes.io/backends: '{"k8s1-27fde173-default-my-service-80-8d4ca500":"HEALTHY"k8s1-27fde173-kube-system-default-http-backend-80-18dfe76c":"HEALTHY"}


################ testing the service 

  test the service 


