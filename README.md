# Configuração de Rotas no Nginx e Correção de Erro 404

## 1. Ambiente

- Ubuntu Linux
- Docker
- Nginx
- React
- Vite
- React Router

## 2. Baixar o projeto

```bash
git clone https://github.com/Ma-Eduarda/censo-escolar.git
```

Entrar na pasta:

```bash
cd censo-escolar
```

Instalar dependências:

```bash
npm install
```

Gerar build:

```bash
npm run build
```

## 3. Verificar o Docker

```bash
docker --version
docker ps
```

Caso o usuário precise executar Docker com permissões administrativas, utilizar `sudo` antes dos comandos.

## 4. Executar o Nginx

```bash
docker run -d --name censo-nginx -p 8080:80 nginx
```

Verificar:

```bash
docker ps
docker logs censo-nginx
```

Acessar:

```text
http://localhost:8080
```

## 5. Criar a configuração inicial

Criar os diretórios:

```bash
mkdir -p deploy/nginx
```

Criar o arquivo:

```bash
nano deploy/nginx/default.conf
```

Adicionar:

```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Salvar no Nano: `Ctrl + O`, Enter e `Ctrl + X`.

## 6. Executar a aplicação no Nginx

Remover o contêiner de teste:

```bash
docker stop censo-nginx
docker rm censo-nginx
```

Na raiz do projeto, executar:

```bash
docker run -d \
  --name censo-nginx \
  -p 8080:80 \
  -v "$(pwd)/dist:/usr/share/nginx/html:ro" \
  -v "$(pwd)/deploy/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro" \
  nginx:alpine
```

Acessar:

```text
http://localhost:8080
```

## 7. Reproduzir o erro 404

Acessar uma rota interna existente do projeto.

Testar:
- Navegação interna.
- Atualização da página com F5.
- Acesso direto pela URL.

Consultar os logs:

```bash
docker logs censo-nginx
```

Consultar a configuração:

```bash
docker exec censo-nginx nginx -T
```

Registrar a causa identificada:

**Causa do erro:** [Preencher após investigação.]

## 8. Corrigir o erro 404

Editar:

```bash
nano deploy/nginx/default.conf
```

Substituir a configuração por:

```nginx
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

Validar:

```bash
docker exec censo-nginx nginx -t
```

Recarregar:

```bash
docker exec censo-nginx nginx -s reload
```

## 9. Testar após a correção

| Teste | Resultado |
|---|---|
| Página inicial | [ ] |
| Rota interna | [ ] |
| Atualização com F5 | [ ] |
| Acesso direto pela URL | [ ] |
| Configuração válida | [ ] |

## 10. Consultar logs e encerrar

Visualizar logs:

```bash
docker logs censo-nginx
```

Parar o contêiner:

```bash
docker stop censo-nginx
```

Remover:

```bash
docker rm censo-nginx
```

## 11. Referências

- [Documentação do Nginx](https://nginx.org/en/docs/)
- [Nginx com Docker](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-docker/)
- [Docker - Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [React Router](https://reactrouter.com/)
