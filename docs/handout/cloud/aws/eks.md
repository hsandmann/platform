
## Elastic Kubernetes Service

!!! quote "Never spend your money before you have it, Jefferson T."

    EKS não tem cota grátis, sempre é muito bem cobrado.



## Rise up an EKS

---

### 1. Creating a role

IAM - Identity and Access Management: gerencia usuários e acessos.

Role é um grupo de policiies que estão vinculadas a serviços AWS, assim, o EKS precisa de permissionamento para acessar os recursos da AWS.

![](../../../assets/images/cloud.aws.eks.role.1.png)

![](../../../assets/images/cloud.aws.eks.role.2.png)

![](../../../assets/images/cloud.aws.eks.role.3.png)

---

### 2. Creating a VPC
Virtual Private Cloud

Organização do Kubernetes

<figure markdown="span">
    ![](https://kubernetes.io/images/docs/components-of-kubernetes.svg)
    <figcaption>Kubernetes Components [^2]</figcaption>
</figure>

É necessário criar uma estrutura de rede para suportar o Kubernetes, para isso, é aconselhável utilizar um template do Cloud Formation. Abaixe o arquivo [amazon-eks-vpc-private-subnets.yaml](../../../assets/templates/amazon-eks-vpc-private-subnets.yaml){:download="amazon-eks-vpc-private-subnets.yaml"} e dê um *upload* na criação da VPC.

![](../../../assets/images/cloud.aws.eks.vpc.1.png)


``` shell
https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
```

![](../../../assets/images/cloud.aws.eks.vpc.2.png)

![](../../../assets/images/cloud.aws.eks.vpc.3.png)


``` mermaid
flowchart TB
  subgraph Region
    direction LR
    subgraph Zone A
      direction LR
      subgraph subpri1["Subnet Private"]
        direction TB
        poda1["pod 1"]
        poda2["pod 2"]
        poda3["pod 3"]
      end
      subgraph subpub1["Subnet Public"]
        loadbalancea["Load Balance"]
      end
    end
    subgraph Zone B
      direction LR
      subgraph subpri2["Subnet Private"]
        direction TB
        podb1["pod 1"]
        podb2["pod 2"]
        podb3["pod 3"]
      end
      subgraph subpub2["Subnet Public"]
        loadbalanceb["Load Balance"]
      end
    end
    User --> loadbalancea
    loadbalancea --> poda1
    loadbalancea --> poda2
    loadbalancea --> poda3
    User --> loadbalanceb
    loadbalanceb --> podb1
    loadbalanceb --> podb2
    loadbalanceb --> podb3
  end
```

gateway --> auth
gateway --> discovery

### 3. Building an EKS

![](../../../assets/images/cloud.aws.eks.1.png)

![](../../../assets/images/cloud.aws.eks.2.png)


### 4. Accessing the EKS

On terminal, after that it had been set up the aws cli.

``` shell
aws configure
```

See the configuration that was done.

``` shell
aws configure list
```
<!-- termynal -->
``` shell
> aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************TTNI shared-credentials-file    
secret_key     ****************zAJ1 shared-credentials-file    
    region                us-east-2      config-file    ~/.aws/config
```

Set up the kube-config to point to the remote aws eks cluster.

``` shell
aws eks update-kubeconfig --name eks-store
```
<!-- termynal -->
``` shell
> aws eks update-kubeconfig --name eks-store
Added new context arn:aws:eks:us-east-2:058264361068:cluster/eks-store to /Users/sandmann/.kube/config
>
>
> kubectl get pods
No resources found in default namespace.
>
>
> kubectl get nodes
No resources found
>
```

Come back to AWS EKS > compute:

![](../../../assets/images/cloud.aws.eks.nodes.group.1.png)

Notice that there no nodes on cluster also, because only the **Control Pane** had been created, there is no exist a node for the worker nodes.

Attach roles to node group, it is exclusive for the worker nodes.

> IAM > Roles

![](../../../assets/images/cloud.aws.eks.nodes.group.2.png)

> Add Permissions

- AmazonEKS_CNI_Policy (*Configuration Network Interface*)
- AmazonEKSWorkerNodePolicy
- AmazonEC2ContainerRegistryReadOnly

> Review

![](../../../assets/images/cloud.aws.eks.nodes.group.3.png)

> Group Node Group

![](../../../assets/images/cloud.aws.eks.nodes.group.4.png)

![](../../../assets/images/cloud.aws.eks.nodes.group.5.png)

Only private subnets:

![](../../../assets/images/cloud.aws.eks.nodes.group.6.png)


``` shell
kubectl get nodes
```
<!-- termynal -->
``` shell
> kubectl get nodes
NAME                                            STATUS   ROLES    AGE   VERSION
ip-192-168-179-174.us-east-2.compute.internal   Ready    <none>   54s   v1.29.3-eks-ae9a62a
ip-192-168-204-234.us-east-2.compute.internal   Ready    <none>   54s   v1.29.3-eks-ae9a62a
```

Now, deploy the microservice.

<!-- termynal -->
``` shell
> kubectl apply -f ./k8s/deployment.yaml
deployment.apps/gateway created
> kubectl apply -f ./k8s/service.yaml
service/gateway created
>
>
>
> kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/gateway-7894679df8-lbngj   1/1     Running   0          81s

NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP                                                               PORT(S)          AGE
service/gateway      LoadBalancer   10.100.245.4   a3a5cc62ba81e466e9746f64f83fc349-1127848642.us-east-2.elb.amazonaws.com   8080:32681/TCP   25m
service/kubernetes   ClusterIP      10.100.0.1     <none>                                                                    443/TCP          87m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gateway   1/1     1            1           82s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/gateway-7894679df8   1         1         1       82s
>
```

!!! note "Jenkins update"

    Jenkins precisa instalar o awscli (adicionar ao `docker-compose.yaml`)
    ```yaml
    RUN apt-get install -y awscli
    ```

    Dentro da instância, configurar:

    ```shell
    > aws configure
    > aws eks update-kubeconfig --name eks-store
    ```

!!! info "Scale"

    ```shell
    > kubectl scale --replicas=3 deployment/gateway
    ```
    <!-- termynal -->
    ```shell
    > kubectl scale --replicas=3 deployment/gateway
    deployment.apps/gateway scaled
    > kubectl get pods
    NAME                       READY   STATUS    RESTARTS   AGE
    gateway-7894679df8-62m7z   1/1     Running   0          12s
    gateway-7894679df8-r2kp2   1/1     Running   0          12s
    gateway-7894679df8-v6xhs   1/1     Running   0          5m58s
    ```

## References:

[^1]: [Setting up to use Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/setting-up.html){target='_blank'}

[^2]: [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/){:target="_blank"}

[^3]: :fontawesome-brands-youtube:{ .youtube } [Como criar um cluster Kubernetes na AWS com EKS](https://youtu.be/JrT5YV1KMeY){:target="_blank"} by [Fabricio Veronez](https://github.com/fabricioveronez){:target="_blank"}

    [![](https://img.youtube.com/vi/JrT5YV1KMeY/0.jpg){ width=60% }](https://youtu.be/JrT5YV1KMeY){:target="_blank"}

[^4]: [Creating a VPC for your Amazon EKS cluster](https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html){:target="_blank"}

[^5]: [AWS Princing Calculator - EKS](https://calculator.aws/#/createCalculator/EKS){:target="_blank"}

[^6]: [Getting started with Amazon EKS – AWS Management Console and AWS CLI](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html){:target='_blank'}

[^7]: [kubectl scale](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_scale/){:target='blank'}
