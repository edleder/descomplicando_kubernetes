Inicio da aula do Day-4
O que iremos ver hoje?

Hoje é dia de falar sobre dois objetos muito importantes no Kubernetes, os ReplicaSets e os DaemonSets.

Nós já sabemos o que é um Deployment e também já sabemos o que é um Pod no detalhe, então agora vamos conhecer essas duas figuras que estão super conectadas com o Deployment e com o Pod. Quando falamos sobre Deployment é impossível não falar sobre ReplicaSet, pois o Deployment é um objeto que cria um ReplicaSet e o ReplicaSet é um objeto que cria um Pod, veja que tudo está conectado.

Já o nosso querido DaemonSet é um objeto que cria um Pod e esse Pod é um objeto que fica rodando em todos os nodes do cluster, super importante para nós, pois é com DaemonSet que nós conseguimos garantir que teremos pelo menos um Pod rodando em cada node do cluster. Por exemplo, imagine que você precisa de instalar os agente do Datadog ou ainda um exporter do Prometheus em todos os nodes do cluster, para isso você precisa de um DaemonSet.

Ainda no dia de hoje, nós iremos aprender como garantir que os nossos Pods estão rodando corretamente, através das Probes do Kubernetes.

Nós vamos falar sobre Readiness Probe, Liveness Probe e Startup Probe, e claro, mostrando todos os detalhes em exemplos práticos e super explicativos.

Hoje é o dia de você aprender sobre esses dois objetos que são super importantes, e ainda, garantir que nós nunca colocaremos os nossos Pods em produção sem antes garantir que eles estão rodando corretamente e sendo checados pelas Probes do Kubernetes.

Bora lá! #VAIIII

Exemplo com todas as probes

Vamos para o nosso exemplo final de hoje, vamos utilizar todas as probes que vimos até aqui, e vamos criar um arquivo chamado nginx-todas-probes.yaml:

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
        livenessProbe: # Onde definimos a nossa probe de vida
          exec: # O tipo exec é utilizado quando queremos executar algo dentro do container.
            command: # Onde iremos definir qual comando iremos executar
              - curl
              - -f
              - http://localhost:80/
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 1 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
        readinessProbe: # Onde definimos a nossa probe de prontidão
          httpGet: # O tipo de teste que iremos executar, neste caso, iremos executar um teste HTTP
            path: / # O caminho que iremos testar
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 1 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
        startupProbe: # Onde definimos a nossa probe de inicialização
          tcpSocket: # O tipo de teste que iremos executar, neste caso, iremos executar um teste TCP
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 1 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
 

Pronto, estamos utilizando as três probes, vamos aplicar:

kubectl apply -f nginx-todas-probes.yaml
 

E vamos ver se os nossos Pods estão saudáveis:

 

Vamos ver na saída do describe pods se as nossa probes estão por lá.

...
    Liveness:     exec [curl -f http://localhost:80/] delay=10s timeout=5s period=10s #success=1 #failure=3
    Readiness:    http-get http://:80/ delay=10s timeout=5s period=10s #success=1 #failure=3
    Startup:      tcp-socket :80 delay=10s timeout=5s period=10s #success=1 #failure=3
 

Todas lá! Maravilha!

Agora podemos dizer que já sabemos como cuidar bem dos nossos Pods e deixá-los sempre saudáveis e no controle.

Não esqueça de acessar a documentação oficial do Kubernetes para saber mais sobre as probes, e claro, se tiver alguma dúvida, não deixe de perguntar.