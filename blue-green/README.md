Create GKE and install istio 

######## Starting Istio

	kubectl label namespace default istio-injection=enabled	


######### Blue Green 
Deploy applications 

    kubectl create -f deploymnet.yml

Create a Gateway for this 

    kubectl create -f  gateway.yml



############### Version 1 

    kubectl create -f only_v1_route.yml

Only v1 is live and v2 is ideal state 

    kubectl get svc -n istio-system istio-ingressgateway

now test 

    curl 35.226.138.123
output:

    {
      "label": "v1"
    }


############ Testing V2 

we need to create a new route for v2 

    kubectl create -f gateway_v2.yml
    kubectl create -f only_v2_route.yml

    curl  -H "Host: v2.istio.com" 35.226.138.123
Output:

    {
      "label": "VERSION2"
    }


after testing the v2 we will shift the Traffic to V2

##################  Version 2

after testing the v2 we will shift the Traffic to V2

    kubectl create -f shift_to_v2.yml
    
    curl 35.226.138.123                         
Output:

    {
      "label": "VERSION2"
    }

#########################################