# -Criando-um-Pipeline-de-Deploy-de-uma-Aplica-o-Utilizando-Gitlab-Docker-e-Kubernetes
/meu-projeto
├── .gitlab-ci.yml
├── Dockerfile
└── app.py
# Use a imagem oficial do Python como base
FROM python:3.9-slim

# Defina o diretório de trabalho
WORKDIR /app

# Copie os arquivos de requisitos e instale as dependências
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copie o restante do código da aplicação
COPY . .

# Exponha a porta que a aplicação usará
EXPOSE 5000

# Comando para rodar a aplicação
CMD ["python", "app.py"]
stages:
    - build
    - deploy
  
  variables:
    REGISTRY: your-docker-registry
    IMAGE_NAME: $REGISTRY/my-app
    KUBE_NAMESPACE: your-kube-namespace
  
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  
  build:
    stage: build
    script:
      - docker build -t $IMAGE_NAME:$CI_COMMIT_SHORT_SHA .
      - docker push $IMAGE_NAME:$CI_COMMIT_SHORT_SHA
  
  deploy:
    stage: deploy
    script:
      - kubectl config set-context --current --namespace=$KUBE_NAMESPACE
      - kubectl set image deployment/my-app my-app=$IMAGE_NAME:$CI_COMMIT_SHORT_SHA
      - kubectl rollout status deployment/my-app
    only:
      - master
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
