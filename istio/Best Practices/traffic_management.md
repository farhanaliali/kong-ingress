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



####### Split large virtual services and destination rules into multiple resources==
Consider the case of a VirtualService bound to an ingress gateway exposing an application host which uses path-based delegation to several implementation services, something like this:

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

The downside of this kind of configuration is that other configuration (e.g., route rules) for any of the underlying microservices, will need to also be included in this single configuration file, instead of in separate resources associated with, and potentially owned by, the individual service teams. See Route rules have no effect on ingress gateway requests for details.

To avoid this problem, it may be preferable to break up the configuration of myapp.com into several VirtualService fragments, one per backend service. For example:

    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: myapp-service1
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
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: myapp-service2
    spec:
      hosts:
      - myapp.com
      gateways:
      - myapp-gateway
      http:
      - match:
        - uri:
            prefix: /service2
        route:
        - destination:
            host: service2.default.svc.cluster.local
    ---
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: myapp-...



######### Avoid 503 errors while reconfiguring service routes

When setting route rules to direct traffic to specific versions (subsets) of a service, care must be taken to ensure that the subsets are available before they are used in the routes. Otherwise, calls to the service may return 503 errors during a reconfiguration period.

Creating both the VirtualServices and DestinationRules that define the corresponding subsets using a single kubectl call (e.g., kubectl apply -f myVirtualServiceAndDestinationRule.yaml is not sufficient because the resources propagate (from the configuration server, i.e., Kubernetes API server) to the Pilot instances in an eventually consistent manner. If the VirtualService using the subsets arrives before the DestinationRule where the subsets are defined, the Envoy configuration generated by Pilot would refer to non-existent upstream pools. This results in HTTP 503 errors until all configuration objects are available to Pilot.

To make sure services will have zero down-time when configuring routes with subsets, follow a “make-before-break” process as described below:

When adding new subsets:

Update DestinationRules to add a new subset first, before updating any VirtualServices that use it. Apply the rule using kubectl or any platform-specific tooling.

Wait a few seconds for the DestinationRule configuration to propagate to the Envoy sidecars

Update the VirtualService to refer to the newly added subsets.

When removing subsets:

Update VirtualServices to remove any references to a subset, before removing the subset from a DestinationRule.

Wait a few seconds for the VirtualService configuration to propagate to the Envoy sidecars.

Update the DestinationRule to remove the unused subsets.
