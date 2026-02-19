# WildFly / JBoss Domain Mode Lab

## Objetivo

Este laboratório tem como objetivo **aprimorar o conhecimento em WildFly/JBoss Domain Mode**, incluindo:

* Configuração de Domain Controller e hosts gerenciados
* Deploy de aplicações (`WAR`) em múltiplos hosts
* Testes de balanceamento interno
* Observabilidade, failover e troubleshooting
* Experimentos com socket bindings, profiles e context paths

> Nota: Este lab é voltado para aprendizado. O NGINX serve apenas para testes de balanceamento externo.

---

## Clone do Projeto

```bash
git clone https://github.com/blcsilva/JbossDomain.git
cd JbossDomain
```

---

## Subindo o Ambiente

O ambiente é totalmente containerizado via Docker Compose.

### Pré-requisitos

* Docker instalado
* Docker Compose instalado

### Subir o laboratório

```bash
docker-compose up -d
```

Acompanhar logs:

```bash
docker-compose logs -f
```

Parar o ambiente:

```bash
docker-compose down
```

---

## Credenciais de Acesso

### Domain Controller (Console Web)

* URL: [http://localhost:9990](http://localhost:9990)
* Usuário: `admin`
* Senha: `Admin#123`

### Host Controller

* Usuário: `jboss`
* Senha: `Jboss@123`

---

## Estrutura do Projeto

```
JbossDomain/
├── domain-controller/
│   ├── app/
│   │   ├── hello-world-war-1.0.0/
│   │   │   ├── META-INF/
│   │   │   ├── WEB-INF/
│   │   │   └── index.jsp
│   │   ├── maven-archiver/
│   │   │   └── pom.properties
│   │   └── app.war
│   ├── config/
│   │   ├── domain.xml
│   │   └── host-master.xml
│   ├── domain/
│   │   └── configuration/
│   │       ├── host-master.xml
│   │       ├── mgmt-groups.properties
│   │       └── mgmt-users.properties
│   └── logs/
├── host1/
│   ├── config/
│   │   └── host1.xml
│   └── logs/
├── host2/
│   ├── config/
│   │   └── host2.xml
│   └── logs/
├── nginx/
│   └── nginx.conf
└── docker-compose.yml
```

---

## Arquitetura do Domain

```
Domain Controller
├─ host1
│  └─ server-one (main-server-group)
└─ host2
   └─ server-two (main-server-group)
```

### Domain Controller

* Porta de management: `9990`
* Porta de domínio: `9999`
* Host master: `host-master.xml`
* Socket bindings e profiles configurados em `domain.xml`

### Hosts Gerenciados

| Host  | Socket Binding Offset | HTTP Port |
| ----- | --------------------- | --------- |
| host1 | 0                     | 8080      |
| host2 | 150                   | 8230      |

Arquivos principais:

* `host1/config/host1.xml`
* `host2/config/host2.xml`

---

## Deploy de Aplicações

WAR de exemplo:

```
domain-controller/app/app.war
```

Deploy via CLI do Domain Controller:

```bash
/host=*/server=*/:deploy(path=domain-controller/app/app.war, enabled=true)
```

### Teste da aplicação

```bash
curl http://localhost:8080/app/
curl http://localhost:8230/app/
```

### Undeploy

```bash
undeploy app.war --server-groups=main-server-group
```

---

## Testes e Observabilidade

Verificar qual host respondeu:

```bash
curl -I http://localhost/app/
```

Monitoramento via CLI:

```bash
/host=*/server=*/:read-attribute(name=status)

/host=*/server=*/deployment=app.war:read-resource(include-runtime=true)

/host=host1/server=server-one:read-boot-errors()
```

Parar e iniciar servidores:

```bash
/host=host1/server=server-one:stop
/host=host1/server=server-one:start
```

---

## Undertow e Context Paths

* WAR `/app` → [http://localhost:8080/app/](http://localhost:8080/app/)
* WAR raiz `/` → [http://localhost:8080/](http://localhost:8080/)

### Experimentos sugeridos

* Deploy de múltiplos WARs em hosts diferentes
* Ajustar context path e observar comportamento do console
* Testar failover
* Alterar `socket-binding-port-offset`

---

## Proxy Reverso (NGINX)

Uso opcional apenas para balanceamento externo.

```nginx
upstream wildfly_cluster {
    server host1:8080;
    server host2:8230;
}

server {
    listen 80;

    location = /app { return 301 /app/; }
    location /app/ {
        proxy_pass http://wildfly_cluster/app/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Backend $upstream_addr;
    }
}
```

---

## Pontos de Aprendizado

* Entender Domain Mode e gerenciamento centralizado
* Diferença entre deploy por server-group e por host
* Uso de `socket-binding-port-offset`
* Observabilidade via CLI
* Testes de failover
* Troubleshooting de boot e runtime

---

## Referências

* WildFly Documentation
* WildFly CLI Guide
* Domain Mode Overview
