# k8s-v2-registry-POC

0. Go through the README steps in k8s-droidcraft/prepare/ to get kubernetes up and running(bare metal in this case). 

1. Both v2 registries will share one redis instance.  
https://github.com/Riptawr/k8s-droidcraft/blob/master/redis/redis.yml - set up redis, use persistent volume insreated of emptydir if needed.
Emptydir is used here for testing. 

2. Set up persistent volume and persistent voume claim to be used by docker registry replication controller  
https://github.com/Riptawr/k8s-droidcraft/tree/master/volumes 
This should be a storage shared among the kubernetes nodes, I used ASW EFS (nfs 4.1) for the sake of testing.  
Change the path in the yml files if needed. 

3. Create registry rc  https://github.com/Riptawr/k8s-droidcraft/blob/master/registry_v2/reg_rc.yml  (change HTTP-SECRET) and the registry service 
https://github.com/Riptawr/k8s-droidcraft/blob/master/registry_v2/reg_svc.yml

4. We need nginx ingress controller to track via api which registry endpoints are alive and update its config accordingy. 
https://github.com/Riptawr/k8s-droidcraft/tree/master/nginx_rbac/nginx
Once you have it up and running, ingress resource for registry can be created  - 
https://github.com/Riptawr/k8s-droidcraft/blob/master/registry_v2/reg_ingress.yml
Note that in production we need to specify a proper hostname for the registry in the ingress yml file. 

5. Once ingress resource is created, nginx config is reloaded.  
Configuration of the registry in nginx can be checked in ingress 
kubectl exec --namespace nginx-ingress nginx-ingress-controller-5bd4fc7ccf-ptvcb cat /etc/nginx/nginx.conf

6. We can manage the config of the registry in nginx via ingress resource annotations (check ingress yml once again) , or via configmap - 
https://github.com/Riptawr/k8s-droidcraft/tree/master/configmap
Annotations should take precedence.  

7. Note that https://github.com/Riptawr/k8s-droidcraft/blob/master/registry_v2/reg_ingress.yml 
has an ingress class specified, and it is also present in the nginx controller config args 
https://github.com/Riptawr/k8s-droidcraft/blob/master/nginx_rbac/nginx/nginx-ingress-controller.yml 
This is what makes nginx take care of this ingress resource. 
