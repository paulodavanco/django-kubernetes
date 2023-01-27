# Local Kubernetes com minikube

Criando ambiente kubernetes de desenvolvimento com minikube.

 Dockerfile

```docker
FROM python:3.10-bullseye
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code/
CMD [ "bash", "entrypoint.sh" ]
```

O comando em bash executa o runserver do Django na porta 8000.

deploy.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: website
spec:
  replicas: 3
  selector:
    matchLabels:
      app: website
  template:
    metadata:
      labels:
        app: website
    spec:
      containers:
      - name: website
        image: paulodavanco/website
        imagePullPolicy: Never
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8000
```

Importante que exista imagePullPolicy com a opção Never para que seja aceita a imagem local.

É possível fazer build da imagem docker direto pelo minikube ou, se preferir, carregar uma imagem já existente.

```bash
minikube image build -t namespace/imagename .
```

```bash
minikube image load namespace/imagename
```

Com a imagem disponível, é possível fazer o deploy.

```bash
kubectl apply -f deploy.yml
```

Para acessar pelo navegador:

```bash
kubectl port-forward deploy/appname 8080:8080
```

### Services

Também é possível expor para a internet através de um serviço.

service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: website-service
spec:
  selector:
    app: website
  ports:
  - port: 8000
    targetPort: 8000
  type: LoadBalancer
```

```bash
kubectl apply -f service.yml
```

Neste momento, porém, nenhum IP externo será atribuído. Para que o IP seja atribuído corretamente, será necessário executar o minikune tunnel:

```bash
minikube tunnel
```

Assim, será possível acessar no navegar através do IP atribuído.
Lembrando que será necessário habilitar o IP no arquivo settings do Django.

Referências:

- [https://levelup.gitconnected.com/two-easy-ways-to-use-local-docker-images-in-minikube-cd4dcb1a5379](https://levelup.gitconnected.com/two-easy-ways-to-use-local-docker-images-in-minikube-cd4dcb1a5379)
- [https://www.youtube.com/watch?v=LqzY9eun6Fk&t=1796s](https://www.youtube.com/watch?v=LqzY9eun6Fk&t=1796s)
- [https://minikube.sigs.k8s.io/docs/handbook/accessing/](https://minikube.sigs.k8s.io/docs/handbook/accessing/)
