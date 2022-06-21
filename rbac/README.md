############# Iam
Its only for GKE user

Go to Iam and create a new user with below permissions


        Kubernetes Engine Cluster Viewer


#############   RBAC

You can define RBAC rules in ClusterRole and Role objects, and then assign those rules with ClusterRoleBinding and RoleBinding objects as follows:

    ClusterRole: a cluster-level grouping of resources and allowed operations that you can assign to a user or a group using a RoleBinding or a ClusterRoleBinding.
    
    Role: a namespaced grouping of resources and allowed operations that you can assign to a user or a group of users using a RoleBinding.
    
    ClusterRoleBinding: assign a ClusterRole to a user or a group for all namespaces in the cluster.
    
    RoleBinding: assign a Role or a ClusterRole to a user or a group within a specific namespace.



############ Role 

Role will give the access within namespace 

    kubectl create -f role.yml

        apiVersion: rbac.authorization.k8s.io/v1
        kind: Role
        metadata:
          name: pod-reader
        rules:
        - apiGroups: [""] # "" indicates the core API group
          resources: ["pods"]
          verbs: ["get", "watch", "list"]

In this role user only list  watch and  get the pods ( user will not create the any pod )

########## Role binding 
Rolebinding will bind the role with user

kubectl create -f rolebinding.yml

        kind: RoleBinding
        apiVersion: rbac.authorization.k8s.io/v1
        metadata:
          name: pod-reader-binding
          
        subjects:
        # Google Cloud user account
        - kind: User
          name: janedoe@example.com
        roleRef:
          kind: Role
          name: pod-reader
          apiGroup: rbac.authorization.k8s.io

######### Test Role and rolebinding

Switch account to other user with you give access 

set the kubeconfig 

         gcloud container clusters get-credentials rbac   --region us-central1-c

         kubectl get pods  # its work fine because you have get access of pod 

         kubectl delete pod  (podname) #

Output 

Error from server (Forbidden): pods "whereami-7cd46875b5-xdmpf" is forbidden: User "pathergarhtest@gmail.com" cannot delete resource "pods" in API group "" in the namespace "default": requires one of ["container.pods.delete"] permission(s).


Now update the user permison to so they can delete the pod 

     kubectl apply -f role_delete.yml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: pod-reader
    rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods"]
      verbs: ["get", "watch", "list", "delete"]

########  

NOw use have the delete permission So run agian delete command

 Output:
    
    pod "whereami-7cd46875b5-xdmpf" deleted

###############   Creating LoadBalancer 

    kubectl expose deployment whereami --type=LoadBalancer --port=8000

OutPut: 

    kubectl expose deployment whereami --type=LoadBalancer --port=8000

        Error from server (Forbidden): deployments.apps "whereami" is forbidden: User "pathergarhtest@gmail.com" cannot get resource "deployments" in API group "apps" in the namespace "default": requires one of ["container.deployments.get"] permission(s).

We can not create a Service beucase user do not have a permission to Create a servce 

#### Allow user to Create a service 

    kubectl create -f role_svc.yml

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: pod-reader
    rules:
    - apiGroups: ["","apps"] # "" indicates the core API group
      resources: ["pods","services","deployments"]
      verbs: ["get", "watch", "list","create","update"]



     kubectl expose deployment whereami --type=LoadBalancer --port=8000

OutPut: 

    service/whereami exposed
