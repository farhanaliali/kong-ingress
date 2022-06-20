############ Kiali Dashboard Expose Via Gateway
Kiali is running on istio-system namespace
serviceName   kiali:20001

Creating Gateway for kiali 

    kubectl create -f kiali_gateway_and_Virtual-service.yml -n istio-system


    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: kiali-gateway
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
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: kiali-vs
    spec:
      hosts:
      - "*"
      gateways:
      - kiali-gateway
      http:
      - route:
        - destination:
            host: kiali 
            port:
              number: 20001


### Accessing Dashboard 

    kubectl get svc istio-ingressgateway -n istio-system 
Get the external IP 
Open in browser 
        http://{external IP}



