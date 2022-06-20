#######    Set default routes for services

Although the default Istio behavior conveniently sends traffic from any source to all versions of a destination service without any rules being set,
creating a VirtualService with a default route for every service, right from the start, is generally considered a best practice in Istio.

Even if you initially have only one version of a service, as soon as you decide to deploy a second version, 
you need to have a routing rule in place before the new version is started, to prevent it from immediately receiving traffic in an uncontrolled way.

For example, consider the following destination rule as the one and only configuration defined for the reviews service,
that is, there are no route rules in a corresponding VirtualService definition

  apiVersion: networking.istio.io/v1alpha3
  kind: DestinationRule
  metadata:
    name: reviews
  spec:
    host: reviews
    subsets:
    - name: v1
      labels:
        version: v1
      trafficPolicy:
        connectionPool:
          tcp:
            maxConnections: 100
            
   
   You can fix the above example in one of two ways. You can either move the traffic policy up a level in the DestinationRule to make it apply to any version:

apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
  subsets:
  - name: v1
    labels:
      version: v1

Or, better yet, define a proper route rule for the service in the VirtualService definition. For example, add a simple route rule for “reviews:v1”:

apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1



==Split large virtual services and destination rules into multiple resources==
Consider the case of a VirtualService bound to an ingress gateway exposing an application host which uses path-based delegation to several implementation services, something like this:
<mark>
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp.com
  gateways:
  - myapp-gateway
  http:
  - match:
    - uri:
        prefix: /service1
    route:
    - destination:
        host: service1.default.svc.cluster.local
  - match:
    - uri:
        prefix: /service2
    route:
    - destination:
        host: service2.default.svc.cluster.local
  - match:
    ...
</mark>
