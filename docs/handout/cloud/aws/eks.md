
## Elastic Kubernetes Service

!!! warning "Never spend your money before you have it, Jefferson T."

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



## References:

[^1]: [Setting up to use Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/setting-up.html){target='_blank'}

[^2]: [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/){:target="_blank"}

[^3]: :fontawesome-brands-youtube:{ .youtube } [Como criar um cluster Kubernetes na AWS com EKS](https://youtu.be/JrT5YV1KMeY){:target="_blank"} by [Fabricio Veronez](https://github.com/fabricioveronez){:target="_blank"}

    [![](https://img.youtube.com/vi/JrT5YV1KMeY/0.jpg){ width=60% }](https://youtu.be/JrT5YV1KMeY){:target="_blank"}

[^4]: [Creating a VPC for your Amazon EKS cluster](https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html){:target="_blank"}

[^5]: [AWS Princing Calculator - EKS](https://calculator.aws/#/createCalculator/EKS){:target="_blank"}