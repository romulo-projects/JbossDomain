# WildFly / JBoss Domain Mode Lab

## Objetivo

Este laboratório tem como objetivo **aprimorar o conhecimento em WildFly/JBoss Domain Mode**, incluindo:

- Configuração de Domain Controller e hosts gerenciados
- Deploy de aplicações (`WAR`) em múltiplos hosts
- Testes de balanceamento interno e externo (NGINX)
- Observabilidade, failover e troubleshooting
- Experimentos com socket bindings, profiles e context paths

> Nota: Este lab é voltado para aprendizado. **Não use essas configurações/credenciais em produção.**

---

## Clone do Projeto

```bash
git clone https://github.com/blcsilva/JbossDomain.git
cd JbossDomain
```

---

## Subindo o Ambiente (Docker Compose)

### Pré-requisitos

- Docker instalado
- Docker Compose instalado

### Observação importante (WSL/Windows/OneDrive)

Se você estiver rodando o projeto a partir de `/mnt/c/...` (ex: OneDrive no Windows), **bind mounts** para bancos (MongoDB/OpenSearch) podem falhar por permissão/locks.
Por isso, o compose deste lab usa **volumes nomeados** do Docker para persistência do MongoDB/OpenSearch.

### Subir o laboratório

1) (Recomendado) Limpar ambiente anterior:

```bash
docker compose down -v --remove-orphans
```

2) Build das imagens:

```bash
docker compose build --no-cache
```

3) Subir o ambiente:

```bash
docker compose up -d
```

4) Acompanhar logs:

```bash
docker compose logs -f
```

Parar o ambiente:

```bash
docker compose down
```

---

## Endpoints / Portas

### Domain Controller (Console Web)

- URL: http://localhost:9990
- Usuário: `admin`
- Senha: `Admin#123`

### Porta do Domain Controller (Host Controllers)

- Porta (native mgmt): `9999` (usada por `host1` e `host2` para se conectarem ao controller)

### Aplicação de exemplo

- Host1: http://localhost:8080/app/
- Host2: http://localhost:8230/app/
- NGINX (balanceador): http://localhost/app/

### Graylog (UI)

- URL: http://localhost:9000

> Observação: o primeiro start do Graylog/OpenSearch/MongoDB pode demorar.

---

## Estrutura do Projeto

```text
JbossDomain/
├── domain-controller/
│   ├── Dockerfile
│   ├── wait-for-it.sh
│   ├── domain.xml
│   ├── module.xml
│   ├── *.jar
│   ├── app/
│   ├── config/
│   │   └── host-master.xml
│   └── domain/
│       └── configuration/
│           ├── mgmt-groups.properties
│           └── mgmt-users.properties
├── host1/
│   └── config/
│       └── host1.xml
├── host2/
│   └── config/
│       └── host2.xml
├── nginx/
│   └── nginx.conf
└── docker-compose.yml
```

> Persistência do MongoDB e OpenSearch é feita via volumes nomeados do Docker (veja `docker volume ls`).

---

## Arquitetura do Domain

```text
Domain Controller
├─ host1
│  └─ server-one (main-server-group)
└─ host2
   └─ server-two (main-server-group)
```

### Hosts Gerenciados

| Host  | Socket Binding Offset | HTTP Port |
| ----- | --------------------- | --------- |
| host1 | 0                     | 8080      |
| host2 | 150                   | 8230      |

Arquivos principais:

- `host1/config/host1.xml`
- `host2/config/host2.xml`
- `domain-controller/config/host-master.xml`
- `domain-controller/domain.xml`

---

## Deploy de Aplicações

WAR de exemplo:

```text
domain-controller/app/app.war
```

### Deploy via CLI do Domain Controller

Exemplo (ajuste o comando conforme sua necessidade de server-group/host):

```bash
/host=*/server=*/:deploy(path=domain-controller/app/app.war, enabled=true)
```

### Teste da aplicação

```bash
curl http://localhost:8080/app/
curl http://localhost:8230/app/
curl -I http://localhost/app/
```

### Undeploy

```bash
undeploy app.war --server-groups=main-server-group
```

---

## Troubleshooting

### Validar o compose

```bash
docker compose config
```

### Ver status dos containers

```bash
docker compose ps
```

### Logs principais

```bash
docker compose logs -f domain-controller host1 host2
docker compose logs -f graylog opensearch mongodb
```

### Problemas comuns

- **Domain Controller não sobe**: verifique se o `docker-compose.yml` **não** está esperando `domain-controller:9999` dentro do próprio serviço `domain-controller` (deadlock).
- **Build falha no Dockerfile**: verifique se `domain-controller/Dockerfile` não contém marcadores de conflito (`<<<<<<<`, `=======`, `>>>>>>>`).
- **Login no console falha**: pode haver divergência entre usuários criados no build (`add-user.sh`) e os arquivos montados via volume (`mgmt-users.properties`).
- **MongoDB cai com WiredTiger/permission**: rode:
  ```bash
  docker compose down -v --remove-orphans
  docker compose up -d
  ```

---

## Referências

- WildFly Documentation
- WildFly CLI Guide
- Domain Mode Overview