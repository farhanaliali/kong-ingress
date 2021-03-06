## Install the Vault Helm chart

		helm repo add hashicorp https://helm.releases.hashicorp.com
		helm repo update 

Search for all the Vault Helm chart versions.

		helm search repo vault --versions
	

Default behavior: By default, it launches Vault on a single pod in standalone mode with a file storage backend.
 Enabling high-availability with Raft Integrated Storage requires that you override these defaults.
 
 
		helm install vault hashicorp/vault \
		--set='server.ha.enabled=true' \
		--set='server.ha.raft.enabled=true'
		

############### Initialize and unseal one Vault pod   

		kubectl exec vault-0 -- vault operator init -key-shares=1 -key-threshold=1 -format=json > cluster-keys.json
		
Display the unseal key found in cluster-keys.json.

		cat cluster-keys.json | jq -r ".unseal_keys_b64[]"
		
	
Insecure operation: Do not run an unsealed Vault in production with a single key share and a single key threshold. 
This approach is only used here to simplify the unsealing process for this demonstration.

Create a variable named VAULT_UNSEAL_KEY to capture the Vault unseal key.

		VAULT_UNSEAL_KEY=$(cat cluster-keys.json | jq -r ".unseal_keys_b64[]")		
		
Unseal Vault running on the vault-0 pod.

	kubectl exec vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY
	
	kubectl exec vault-0 -- vault status
	
############## Join the other Vaults to the Vault cluster
 
		cat cluster-keys.json | jq -r ".root_token"
	
Create a variable named CLUSTER_ROOT_TOKEN to capture the Vault unseal key.

		CLUSTER_ROOT_TOKEN=$(cat cluster-keys.json | jq -r ".root_token")
	
Login with the root token on the vault-0 pod.

		kubectl exec vault-0 -- vault login $CLUSTER_ROOT_TOKEN
	
List all the nodes within the Vault cluster for the vault-0 pod.

	kubectl exec vault-0 -- vault operator raft list-peers

Join the Vault server on vault-1 to the Vault cluster.

		kubectl exec vault-1 -- vault operator raft join http://vault-0.vault-internal:8200

Unseal the Vault server on vault-1 with the unseal key.

		kubectl exec vault-1 -- vault operator unseal $VAULT_UNSEAL_KEY

Join the Vault server on vault-2 to the Vault cluster.

		kubectl exec vault-2 -- vault operator raft join http://vault-0.vault-internal:8200

Unseal the Vault server on vault-2 with the unseal key.

		kubectl exec vault-2 -- vault operator unseal $VAULT_UNSEAL_KEY


List all the nodes within the Vault cluster for the vault-0 pod.

		kubectl exec vault-0 -- vault operator raft list-peers


############### Ceating Secert 

		kubectl exec --stdin=true --tty=true vault-0 -- /bin/sh

Enable kv-v2 secrets at the path secret.		

		vault secrets enable -path=secret kv-v2
		
Create a secret at path secret/devwebapp/config with a username and password.
		
		vault kv put secret/devwebapp/config username='test' password='test123'

Verify that the secret is defined at the path secret/data/devwebapp/config.

		vault kv get secret/devwebapp/config
	
Lastly, exit the vault-0 pod.

		exit
		
############## Configure Kubernetes authentication

First, start an interactive shell session on the vault-0 pod.

		kubectl exec --stdin=true --tty=true vault-0 -- /bin/sh

Enable the Kubernetes authentication method.

		vault auth enable kubernetes
		
	  vault write auth/kubernetes/config kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"

Write out the policy named devwebapp that enables the read capability for secrets at path secret/data/devwebapp/config

		vault policy write devwebapp - <<EOF
		path "secret/data/devwebapp/config" {
			capabilities = ["read"]
		}
	EOF
	
Create a Kubernetes authentication role named devweb-app.

		vault write auth/kubernetes/role/devweb-app \
        bound_service_account_names=internal-app \
        bound_service_account_namespaces=default \
        policies=devwebapp \
        ttl=24h
		
		
		exit

#####################Deploy web application

		Create a Kubernetes service account named internal-app
		
		kubectl create sa internal-app
		
		cat > devwebapp.yaml <<EOF
---
apiVersion: v1
kind: Pod
metadata:
  name: devwebapp
  labels:
    app: devwebapp
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "devweb-app"
    vault.hashicorp.com/agent-inject-secret-credentials.txt: "secret/data/devwebapp/config"
spec:
  serviceAccountName: internal-app
  containers:
    - name: devwebapp
      image: jweissig/app:0.0.1
EOF

			kubectl create -f devwebapp.yaml
			kubectl exec --stdin=true --tty=true devwebapp -c devwebapp -- cat /vault/secrets/credentials.txt

#############  Format data: 
A template can be applied to structure this data to meet the needs of the application.

		kubectl exec -it vault-0 -- /bin/sh
		
		vault secrets enable -path=internal kv-v2
		
		vault kv put internal/database/config username="db-readonly-username" password="db-secret-password"
		
		vault kv get internal/database/config


########   Configure Kubernetes authentication

		vault auth enable kubernetes
		vault write auth/kubernetes/config kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"
		vault policy write internal-app - <<EOF
		path "internal/data/database/config" {
		capabilities = ["read"]
		}
	EOF

Create a Kubernetes authentication role named internal-app

		vault write auth/kubernetes/role/internal-app \
		bound_service_account_names=internal-app \
		bound_service_account_namespaces=default \
		policies=internal-app \
		ttl=24h
		

Define a Kubernetes service account

		kubectl create sa internal-app	
		

		kubectl create -f  deployment-orgchart.yaml
			
		apiVersion: apps/v1
		kind: Deployment
		metadata:
		name: orgchart
		labels:
			app: orgchart
		spec:
		selector:
			matchLabels:
			app: orgchart
		replicas: 1
		template:
			metadata:
			annotations:
			labels:
				app: orgchart
			spec:
			serviceAccountName: internal-app
			containers:
				- name: orgchart
				image: jweissig/app:0.0.1


#####  Inject secrets into the pod

		kubectl patch deployment orgchart --patch "$(cat patch-inject-secrets-as-template.yaml)"
		
		
	spec:
		template:
			metadata:
				annotations:
					vault.hashicorp.com/agent-inject: 'true'
					vault.hashicorp.com/agent-inject-status: 'update'
					vault.hashicorp.com/role: 'internal-app'
					vault.hashicorp.com/agent-inject-secret-database-config.txt: 'internal/data/database/config'
					vault.hashicorp.com/agent-inject-template-database-config.txt: |
						{{- with secret "internal/data/database/config" -}}
						postgresql://{{ .Data.data.username }}:{{ .Data.data.password }}@postgres:5432/wizard
						{{- end -}}
						
					kubectl exec \
						$(kubectl get pod -l app=orgchart -o jsonpath="{.items[0].metadata.name}") \
						-c orgchart -- cat /vault/secrets/database-config.txt
						

Output:
	
	postgresql://db-readonly-user:db-secret-password@postgres:5432/wizard
	
	
###############Note 
		
		secret only access form  service account with bind with role 
		aslo namespace will bind with role  so you can only access within namespace 
		
	
