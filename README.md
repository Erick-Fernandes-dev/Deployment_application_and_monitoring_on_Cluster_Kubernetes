# Desafio Esig para DevOps

## Etapa 1 - Docker

1. Baixe o arquivo WAR do Jenkins mais recente do site oficial ou de outra fonte
confiável.
    
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image.png)
    
    ```bash
    wget https://get.jenkins.io/war-stable/2.504.1/jenkins.war
    ```
    
2. Configure um servidor de aplicação local, como JBoss ou Tomcat.
    - Optei em usar o servidor do Tomcat
    
    ```bash
    sudo apt update
    
    # Caso tenha o Java instalado em sua máquina pule a etapa de instalação.
    sudo apt install openjdk-21-jdk -y 
    
    wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.40/bin/apache-tomcat-10.1.40.tar.gz
    ```
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%201.png)
    
    Arquivo descampactado com sucesso.
    
    ```bash
    tar -xzvf apache-tomcat-10.1.40.tar.gz
    ```
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%202.png)
    
    Agora mova esse arquivo.
    
    ```bash
    sudo mv apache-tomcat-10.1.40 /opt/tomcat 
    ```
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%203.png)
    
3. Implante o arquivo WAR do Jenkins no servidor configurado.
    
    
    Antes de enviar o arquivo WAR perceba que o arquivo jenkins.war não está descompactado no diretório /webapps
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%204.png)
    
    Copiar o WAR para o diretório **webapp** que está localizado no **`/opt/tomcat/webapps/`**
    
    ```bash
    cp jenkins.war /opt/tomcat/webapps 
    ```
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%205.png)
    
    Assim que executar o tomcat ele vai descompactar esse arquivo .war
    
    Vamos configurar um usuário e um grupo para o tomcat, caso não exista nenhum usuário ou grupo criado:
    
    ```bash
    # Criando um grupo para o nosso tomcat
    sudo groupadd tomcat
    
    # Criando um usuário "tomcat" sem acesso a shell
    sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
    ```
    
    Logo, em seguida, configuramos as permissões no Linux.
    
    ```bash
    sudo chown -R tomcat:tomcat /opt/tomcat
    ```
    
4. Inicie o servidor de aplicação e verifique se o Jenkins está sendo executado
corretamente.
    
    
    Iniciar servidor
    
    ```bash
    # Entre no modo sudo
    sudo su
    
    # Navegue até o diretório bin do tomcat
    cd /opt/tomcat/bin
    
    # Inicie o servidor
    ./startup.sh
    ```
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%206.png)
    
    Jenkins executado com sucesso, porém ainda falta um procedimento que é pega o pessword padrão para desbloquear o Jenkins de fato.
    
5. Acesse a interface web do Jenkins e verifique se é possível fazer login e
acessar as configurações básicas.
    
    
    Agora vamos usar um comando para pegar a senha para loar no jenkins
    
    ```bash
    cat /opt/tomcat/logs/catalina.out | grep "Please use the following password"
    ```
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%207.png)
    
    Localizei a senha temporária.
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%208.png)
    
    Instalando plugins
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%209.png)
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2010.png)
    
    Nosso Jenkins já está funcionando perfeitamente 😁
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2011.png)
    
6. Construa e execute o contêiner , garantindo que as métricas estejam
acessíveis via jolokia.

```bash
 wget -O jolokia-agent-jvm-2.2.9-javaagent.jar https://repo1.maven.org/maven2/org/jolokia/jolokia-agent-jvm/2.2.9/jolokia-agent-jvm-2.2.9-javaagent.jar
 wget https://get.jenkins.io/war-stable/2.504.1/jenkins.war
```

Dockerfile 🐳:

```docker
FROM tomcat:11.0.6-jdk21-temurin-jammy
COPY jolokia-agent-jvm-2.2.9-javaagent.jar /usr/local/tomcat/lib/jolokia-agent.jar
ENV CATALINA_OPTS="-javaagent:/usr/local/tomcat/lib/jolokia-agent.jar=port=8778,host=0.0.0.0"
EXPOSE 8080 8778

RUN rm -rf /usr/local/tomcat/webapps/*
COPY jenkins.war /usr/local/tomcat/webapps/ROOT.war

```

```docker
 docker login
 
 docker build -t docker.io/erickwolf/jenkins-tomcat-jolokia:latest .
 
 # Executar o container 
 docker container run -d -p 8080:8080 -p 8778:8778 docker.io/erickwolf/jenkins-tomcat-jolokia:latest 
 
```

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2012.png)

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2013.png)

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2014.png)

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2015.png)

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2016.png)

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2017.png)

Agora vamos dar um push para o docker hub

```docker
docker push docker.io/erickwolf/jenkins-tomcat-jolokia:latest
```

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2018.png)

## Etapa 2 - Kubernetes

Tarefa: Execute a atividade da Etapa 1 em um cluster kubernetes

### 🚨Estou usando o kind para criar o cluster

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2019.png)

Configuração do meu kind

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30080 # Porta do Jenkins
        hostPort: 30080 # Porta do host
        protocol: TCP
      - containerPort: 30778 # Porta do Jolokia
        hostPort: 30778 # Porta do host
        protocol: TCP
```

```docker
kubectl get nodes -owide
```

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2020.png)

1. Crie os manifestos YAML necessários para implantar a aplicação no
kubernetes, incluindo Deployment e Service.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deployment 
spec:
  replicas: 1 
  selector:
    matchLabels:
      app: jenkins # Deve corresponder aos labels no template do Pod
  template:
    metadata:
      labels:
        app: jenkins # Label para identificar os Pods
        metrics: "jolokia" # Label para identificar que expõe métricas via Jolokia
    spec:
      containers:
      - name: jenkins-container
        image: erickwolf/jenkins-tomcat-jolokia:latest
        ports:
        - containerPort: 8080 # Porta do Jenkins/Tomcat
          name: http
        - containerPort: 8778 # Porta do Jolokia
          name: jolokia
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"

---

apiVersion: v1
kind: Service
metadata:
  name: jenkins-service 
spec:
  type: NodePort # NodePort ou LoadBalancer
  selector:
    app: jenkins 
  ports:
    - name: http
      protocol: TCP
      port: 8080 # Porta interna do Service
      targetPort: 8080 # Porta do contêiner (Tomcat/Jenkins)
      nodePort: 30080
    - name: jolokia
      protocol: TCP
      port: 8778 # Porta interna do Service para Jolokia
      targetPort: 8778 # Porta do contêiner (Jolokia)
      nodePort: 30778

```

Navegue até o diretório **`/kubernetes/toncat_jalokia_jenkins`**

```yaml
kubectl apply -f .
kubectl get po -w
```

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2021.png)

```yaml
kubectl get svc 
```

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2022.png)

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2023.png)

Aplicação rodando e estou utilizando o k9s que é uma ferramenta bastante útil para vizualizar nossas aplicações no Cluster.

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2024.png)

```yaml

curl http://localhost:30778/jolokia/version
curl http://localhost:30778/jolokia/version -I
```

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2025.png)

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2026.png)

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2027.png)

Copi e cole essa senha para desbloquear Jenkins

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2028.png)

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2029.png)

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2030.png)

Jenkins

![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2031.png)

Jenkins e Jolokia executado no Kubernetes com sucesso 😁

## Etapa 3 - Monitoramento

Tarefa: Configurar o Prometheus para coletar métricas via Jolokia e do Node
Exporter.

1. Implante o Prometheus no cluster Kubernetes.
    
    
    YAML do prometheus
    
    ```yaml
    
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: prometheus
      labels:
        app: prometheus
      namespace: monitoring
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: prometheus
      template:
        metadata:
          labels:
            app: prometheus
        spec:
          containers:
            - name: prometheus
              image: prom/prometheus:latest
              args:
                - "--config.file=/etc/prometheus/prometheus.yml"
                - "--storage.tsdb.path=/prometheus/"
              ports:
                - containerPort: 9090
                  protocol: TCP
              volumeMounts:
                - name: prometheus-config-volume
                  mountPath: /etc/prometheus/
                - name: prometheus-storage-volume
                  mountPath: /prometheus/
              resources: {}
          volumes:
            - name: prometheus-config-volume
              configMap:
                defaultMode: 420
                name: prometheus-config
      
            - name: prometheus-storage-volume
              emptyDir: {}
    
    --- 
    
    apiVersion: v1
    kind: Service
    metadata:
      name: prometheus
      annotations:
          prometheus.io/scrape: 'true'
          prometheus.io/port:   '9090'
      namespace: monitoring
    spec:
      selector:
        app: prometheus
      ports:
        - protocol: TCP
          name: prometheus
          port: 9090
          targetPort: 9090
    
    ---
    
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: prometheus-config
      namespace: monitoring
    data:
      prometheus.yml: |-
        # my global config
        global:
          scrape_interval: 15s
          evaluation_interval: 30s
          scrape_timeout: "10s"
    
          #external_labels:
          #  monitor: codelab
    
        scrape_configs: 
          - job_name: 'Prometheus'
            static_configs:
              - targets: ['localhost:9090']
                labels:
                  grupo: 'Prometheus'
    			# Coletando as métricas do nosso node internamente, para evitar a 
    			# a exposição publica das métricas
          - job_name: 'Node Exporter'         
            static_configs:    
              - targets: ['node-exporter.monitoring.svc.cluster.local:9100']
                labels:
                  grupo: 'Node Exporter'
    
    ```
    
    Prometheus executado com sucesso
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2032.png)
    
2. Configure o Prometheus para coletar métricas da aplicação utilizando o
endpoint Jolokia.
    
    
    Primeiro, é necessário que você configure o jolokia exporter, abaixo, teremos um manifesto completo, com deployment, service e configMap
    
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: jolokia-exporter-config
      namespace: default
    data:
      config.yaml: |
        metrics:
        - source:
            mbean: java.lang:type=Memory
            attribute: HeapMemoryUsage
            path: used
          target: java_memory_heap_memory_usage_used
        - source:
            mbean: java.lang:type=Memory
            attribute: HeapMemoryUsage
            path: max
          target: java_memory_max
        - source:
            mbean: java.lang:type=Threading
            attribute: ThreadCount
          target: java_threading_thread_count
        - source:
            mbean: java.lang:type=OperatingSystem
          target: java_os
    
    ---
    
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: jolokia-exporter
      namespace: default
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: jolokia-exporter
      template:
        metadata:
          labels:
            app: jolokia-exporter
        spec:
          containers:
            - name: jolokia-exporter
              image: scalify/jolokia_exporter:latest
              args:
                - "export"
                - "/config/config.yaml"
                - "http://jenkins-service.default.svc.cluster.local:8778/jolokia/read"
              ports:
                - containerPort: 9422
                  name: metrics
              volumeMounts:
                - name: config-volume
                  mountPath: /config
                  readOnly: true
          volumes:
            - name: config-volume
              configMap:
                name: jolokia-exporter-config
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: jolokia-exporter
      namespace: default
    spec:
      ports:
        - port: 9422
          targetPort: 9422
          protocol: TCP
      selector:
        app: jolokia-exporter
      type: ClusterIP
    ```
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2033.png)
    
    Criei um Pod do alpine para fazer alguns testes de comunicação entre Pods
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2034.png)
    
    Configuração do Prometheus para coletar as métricas do Jolokia.
    
    ```yaml
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: prometheus
      labels:
        app: prometheus
      namespace: monitoring
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: prometheus
      template:
        metadata:
          labels:
            app: prometheus
        spec:
          containers:
            - name: prometheus
              image: prom/prometheus:latest
              args:
                - "--config.file=/etc/prometheus/prometheus.yml"
                - "--storage.tsdb.path=/prometheus/"
              ports:
                - containerPort: 9090
                  protocol: TCP
              volumeMounts:
                - name: prometheus-config-volume
                  mountPath: /etc/prometheus/
                - name: prometheus-storage-volume
                  mountPath: /prometheus/
              resources: {}
          volumes:
            - name: prometheus-config-volume
              configMap:
                defaultMode: 420
                name: prometheus-config
      
            - name: prometheus-storage-volume
              emptyDir: {}
    
    --- 
    
    apiVersion: v1
    kind: Service
    metadata:
      name: prometheus
      annotations:
          prometheus.io/scrape: 'true'
          prometheus.io/port:   '9090'
      namespace: monitoring
    spec:
      selector:
        app: prometheus
      ports:
        - protocol: TCP
          name: prometheus
          port: 9090
          targetPort: 9090
    
    ---
    
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: prometheus-config
      namespace: monitoring
    data:
      prometheus.yml: |-
        # my global config
        global:
          scrape_interval: 15s
          evaluation_interval: 30s
          scrape_timeout: "10s"
    
          #external_labels:
          #  monitor: codelab
    
        scrape_configs: 
          - job_name: 'Prometheus'
            static_configs:
              - targets: ['localhost:9090']
                labels:
                  grupo: 'Prometheus'
    
          - job_name: 'Node Exporter'         
            static_configs:    
              - targets: ['node-exporter.monitoring.svc.cluster.local:9100']
                labels:
                  grupo: 'Node Exporter'
    
          - job_name: 'Jolokia Exporter'
            static_configs:
              - targets: ['jolokia-exporter.default.svc.cluster.local:9422']
                labels:
                  grupo: 'Jolokia Exporter'
    
    ```
    
    ```yaml
    kubectl port-forward pod/prometheus-68dc8567f7-wg8zg 9090:9090 -n monitoring
    ```
    
    Entre na interface do Prometheus
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2035.png)
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2036.png)
    
3. Implante o Node Exporter para coletar métricas dos nós do cluster.
    
    
    Para fazer a colete de métricas do Node, é necessário que você crie Daemonset que será responsável por criar Pods em todos os Nós do cluster. Segue abaixo as configurações para a coleta de métricas do Node Exporter:
    
    ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      labels:
        app.kubernetes.io/component: exporter
        app.kubernetes.io/name: node-exporter
      name: node-exporter
      namespace: monitoring
    spec:
      selector:
        matchLabels:
          app.kubernetes.io/component: exporter
          app.kubernetes.io/name: node-exporter
      template:
        metadata:
          labels:
            app.kubernetes.io/component: exporter
            app.kubernetes.io/name: node-exporter
        spec:
          containers:
          - args:
            - --path.sysfs=/host/sys
            - --path.rootfs=/host/root
            - --no-collector.wifi
            - --no-collector.hwmon
            - --collector.filesystem.ignored-mount-points=^/(dev|proc|sys|var/lib/docker/.+|var/lib/kubelet/pods/.+)($|/)
            - --collector.netclass.ignored-devices=^(veth.*)$
            name: node-exporter
            image: prom/node-exporter
            ports:
              - containerPort: 9100
                protocol: TCP
            resources:
              limits:
                cpu: 250m
                memory: 180Mi
              requests:
                cpu: 102m
                memory: 180Mi
            volumeMounts:
            - mountPath: /host/sys
              mountPropagation: HostToContainer
              name: sys
              readOnly: true
            - mountPath: /host/root
              mountPropagation: HostToContainer
              name: root
              readOnly: true
          volumes:
          - hostPath:
              path: /sys
            name: sys
          - hostPath:
              path: /
            name: root
    
    --- 
    
    kind: Service
    apiVersion: v1
    metadata:
      name: node-exporter
      namespace: monitoring
      annotations:
          prometheus.io/scrape: 'true' # Vai fazer a raspagem das métricas
          prometheus.io/port:   '9100'
    spec:
      selector:
          app.kubernetes.io/component: exporter
          app.kubernetes.io/name: node-exporter
      ports:
      - name: node-exporter
        protocol: TCP
        port: 9100
        targetPort: 9100
    
    ```
    
    Configuração do Prometheus
    
    ```yaml
    
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: prometheus
      labels:
        app: prometheus
      namespace: monitoring
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: prometheus
      template:
        metadata:
          labels:
            app: prometheus
        spec:
          containers:
            - name: prometheus
              image: prom/prometheus:latest
              args:
                - "--config.file=/etc/prometheus/prometheus.yml"
                - "--storage.tsdb.path=/prometheus/"
              ports:
                - containerPort: 9090
                  protocol: TCP
              volumeMounts:
                - name: prometheus-config-volume
                  mountPath: /etc/prometheus/
                - name: prometheus-storage-volume
                  mountPath: /prometheus/
              resources: {}
          volumes:
            - name: prometheus-config-volume
              configMap:
                defaultMode: 420
                name: prometheus-config
      
            - name: prometheus-storage-volume
              emptyDir: {}
    
    --- 
    
    apiVersion: v1
    kind: Service
    metadata:
      name: prometheus
      annotations:
          prometheus.io/scrape: 'true'
          prometheus.io/port:   '9090'
      namespace: monitoring
    spec:
      selector:
        app: prometheus
      ports:
        - protocol: TCP
          name: prometheus
          port: 9090
          targetPort: 9090
    
    ---
    
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: prometheus-config
      namespace: monitoring
    data:
      prometheus.yml: |-
        # my global config
        global:
          scrape_interval: 15s
          evaluation_interval: 30s
          scrape_timeout: "10s"
    
          #external_labels:
          #  monitor: codelab
    
        scrape_configs: 
          - job_name: 'Prometheus'
            static_configs:
              - targets: ['localhost:9090']
                labels:
                  grupo: 'Prometheus'
    
          - job_name: 'Node Exporter'         
            static_configs:    
              - targets: ['node-exporter.monitoring.svc.cluster.local:9100']
                labels:
                  grupo: 'Node Exporter'
    
          - job_name: 'Jolokia Exporter'
            static_configs:
              - targets: ['jolokia-exporter.default.svc.cluster.local:9422']
                labels:
                  grupo: 'Jolokia Exporter'
    
    ```
    
    Vamos fazer um teste de comunicação no nosso Pod Alpine
    
    ```bash
    curl node-exporter.monitoring.svc.cluster.local:9100/metrics
    ```
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2037.png)
    
    Coleta de métricas do Node Exporter realizada com sucesso 😁
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2038.png)
    
4. Garanta que as métricas coletadas estejam acessíveis no Prometheus.
    
    
    Todas as métricas acessíveis no Prometheus
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2039.png)
    
    Manifesto YAML do grafana:
    
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: grafana-datasources
      namespace: monitoring
    data:
      datasources.yaml: |-
        apiVersion: 1
        datasources:
          - name: Prometheus
            type: prometheus
            access: proxy
            url: http://prometheus.monitoring.svc.cluster.local:9090
            isDefault: true
    
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: grafana
      namespace: monitoring
      labels:
        app: grafana
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: grafana
      template:
        metadata:
          labels:
            app: grafana
        spec:
          containers:
            - name: grafana
              image: grafana/grafana:latest
              ports:
                - containerPort: 3000
                  protocol: TCP
              env:
                - name: GF_SECURITY_ADMIN_USER
                  value: admin
                - name: GF_SECURITY_ADMIN_PASSWORD
                  value: admin
              volumeMounts:
                - name: grafana-storage
                  mountPath: /var/lib/grafana
                - name: grafana-datasources
                  mountPath: /etc/grafana/provisioning/datasources
          volumes:
            - name: grafana-storage
              emptyDir: {}
            - name: grafana-datasources
              configMap:
                name: grafana-datasources
    ---
    
    apiVersion: v1
    kind: Service
    metadata:
      name: grafana
      namespace: monitoring
    spec:
      selector:
        app: grafana
      ports:
        - protocol: TCP
          port: 3000
          targetPort: 3000
      type: ClusterIP
    
    ---
    
    ```
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2040.png)
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2041.png)
    
    ![image.png](Desafio%20Esig%20para%20DevOps%201e81e83b35a08027b5b8f3ba66a4ec9d/image%2042.png)
    
    ---

● Os (as) aprovados (as) após avaliação do vídeo participarão da segunda fase
para a apresentação técnica do projeto.

● Durante a demonstração em vídeo, caso você tenha adotado, cite os pontos
que você usou ferramentas de IA para agregar valor ao seu projeto.
