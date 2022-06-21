#####    Canary Deployment

Create a cluster and installed Istio

######## Starting Istio 

	kubectl label namespace default istio-injection=enabled
#################   Deploymnent 

We use a simple test app with frontend-nginx and backend-python pods.
The nginx pods simply redirect every request to the backend pods and act as a proxy. 


    kubectl create -f frontend.yml
    kubectl create -f backend.yml
    kubectl create -f gateway.yml 
Now both both versions of fronted and backend are deployed 

################  Shift traffic to only v1


    kubectl create -f only_v1_route.yml

Now all the Traffic will route to  frontend_v1 and backend_v1

if you curl the gateway ingress ip   

        curl 35.225.234.1      
Output: 

        {
          "label": "v1"
        }



###################### Canary Relase for backend

Now we want to route the 10 persent user to backend_v2

    kubectl apply -f backend_v1_80_v2_20.yml

Now run curl multiple time you get also v2 

    curl 35.225.234.1

Output: 

        {
          "label": "v1"
        }

after 3 to 4 curl requests 
Output:

    {
      "label": "VERSION2"
    }




For testing Team they can  use  Header for request only v2 

    curl -H "canary: canary-tester" 35.225.234.1 
output:

    {
      "label": "VERSION2"
    }

After version 2 testing is ok 


########### Route traffic to v2 only for backend 

    kubectl apply -f only_v2_route.yml

    url 35.225.234.1
Output:
    
    {
    
      "label": "VERSION2"
    
    }

Now V2 is Live 

you can also remove v1 


###################################  Canary Release for fronted 

    kubectl apply -f fronted_v1_80_v2_20.yml

test the fronted then apply a final v2 only 

    kubectl apply -f only_v2_fronted.yml


Now both applications are Upgrade to V2

##############################