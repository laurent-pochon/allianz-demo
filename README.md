# allianz-demo

# EKS / terraform
The cluster has been provisioned by terraform. it is base on kubernetes 1.26, the latest stable version. The terraform repository is here : https://github.com/laurent-pochon/allianz-demo-infra

here are the namespaces :

kubectl get namespaces                   
NAME              STATUS   AGE
argo-rollouts     Active   23h
blue-green        Active   44h
canary            Active   62m
default           Active   46h
jenkins           Active   22h
kube-node-lease   Active   46h
kube-public       Active   46h
kube-system       Active   46h

kubectl get all -n jenkins
NAME            READY   STATUS    RESTARTS        AGE
pod/jenkins-0   2/2     Running   4 (6h31m ago)   21h

NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP                                                                   PORT(S)        AGE
service/jenkins         LoadBalancer   172.20.231.4    a5e7ee3b8831a46e297778b49de74482-867282172.ap-southeast-1.elb.amazonaws.com   80:31062/TCP   21h
service/jenkins-agent   ClusterIP      172.20.146.62   <none>                                                                        50000/TCP      21h

NAME                       READY   AGE
statefulset.apps/jenkins   1/1     21h

kubectl get all -n argo-rollouts
NAME                                           READY   STATUS    RESTARTS   AGE
pod/argo-rollouts-558b87c8d7-qrpqm             1/1     Running   0          23h
pod/argo-rollouts-558b87c8d7-rxw4g             1/1     Running   0          23h
pod/argo-rollouts-dashboard-6676cd486d-qfsrq   1/1     Running   0          23h

NAME                              TYPE           CLUSTER-IP     EXTERNAL-IP                                                                    PORT(S)        AGE
service/argo-rollouts-dashboard   LoadBalancer   172.20.103.6   aac9b365f58b44140a0ee2e6f9ae2883-1292865551.ap-southeast-1.elb.amazonaws.com   80:32334/TCP   23h

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argo-rollouts             2/2     2            2           23h
deployment.apps/argo-rollouts-dashboard   1/1     1            1           23h

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/argo-rollouts-558b87c8d7             2         2         2       23h
replicaset.apps/argo-rollouts-dashboard-6676cd486d   1         1         1       23h

kubectl get all -n canary       
NAME                                       READY   STATUS    RESTARTS   AGE
pod/allianz-demo-canary-847c4f94d9-ctvnx   1/1     Running   0          55m
pod/allianz-demo-canary-847c4f94d9-j5q7d   1/1     Running   0          56m
pod/allianz-demo-canary-847c4f94d9-srs5g   1/1     Running   0          56m
pod/allianz-demo-canary-847c4f94d9-wwfg5   1/1     Running   0          56m
pod/allianz-demo-canary-847c4f94d9-z9b7p   1/1     Running   0          56m

NAME                     TYPE           CLUSTER-IP      EXTERNAL-IP                                                                    PORT(S)        AGE
service/canary-service   LoadBalancer   172.20.80.38    a95f6154b644a48f6b8a8ab5f61b7b7e-1797955811.ap-southeast-1.elb.amazonaws.com   80:30006/TCP   60m
service/root-service     LoadBalancer   172.20.42.42    ade0d163f067447c6ab63ca486a2d9e5-1085723179.ap-southeast-1.elb.amazonaws.com   80:32282/TCP   60m
service/stable-service   LoadBalancer   172.20.17.251   a6bdf0003274d4381b19aae33581f092-27856421.ap-southeast-1.elb.amazonaws.com     80:30367/TCP   60m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/allianz-demo-canary-847c4f94d9   5         5         5       60m

kubectl get all -n blue-green
NAME                                     READY   STATUS    RESTARTS   AGE
pod/allianz-demo-blue-5b9479c57c-rpq5m   1/1     Running   0          19m
pod/allianz-demo-blue-5b9479c57c-sbbd9   1/1     Running   0          19m

NAME                      TYPE           CLUSTER-IP       EXTERNAL-IP                                                                    PORT(S)        AGE
service/active-service    LoadBalancer   172.20.198.242   a2d1cee07fe2e49dab27cdcba38f7231-1830196260.ap-southeast-1.elb.amazonaws.com   80:30542/TCP   20m
service/preview-service   LoadBalancer   172.20.225.70    abf1805361b5d4954bb08e99fdcd22b7-820380043.ap-southeast-1.elb.amazonaws.com    80:31660/TCP   20m

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/allianz-demo-blue-5b9479c57c   2         2         2       20m

# CI (with github actions)
The docker build and deployment to ECR is done by github actions in this repo. I started to do it with jenkins in my EKS cluster but i saw i would be short in time as my jenkins server is running in EKS. So i would need a kaniko agent to build and deploy the image. I think it is too long to implement in a test like it. 

The jenkins instance is deployed with helm. I did not attach an EFS to it which is a problem. It was just a time issue. the instance is here : http://a5e7ee3b8831a46e297778b49de74482-867282172.ap-southeast-1.elb.amazonaws.com . User admin, pass allianz . It is functional, but pretty much empty.

The CI is functional but implemented in github.

# CD (with argo rollouts)

The deployment is done with argo rollouts which completly make sense base on the fact we want to deploy using canary and blue/green.

The deployment can be controlled by argo rollouts : http://aac9b365f58b44140a0ee2e6f9ae2883-1292865551.ap-southeast-1.elb.amazonaws.com/rollouts/ the select the namespace canary or blue / green.

here are the app url.

canary : http://ade0d163f067447c6ab63ca486a2d9e5-1085723179.ap-southeast-1.elb.amazonaws.com
blue-green : http://a2d1cee07fe2e49dab27cdcba38f7231-1830196260.ap-southeast-1.elb.amazonaws.com

# Conclusion

I did not do it the way i expected to do it. Normally Jenkins should have been in charge of building the docker image and pushing it to ECR. To do it i would have used kaniko. But when i saw it took me time to set it up, i though i could not finish it in time and i was right. I decided to focus on have a deployement to show.
I hope it won't be one issue. This test took me 15-20 hours already (i did not do this kind of stuff for almost 3 years now, it took me some time to get it going), in my current situation i can not spend more time on it. i have not been paid for 5 month now. In addition to my current job that i continue to do, i need to find other ways to get some money coming.




