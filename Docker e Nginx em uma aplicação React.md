# Tutorial: Docker e Nginx em uma aplicação React

**Projeto:** Censo Escolar\
**Objetivo:** executar a mesma aplicação React em dois cenários: 1. **1
Stage:** Node.js executando a aplicação. 2. **Multi-Stage:** Node.js
construindo a aplicação e Nginx servindo os arquivos gerados.

> Execute os comandos no terminal Ubuntu/WSL integrado ao VS Code. Crie
> e edite os arquivos pelo Explorador do VS Code.

------------------------------------------------------------------------

## 1. Abrir o projeto no VS Code

Abra a pasta do projeto `censo-escolar` no VS Code.

No menu superior, selecione **Terminal → Novo Terminal**. Confira se o
terminal aberto é o Ubuntu/WSL.

Se precisar entrar na pasta do projeto, use o caminho correspondente no
seu computador:

``` bash
cd ~/caminho/para/censo-escolar
```

Confira se está na raiz do projeto:

``` bash
ls
```

Você deve encontrar, entre outros, `package.json` e `package-lock.json`.

## 2. Verificar o Docker

**Onde:** terminal Ubuntu/WSL dentro do VS Code.

``` bash
docker --version
docker info
```

Se `docker info` apresentar informações do servidor Docker, continue. Se
houver erro de conexão com `/var/run/docker.sock`, verifique se o Docker
Desktop está aberto e se a integração com WSL está habilitada.

------------------------------------------------------------------------

# Parte 1 --- Node.js (1 Stage)

## 3. Criar o Dockerfile.node

**Onde:** VS Code → Explorador de arquivos.

Na raiz do projeto, crie um arquivo chamado `Dockerfile.node` e coloque:

``` dockerfile
FROM node:26-alpine

WORKDIR /app

COPY package*.json .

RUN npm ci

COPY . .

EXPOSE 5000

CMD ["npm", "start", "--", "--host", "0.0.0.0"]
```

Salve o arquivo.

> O parâmetro `--host 0.0.0.0` permite que o servidor Vite receba
> conexões de fora do contêiner. Este exemplo pressupõe que o script
> `start` do `package.json` inicia o Vite na porta 5000. Se o seu script
> usar outra porta ou comando, ajuste o Dockerfile de acordo.

## 4. Construir a imagem (Build)

**Onde:** terminal Ubuntu/WSL dentro do VS Code.

``` bash
docker build -t censo-escolar:1.0.0-node -f Dockerfile.node .
```

Confira se a imagem foi criada:

``` bash
docker images
```

Procure por `censo-escolar` com a tag `1.0.0-node`.

**O que significa o comando:** - `docker build`: constrói uma imagem. -
`-t`: define o nome e a tag da imagem. - `censo-escolar:1.0.0-node`:
nome e versão escolhidos para a imagem. - `-f Dockerfile.node`: informa
qual Dockerfile será utilizado. - `.`: usa a pasta atual como contexto
de construção.

## 5. Criar e executar o contêiner (Run)

**Onde:** terminal Ubuntu/WSL dentro do VS Code.

``` bash
docker run -d --name censo-node -p 5000:5000 censo-escolar:1.0.0-node
```

Confira se está em execução:

``` bash
docker ps
```

Abra no navegador:

<http://localhost:5000>

**Resultado esperado:** a aplicação React deve abrir no navegador.

------------------------------------------------------------------------

# Parte 2 --- Node.js + Nginx (Multi-Stage)

## 6. Criar o Dockerfile.nginx

**Onde:** VS Code → Explorador de arquivos.

Na raiz do projeto, crie `Dockerfile.nginx`:

``` dockerfile
# Primeiro estágio: construir a aplicação
FROM node:26-alpine AS build

WORKDIR /app

COPY package*.json .

RUN npm ci

COPY . .

RUN npm run build -- --base=/

# Segundo estágio: servir os arquivos com Nginx
FROM nginx:1.31.8-alpine AS production

COPY --from=build /app/dist /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Salve o arquivo.

O primeiro estágio usa Node.js para instalar dependências e gerar a
pasta `dist`. O segundo estágio usa Nginx para servir os arquivos dessa
pasta.

## 7. Criar o nginx.conf

**Onde:** VS Code → Explorador de arquivos.

Na raiz do projeto, crie `nginx.conf`:

``` nginx
server {
    listen 80;

    server_name localhost;

    root /usr/share/nginx/html;

    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Essa configuração encaminha rotas da aplicação React para `index.html`,
evitando erros 404 ao atualizar páginas de rotas internas do React
Router.

## 8. Criar o .dockerignore

**Onde:** VS Code → Explorador de arquivos.

Na raiz do projeto, crie `.dockerignore`:

``` text
node_modules
dist
.git
.gitignore
```

Esse arquivo evita enviar essas pastas e arquivos no contexto de
construção do Docker.

## 9. Construir a imagem Nginx (Build)

**Onde:** terminal Ubuntu/WSL dentro do VS Code.

``` bash
docker build -t censo-escolar:1.0.0-nginx -f Dockerfile.nginx .
```

Confira as imagens:

``` bash
docker images
```

Devem aparecer as imagens `censo-escolar:1.0.0-node` e
`censo-escolar:1.0.0-nginx`.

## 10. Criar e executar o contêiner Nginx (Run)

**Onde:** terminal Ubuntu/WSL dentro do VS Code.

``` bash
docker run -d --name censo-nginx -p 8080:80 censo-escolar:1.0.0-nginx
```

Confira:

``` bash
docker ps
```

Abra no navegador:

<http://localhost:8080>

**Resultado esperado:** a aplicação React deve abrir sendo servida pelo
Nginx.

------------------------------------------------------------------------

## 11. Comandos úteis

**Ver contêineres em execução:**

``` bash
docker ps
```

**Ver todas as imagens:**

``` bash
docker images
```

**Ver logs:**

``` bash
docker logs censo-node
docker logs censo-nginx
```

**Testar a configuração do Nginx:**

``` bash
docker exec censo-nginx nginx -t
```

**Parar os contêineres:**

``` bash
docker stop censo-node censo-nginx
```

**Iniciar novamente:**

``` bash
docker start censo-node censo-nginx
```

**Remover os contêineres:**

``` bash
docker rm -f censo-node censo-nginx
```

> Para executar novamente um contêiner com o mesmo nome depois de
> removê-lo, use o comando `docker run` correspondente.

------------------------------------------------------------------------

## 12. Enviar os arquivos para o GitHub

**Onde:** terminal Ubuntu/WSL dentro do VS Code.

Confira os arquivos alterados:

``` bash
git status
```

Adicione, faça o commit e envie:

``` bash
git add .
git commit -m "Adiciona Docker e Nginx"
git push origin main
```

Se a branch principal do seu repositório não for `main`, substitua pelo
nome correto da branch.

------------------------------------------------------------------------

## 13. Resultado esperado

  -----------------------------------------------------------------------------------------
  Cenário           Imagem                        Contêiner         Endereço
  ----------------- ----------------------------- ----------------- -----------------------
  Node.js (1 Stage) `censo-escolar:1.0.0-node`    `censo-node`      http://localhost:5000

  Node.js + Nginx   `censo-escolar:1.0.0-nginx`   `censo-nginx`     http://localhost:8080
  (Multi-Stage)                                                     
  -----------------------------------------------------------------------------------------

## 14. Entendendo a atividade

-   **Dockerfile:** arquivo com instruções para construir uma imagem.
-   **Build (`docker build`):** constrói a imagem a partir do
    Dockerfile.
-   **Run (`docker run`):** cria e inicia um contêiner usando uma
    imagem.
-   **1 Stage:** neste exercício, Node.js executa a aplicação React.
-   **Multi-Stage:** Node.js constrói a aplicação e Nginx serve os
    arquivos gerados.
-   **`dist`:** pasta produzida pelo comando `npm run build`.

**Fluxo do primeiro cenário:**

`React → Node.js → Dockerfile.node → Build → Run → Contêiner`

**Fluxo do segundo cenário:**

`React → Node.js (build) → Nginx → Dockerfile.nginx → Build → Run → Contêiner`

------------------------------------------------------------------------
docker exec censo-nginx ls -la /usr/share/nginx/html/assets
