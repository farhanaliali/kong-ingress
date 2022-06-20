Service association: 
         A pod must belong to at least one Kubernetes service even if the pod does NOT expose any port. 
         If a pod belongs to multiple Kubernetes services, the services cannot use the same port number for different protocols, for instance HTTP and TCP.

Application UIDs: 
      Ensure your pods do not run applications as a user with the user ID (UID) value of 1337 because 1337 is reserved for the sidecar proxy.

NET_ADMIN and NET_RAW capabilities: 
      If pod security policies are enforced in your cluster and unless you use the Istio CNI Plugin, your pods must have the NET_ADMIN and NET_RAW capabilities allowed. The initialization containers of the Envoy proxies require these capabilities.
      
To check if the NET_ADMIN and NET_RAW capabilities are allowed for your pods, you need to check if their service account can use a pod security policy that allows the NET_ADMIN and NET_RAW capabilities. If you haven’t specified a service account in your pods’ deployment, the pods run using the default service account in their deployment’s namespace.
