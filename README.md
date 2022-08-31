# Como criar seu primeiro Cluster Kubernetes na prática com KinD - Kubernetes para Iniciantes

Projeto construido durante as aulas disponibilizadas no canal do Erick Wendel, agradecimentos no final. Este repositorio será dividido em branchs de acordo com as aulas passadas.

- Primeira aula - criando seu primeiro cluster com kind

```
git checkout first-cluster
```

- Segunda aula - Como publicar sua primeira aplicação Node.JS no Kebernetes

```
git checkout node-with-kubernetes
```

# Primeira aula

## Comandos basicos

Criando um kubernate com o kind

```
kind create cluster --name demo-cluster
```

caso não seja passado o nome ( --name) para o cluster, por padrão será criado com o nome kind

Para ver as informações do nosso cluster

```
kubectl cluster-info --context kind-demo-cluster
```

Para excluir o cluster

```
kind delete cluster --name demo-cluster
```

## Criando nosso cluster personalizado

Muitas vezes precisamos criar um aplicação que seja bem ditribuida, então vamos criar um cluster de nós do kind, para isso vamos criar um arquivo de configuração para o kind.

```
vim kind-3-nodes.yml
```

Após isso sera criado um arquivo com o nome 'kind-3-nodes.yml', neste arquivo vamos definir nossas configurações para os nós, dentro dele vamos colocar as seguintes configurações

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
    - role: control-plane
    - role: worker
    - role: worker
    - role: worker
```

Devemos ficar atentos a essa versão pois ela pode mudar a qualquer momento no decorrer do desenvolvimento do projeto
Acima definimos o nó de plano de controle e a quantidade de nós que desejamos para a aplicação. Em seguida podemos salvar o arquivo e rodar o comando:

```
kind create cluster --name demo-cluster-3 --config ./kind-3-nodes.yml
```

Se digitarmos o comando a baixo teremos o retorno dos nós que criamos atraves do arquivo de configuração.

```
kubectl get nodes
```

retorno esperado:

```
NAME                 STATUS    ROLES              AGE     VERSION
kind-control-plane   Ready     control-plane     4m8s     v1.24.0
kind-worker          Ready     <none>            3m45s    v1.24.0
kind-worker2         Ready     <none>            3m45s    v1.24.0
kind-worker3         Ready     <none>            3m47s    v1.24.0

```

Com o Cluster criado agora precisamos subir uma imagem para ele, do contrario de nada serve tudo isso...

Primeiramente precisamos fazer o build de nossa imagem que está no diretorio node-image

```
docker build -t hello-api .
```

Feito isso temos a nossa imagem criada, agora podemos carregar essa imagem para dentro do cluster do kind.

```
kind load docker-image hello-api --name demo-cluster-3
```

Agora que temos a imagem enviada ao nosso cluster, podemos baixar a imagem diretamente de dentro do docker, sem precisar de um servidor externo.

## Configurando o deploy

Para isso vamos criar um arquivo deploy.yaml, com ajuda da extensão do kubernates vamos declarar algumas configurações. Quando digitamos deploy no VsCode a extensão nos da todas as configurações basicas para o arquivo.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: <Image>
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: <Port>
```

Dentro dele vamos alterar onde estiver myapp para api apenas, e dentro de containers > image vamos colocar o nome da nossa imagem (hello-api).
Vamos acrescentar também, dentro de containers a configuração de imagePullPolicy, logo a baixo de image.

```
imagePullPolicy: IfNotPresent
```

Por padrão nossa porta será sempre a 3000, então podemos substituir no final do arquivo onde consta Port

Logo após vamos criar um serviço do Kubernetes. Com o comando service da extensão.

```
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: <Port>
    targetPort: <Target Port>
```

Nele vamos substitui o myapp por api, igual na primeira parte, e a nossa porta de serviço sera a 80 que redirecionara para a 3000, assim:

```
- port: 80
  targetPort: 3000
```

Para aplicarmos as configurações dentro do nosso cluster, rodamos o comando

```
kubectl apply -f ./deploy.yaml
```

Para conferir se esta tudo certo, digitamos o comando a baixo onde podemos ver que temos um pod rodando.

```
kubectl get pods
```

Podemos também ver seus logs

```
kubectl logs deploy/api
```

## Acessando o cluster localmente

Para acessar localmente precisamos configurar um port-forward, temos que entrar dentro do servidor e realizar a configuração.

```
kubectl port-forward service/api 3000:80
```

Acima definimos que o serviço local, service/api, vai ouvir a porta local 3000 acessando a porta do cluster 80. Apos isso podemos ir direto ao nosso navegador e testar http://localhost:3000 e teremos a resposta setada em nossa api.

## Ferramentas

- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/)
- [Docker](https://www.docker.com/)

## Agradecimento

Esse é um breve resumo da aula de Kubernetes que nosso querido [Erick Wendel](https://www.youtube.com/c/ErickWendelTreinamentos) disponibilizou em seu canal no Youtube em parceiria com o mestre [Lucas Santos](https://www.youtube.com/channel/UCki-WnBzwzpvbBDk4swJniQ), em seu blog temos muitas outras informações a repeito de Docker, Kind e kubernetes. Obrigado.
