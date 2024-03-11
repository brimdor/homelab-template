## Retrieving Secrets and Password Resets

### Gitea
`kubectl get secret gitea.admin -n global-secrets -o jsonpath='{.data.password}' | base64 --decode && echo`

### ArgoCD
`export KUBECONFIG=./metal/kubeconfig.yaml && kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

### Grafana
`kubectl get secret dex.grafana -n global-secrets -o jsonpath='{.data.client_secret}' | base64 --decode && echo`

### Reset Grafana Admin Password
This will reset the password to 'admin'. Reset it in Grafana after login.  
`export KUBECONFIG=./metal/kubeconfig.yaml && kubectl -n grafana exec -it $(kubectl get pods -n grafana --no-headers | grep grafana | awk '{print $1}') -- /bin/sh -c 'grafana cli admin reset-admin-password admin'`

### KanIDM
User - admin  
`export KUBECONFIG=./metal/kubeconfig.yaml && kubectl exec -it -n kanidm statefulset/kanidm -- kanidmd recover-account admin`  

User - idm_admin  
`export KUBECONFIG=./metal/kubeconfig.yaml && kubectl exec -it -n kanidm statefulset/kanidm -- kanidmd recover-account idm_admin`

## Fixes and Processes

### Stop Auto-Heal and Set Manual-Sync
`argocd login argocd.eaglepass.io --grpc-web --no-verify`  
`argocd proj windows add default -k deny --schedule "* * * * *" --duration 24h --namespaces * --manual-sync`

### Start Auto-Heal and Set Auto-Sync
`argocd proj windows delete default`

### Scale Deployments
`kubectl scale deployment (name of deployment)-deployment --replicas=(count) -n (namespace)`

### Gitea Fix
`argocd app set gitea --repo https://github.com/brimdor/homelab`  
`argocd app sync gitea`

## Backup
1. `locate backup -> Operation -> Restore Latest Backup`
2. `Use Previous Name -> Click OK`
3. `Volume -> search by PVC Name -> input name -> Click Go`
4. `k3s kubectl scale deployment (name of deployment)-deployment --replicas=0 -n (namespace)`
5. `Hover over action menu to the right of the restored Backup -> Create PV/PVC -> Click OK`
6. `k3s kubectl scale deployment (name of deployment)-deployment --replicas=1 -n (namespace)`

## Restore Backup
1. `Login to ArgoCD`
2. `Set Window to stop Sync`
3. `Do a more recent backup of whatever you need to expand. You will have to delete the current PVC in order to establish an expanded one.`
4. `Spin down to 0 replicas`
5. `Adjust Helm Chart or Yaml deployment to claim the new amount.`
6. `After backup, delete current pvc`
7. `Restore backup`
8. `Expand backup to designated size matching the yaml.`
9. `Create PV/PVC`
10. `Spin up replicas for application`
11. `All done`