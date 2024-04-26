
## Minikube

Versão light do kubernetes, para rodar em máquinas locais. [Instalação do Kubernetes](https://minikube.sigs.k8s.io/docs/start/){target="_blank"}.

Para Inicializar o Minikube após a instalação, utilize:

``` shell
minikube start --driver=docker --profile=store
```

``` shell
minikube profile list
```

``` shell
minikube delete --all
```

Dashboard
``` shell
minikube dashboard
```

## Kubectl

Comando cliente de gerenciamento do Kubernetes.

``` shell
kubectl apply -f <filename>
```

``` shell
kubectl get deployments
```

``` shell
kubectl get svc
```

``` shell
kubectl get pods
```

``` shell
kubectl port-forward <pod> 8080:8080
```

``` shell
kubectl exec -it <pod> -- bash
```

``` shell
kubectl delete --all
```

``` shell
kubectl api-resources
```

``` shell
kubectl logs <pod>
```

## Services

- ClusterIp: apenas dentro do cluster.

- NodePort: permite exposição de porta para fora do cluster.

- LoadBalance: uma porta para diversas instâncias no cluster.


## Deploying a Postgres 

Crie um novo repositório para armazenar as configurações do banco de dados: `platform.241.store.db`.

``` tree title="estrutura de diretório sugerida"
store.account
store.db
    k8s
        configmap.yaml
        credentials.yaml
        pv.yaml
        pvc.yaml
        deployment.yaml
        service.yaml
```
=== "configmap.yaml"

    Configuração de conexão do banco

    ``` yaml title='configmap.yaml'
    apiVersion: v1
    kind: ConfigMap
    metadata:
    name: postgres-configmap
    labels:
        app: postgres
    data:
        POSTGRES_HOST: postgres
        POSTGRES_DB: store
    ```

    ``` shell
    kubectl apply -f ./k8s/configmap.yaml
    kubectl get configmap
    ```

=== "credentials.yaml"

    Configuração de acesso ao banco

    ``` yaml title='credentials.yaml'
    apiVersion: v1
    kind: Secret
    metadata:
        name: postgres-credentials
    data:
        POSTGRES_USERNAME: c3RvcmU=
        POSTGRES_PASSWORD: c3RvcmU=
    ```

    ``` shell
    kubectl apply -f ./k8s/configmap.yaml
    kubectl get configmap
    ```

    Use encode base64 para ofuscar a senha. Vide: [Base64Encode](https://www.base64encode.org/).

=== "pv.yaml"

    Persistence Volume: espaço alocado no cluster

    ``` yaml title="pv.yaml"
    apiVersion: v1
    kind: PersistentVolume
    metadata:
    name: postgres-volume
    labels:
        type: local
        app: postgres
    spec:
    storageClassName: manual
    capacity:
        storage: 10Gi
    accessModes:
        - ReadWriteMany
    hostPath:
        path: /data/postgresql
    ```

    ``` shell
    kubectl apply -f ./k8s/pv.yaml
    kubectl get pv
    ```

=== "pvc.yaml"

    Persistence Volume Claim: espaço alocado do cluster para o pods.

    ``` yaml title="pvc.yaml"
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
    name: postgres-volume-claim
    labels:
        app: postgres
    spec:
    storageClassName: manual
    accessModes:
        - ReadWriteMany
    resources:
        requests:
        storage: 10Gi
    ```

    ``` shell
    kubectl apply -f ./k8s/pvc.yaml
    kubectl get pvc
    ```

=== "deployment.yaml"

    ``` yaml title="deployment.yaml"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: postgres
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: postgres
    template:
        metadata:
        labels:
            app: postgres
        spec:
        containers:
            - name: postgres
            image: 'postgres:latest'
            imagePullPolicy: IfNotPresent
            ports:
                - containerPort: 5432
            env:
                - name: DATABASE_DB
                valueFrom:
                    configMapKeyRef:
                    name: postgres-configmap
                    key: POSTGRES_DB
                - name: DATABASE_USERNAME
                valueFrom:
                    secretKeyRef:
                    name: postgres-credentials
                    key: POSTGRES_USERNAME
                - name: DATABASE_PASSWORD
                valueFrom:
                    secretKeyRef:
                    name: postgres-credentials
                    key: POSTGRES_PASSWORD
            volumeMounts:
                - mountPath: /var/lib/postgresql/data
                name: postgresdata
        volumes:
            - name: postgresdata
            persistentVolumeClaim:
                claimName: postgres-volume-claim
    ```

    ``` shell
    kubectl apply -f ./k8s/deployment.yaml
    kubectl get deployments
    kubectl get pods
    ```

=== "service.yaml"

    ``` yaml title="service.yaml"
    apiVersion: v1
    kind: Service
    metadata:
    name: postgres
    labels:
        app: postgres
    spec:
    type: NodePort
    ports:
        - port: 5432
    selector:
        app: postgres
    ```

    ``` shell
    kubectl apply -f ./k8s/service.yaml
    kubectl get services
    ```

Acessando o pod do Postgres:

``` shell
kubectl exec -it postgres-<pod-id> -- psql -h localhost -U store --password -p 5432 store
```

Redirecionando porta:
``` shell
kubectl port-forward <pod> 5432:5432
```

## Deploying a microservice

``` tree title="account"
store.account-resource
    src
        main
            resources
                application.yaml
    k8s
        deployment.yaml
        service.yaml
    Dockerfile
    Jenkins
    pom.xml
```

=== "application.yaml"

    ``` yaml title="application.yaml"
    server:
    port: 8080

    spring:
    application:
        name: account
    datasource:
        url: jdbc:postgresql://${DATABASE_HOST}:5432/${DATABASE_NAME}
        username: ${DATABASE_USERNAME:postgres}
        password: ${DATABASE_PASSWORD:Post123321}
        driver-class-name: org.postgresql.Driver
    flyway:
        baseline-on-migrate: true
        schemas: account
    jpa:
        properties:
        hibernate:
            default_schema: account

    management:
        endpoints:
            web:
                base-path: /account/actuator
                exposure:
                    include: [ 'prometheus' ]

    eureka:
        client:
            register-with-eureka: true
            fetch-registry: true
            service-url:
            defaultZone: ${EUREKA_URI:http://localhost:8761/eureka/}
    ```

    Subir no Git e rodar o Jenkins.

=== "deployment.yaml"

    ``` yaml title="deployment.yaml"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: account
    spec:
    selector:
        matchLabels:
        app: account
    replicas: 1
    template:
        metadata:
        labels:
            app: account
        spec:
        containers:
            - name: account
            image: humbertosandmann/account:latest
            ports:
                - containerPort: 8080
            env:
                - name: DATABASE_HOST
                valueFrom:
                    configMapKeyRef:
                    name: postgres-configmap
                    key: POSTGRES_HOST
                - name: DATABASE_DB
                valueFrom:
                    configMapKeyRef:
                    name: postgres-configmap
                    key: POSTGRES_DB
                - name: DATABASE_USERNAME
                valueFrom:
                    secretKeyRef:
                    name: postgres-credentials
                    key: POSTGRES_USERNAME
                - name: DATABASE_PASSWORD
                valueFrom:
                    secretKeyRef:
                    name: postgres-credentials
                    key: POSTGRES_PASSWORD
    ```


=== "service.yaml"

    ``` yaml title="service.yaml"
    apiVersion: v1
    kind: Service
    metadata:
    name: account
    labels:
        name: account
    spec:
    type: NodePort
    ports:
        - port: 8080
        targetPort: 8080
        protocol: TCP
    selector:
        app: account
    ```

    ``` shell
    kubectl apply -f ./k8s/service.yaml
    kubectl get services
    ```



## References:

[Using a Service to Expose Your App](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/){target='_blank'}

[Install Kubernetes's Tools](https://kubernetes.io/docs/tasks/tools/){target='_blank'}

[How to Deploy Postgres to Kubernetes Cluster](https://www.digitalocean.com/community/tutorials/how-to-deploy-postgres-to-kubernetes-cluster){target='_blank'}

[Spring boot, PostgreSQL and Kubernetes](https://medium.com/@dickanirwansyah/spring-boot-postgresql-kubernetes-e3eb726570bd)