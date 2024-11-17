# Prometheus & Grafana no kubernetes 
- aplique o arquivo manifesto do prometheus e grafana
- digite o seguinte comando para verificar em qual porta o prometheus está rodando
```bash
kubectl get all

# identifique o service do prometheus
NAME                                                 READY   STATUS              RESTARTS   AGE
pod/grafana-6df66cf9d9-k5bk6                         0/1     ContainerCreating   0          36s
pod/prometheus-kube-state-metrics-75b5bb4bf8-tbvhs   1/1     Running             0          36s
pod/prometheus-prometheus-node-exporter-zjvb9        1/1     Running             0          36s
pod/prometheus-server-cdfc5b579-c65vt                0/2     ContainerCreating   0          36s

NAME                                          TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/grafana                               LoadBalancer   10.104.162.74    <pending>     80:31924/TCP   36s
service/kubernetes                            ClusterIP      10.96.0.1        <none>        443/TCP        39m
service/prometheus-kube-state-metrics         ClusterIP      10.103.111.151   <none>        8080/TCP       36s
service/prometheus-prometheus-node-exporter   ClusterIP      10.111.236.174   <none>        9100/TCP       36s
service/prometheus-server                     LoadBalancer   10.103.92.246    <pending>     80:31868/TCP   36s

NAME                                                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/prometheus-prometheus-node-exporter   1         1         1       1            1           kubernetes.io/os=linux   36s

NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana                         0/1     1            0           36s
deployment.apps/prometheus-kube-state-metrics   1/1     1            1           36s
deployment.apps/prometheus-server               0/1     1            0           36s

NAME                                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-6df66cf9d9                         1         1         0       36s
replicaset.apps/prometheus-kube-state-metrics-75b5bb4bf8   1         1         1       36s
replicaset.apps/prometheus-server-cdfc5b579                1         1         0       36s

# service:
service/prometheus-server                     LoadBalancer   10.103.92.246    <pending>     80:31868/TCP   36s

# entre na porta que ele estiver utilizando o ip do minikube, ou ip do node
http://192.168.49.2:31868/

``` 
## Dashboard - Prometheus

 - **Alerts:** Configurações de alertas

 - **Graph:** Consulta das metricas, consegue consultar tanto em tables e graph

 - Mais informações, parte de configuração do prometheus para verificar de qual metricas puxar, regras... etc

 - **Runtime & Build Information:** Configurações do RUntime e Build da versão do prometheus que está rodando.

 - **TSDB Status:** Informações do banco, caso tenha

 - **Command-Line Flags:** Informações sobre os paramatros utilizados do arquivo yaml para iniciar o prometheus 

 - **Configuration:** Arquivo de configuração do prometheus

 - **Rules** Regras para manipular as metricas, pegar alguma metrica e transformar em outro tipo de métricas

 - **Targets:** Pontos de coleções da métrica

 - **Service Discovery:** Não faço ideia

## Targets
Clicando em status/target note que os targets está coletando as seguintes métricas:

- kubernetes-apiservers (1/1 up)
- kubernetes-nodes (1/1 up)
- kubernetes-nodes-cadvisor (1/1 up)
- kubernetes-nodes-cadvisor (1/1 up)
- prometheus (1/1 up)

O Prometheus em si, coleta metricas do kubernetes e até dele mesmo. Se adicionar **metrics** na url da seguinte forma:
http://192.168.49.2:31868/metrics e apertar enter, vai retornar as metricas do prometheus


## PromQL
PromQL, é uma linguagem do prometheus utilizada para coletar as metricas. As métricas, são coletadas por minuto.

- Exemplo de uso do PromQL coletando métricas do próprio prometheus:
- Verificnado o total de requisições http, conforme o link citado acima, pesquise por **prometheus_http_requests_total** utiliznado o **CTRL+F** e verá a quantidade de métricas de requisições coletadas. 

Na dashboard do prometheus, na aba table, cole o nome **prometheus_http_requests_total** na barra de pesquisa das metricas para verificar todas as métricas com este nome. 

Cada informação mostrada na tela, é um instant Vector, cada métrica mostrada são métricas capturada em **tempo real**. Como é em tempo real, consegue verificar as métricas de segundos, minutos e horas atrás clicando no campo **Evaluation time**. É possível observar essas métricas em tempo real, basta clicar apenas em **Graph**

- Pegando uma métrica, um caminho específico atráves de labels.

## Labels
No Prometheus, labels são pares de chave-valor associados a métricas, usados para distinguir diferentes instâncias de uma métrica. Eles permitem uma categorização mais detalhada dos dados coletados, facilitando consultas específicas e análises detalhadas.

 - Por exemplo, considere a métrica:
 ```bash
prometheus_http_requests_total{handler="/metrics"} # clique em execute
```

- **http_requests_total** é o nome da métrica.
* **Labels:** handler="/metrics"
- **/metrics:** Não é uma métrica, é o valor de um label **(handler)** neste caso específico. Ele indica qual endpoint está sendo monitorado

- Essa configuração indica que o Prometheus está coletando o total de requisições HTTP feitas especificamente ao endpoint /metrics.

- Com base nesse label, Filtrar requisições apenas para esse handler específico.

- Comparar com outras rotas monitoradas pelo Prometheus.

- É possível verificar labels atráves de **expressões regulares**. Exemplo:

```bash
prometheus_http_requests_total{handler=~"/metrics|/api/v1/labels"} # clique em execute

# Vai mostrar registros de **metrics** e **/api/v1/labels**
```


Também funciona com o **diferente** de, Exemplo: 
```bash
prometheus_http_requests_total{handler!~"/metrics|/api/v1/labels"} # clique em execute

# vai mostrar todos os registros que não seja esses endpoint: **"/metrics|/api/v1/labels"**
```


### Pegando uma métrica de um determinado período. ( Range Vector)
- Neste exemplo, vai ser uma métrica do momento atual até 10 minutos atrás
```bash
prometheus_http_requests_total{handler="/metrics"}[10m] # clique em execute

# note que vai retornar 10 registros nos ultimos 10min 
44 @1731733963.054
45 @1731734023.054
46 @1731734083.054
47 @1731734143.054
48 @1731734203.054
49 @1731734263.054
50 @1731734323.054
51 @1731734383.054
52 @1731734443.054
53 @1731734503.054
```

- Verificando as chamadas de /metrics por segundos no gráfico. Basta apenas adicionar uma rate
```bash
rate(prometheus_http_requests_total{handler="/metrics"}[10m])  # clique em execute

```

- Verificando de a soma de todos os endpoint no gráfico
```bash
sum(rate(prometheus_http_requests_total[10m]))
```

- Agrupando por instancias, que neste caso é uma label específica **handler** no gráfico
```bash
sum(rate(prometheus_http_requests_total[10m])) by (handler)

sum(rate(prometheus_http_requests_total[10m])) by (instance)  # por instancia

sum(rate(prometheus_http_requests_total[10m])) by (handler,instance) # por handler e instancia
```

### Configurando a aplicação para que o prometheus consiga capturar metricas do objeto

Para que isso seja possível, é preciso adicionar uma annotations ao pod. Precisa ser identado da seguinte forma, nas especificações do objeto, neste caso de um deploymente: **spec.template.metadata.annotations**

```bash
# exemplo:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: xxxx
spec: 
  replicas: 10
  selector:
    matchLabels:
      app: xxxx
  template:
    metadata:
      annotations:
        prometheus.io/scrape: 'true' # permitindo que o prometheus capture a metrica
        prometheus.io/port: '5000' # a porta do endpoint que esta sendo exposto
        prometheus.io/path: '/metrics'

# execute o manifesto para aplicar as configurações
```

- Depois de aplicar o arquivo manifesto, vá em **Targets** e verifique se o prometheus está coletando as metricas
- Note que vai ter um nome **kubernetes-pods (10/10 up)** e clique em **show more** para ver as metricas
- Verifique a porta que o service da aplicaçação está rodando, com o comando: 

```bash
kubectl get svc
NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
fakeshop                              LoadBalancer   10.96.123.143    <pending>     80:30447/TCP   6m30s

#entre na porta com o ip-minikube: http://192.168.49.2:30447/metrics
```

- Depois de ter entrado na url da aplicação/metrics. Pegue o nome da métrica **flask_http_request_duration_seconds_sum** e coloque em graph para ver as metricas coletadas.
- Documentação sobre as query do prometheus: https://prometheus.io/docs/prometheus/latest/querying/basics/
- Agora, com as métricas sendo coletada da aplicação, é interessante criar uma dashboard de gráficos com o Grafana.

# Dashboard - Grafana
Com o Prometheus já coletando as métricas, certifique-se que o service do grafana está rodando e entre na porta que ele está.

```bash
kubectl get svc

# Saída esperada
NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
grafana                               LoadBalancer   10.104.162.74    <pending>     80:31924/TCP   110m

# No meu caso, vou entrar na porta 31924

``` 

- Para fazer o login, o usuário é **admin** a senha, execute o comando abaixo para pegar a senha que está em uma secret:

```bash
kubectl get secret -n default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

# saída esperada da senha:
maaMPA1FtfPZfE0KwrftEHV025Wnuv7WyYdm2R4t

# cole na parte de senha do grafana
```

* O grafana, é uma ferramenta de dashboard e suporta diversas conexões de dados. No canto esquero do grafana, clique em **Connections** e depois em **Data sources**
- Clique em **Add data source** e note a diversidade de conexões que ele possui

### Aplicando o Prometheus como Data Source no grafana
- Clique em Prometheus
- Defina um Nome (opcional)
- Em **Connection**, precisa colocar o endpoint, e o endpoint é o service do prometheus e a porta que esta sendo usado. Apenas execute o comando abaixo para verificar o service do prometheus e adicionar em **Connection**
```bash
kubectl get svc

# saída esperada:

NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
fakeshop                              LoadBalancer   10.96.123.143    <pending>     80:30447/TCP   44m
grafana                               LoadBalancer   10.104.162.74    <pending>     80:31924/TCP   125m
kubernetes                            ClusterIP      10.96.0.1        <none>        443/TCP        164m
postgre                               ClusterIP      10.101.132.137   <none>        5432/TCP       44m
prometheus-kube-state-metrics         ClusterIP      10.103.111.151   <none>        8080/TCP       125m
prometheus-prometheus-node-exporter   ClusterIP      10.111.236.174   <none>        9100/TCP       125m
prometheus-server                     LoadBalancer   10.103.92.246    <pending>     80:31868/TCP   125m

# Adicione o nome do serviço do prometheus na url em Connection do Grafana, dessa maneira: http://prometheus-server:80
``` 

- Depois de ter colocado a url http://prometheus-server em **Connection**, clique em Save & Test

### Testando se realmente a conexão entre Prometheus e Grafana
- No canto esquerdo, clique em **Explore**
- Observe que vai ter uma consulta de métricas para configurar
- Criando uma configuração de teste com a métrica **flask_http_request_total** ( O nome dessa métrica, peguei em: http://192.168.49.2:30447/metrics)
- Adicione a métrica **flask_http_request_total** em *Metric*
- Em **labels filters**, pode decidir que tipo de label deseja, eu coloquei instance
- E tem os valores, esses valores são basicamente a quantidade de replicas que seu deploy, replicaset, statefulsets tiver. No caso dessa aplicação, vai ter 10 valores, ou seja, 10 replicas.
- Selecione um valor, e clique em **Run Query** no canto superior direito
- Note que o Prometheus vai trazer as métricas

### É possível filtrar por vários modelos de consulta
- Clique na opção que tem um [ + ] do lado do valor inserido
- Pode escolher por onde deseja filtrar as labels, se é status, pod, namespace, node e etc. Cada label tem um valor diferente.

## Montando uma dashboard
- No menu lateral do Grana, clique em dashboard
- Clique em Create Dashboard
- Adicione uma visualização
- Escolha o Prometheus ou algum outro de sua preferencia

Configure a dashboard de acordo com a sua preferencia.
- Adicione uma Query com o nome da métrica, escolha uma label e clique em **Run queries**
- Com a query montada, no canto superior **direito** clique em **Save Dashboard**
- Defina um nome para a dashboard e clique em save.
- Verifique Relmente se salvou a dashboard, clique em Dashboard novamente na lateral esquerda, clique no nome definido. Se ocorreu tudo certo, vai ver a dashboard salva
- No canto superior direito, consegue definir o tempo de atualização das metricas


## Costumização da dashboard
Note que a dashboard possui uma legenda não muito agradável, é possível costumizar de uma forma mais elaborada.

- Na **dashboards**, clique nos 3 pontinhos no canto superior direito, e clique em **Edit**
- No campo da **query**, clique em **options**
- Clique m **Legend**, depois em **Custom**
- Adicione o nome de uma label que deseja ver, exemplo:

```bash
{{instance}} #aperte enter
```

- Se quiser ver mais de uma label, adicione mais uma label ao lado de **{{instance}}** com um **-**. Exemplo:

```bash
{{instance}} - {{status}} #aperte enter
```
- É possível somar uma métrica com uma ou várias labels específicas. Em Metrics, clique em **Code** que está próximo a **Run queries** e adicione

```bash
sum(nome-da-metrica) by (nome-da-label)

sum(flask_http_request_total) by (status) # Clique em Run queries
```  

- no campo ao lado, **Min step**



## Importando uma Dashboard do Grafana
- entre no link: https://grafana.com/grafana/dashboards/ e escolha a dashboard de sua preferencia
- quando escolher uma dashboard, clique em **Copy ID to clipboard** na laterado direito do site **Grafana Labs**
- Volte ao Grafana, clique na lateral esquerdo em Dashboard e clique em **New** depois em **import**
- Cole o ID que copiou anteriormente no campo **Grafana.com dashboard URL or ID** e depois clique em **Load**
- Escolha o Prometheus no campo **Select a Prometheus data Source**
- clique em **import**












