# WildFly / JBoss Domain Mode Lab

## Objetivo

Este laboratório tem como objetivo **aprimorar o conhecimento em WildFly/JBoss Domain Mode**, incluindo:

- Configuração de **Domain Controller** e **hosts gerenciados**
- Deploy de aplicações (`WAR`) em múltiplos hosts
- Testes de **balanceamento interno**
- Observabilidade, failover e troubleshooting
- Experimentos com **socket bindings**, profiles e context paths

> Nota: Este lab é voltado para aprendizado. O NGINX serve apenas para testes de balanceamento externo.

---
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
│   │   ├── domain.xml          <-- Domain Controller config
│   │   └── host-master.xml     <-- Host Controller principal
│   ├── domain/
│   │   └── configuration/
│   │       ├── host-master.xml <-- Host Controller runtime copy
│   │       ├── mgmt-groups.properties
│   │       └── mgmt-users.properties
│   └── logs/
├── host1/
│   ├── config/
│   │   └── host1.xml            <-- Slave host1
│   └── logs/
├── host2/
│   ├── config/
│   │   └── host2.xml            <-- Slave host2
│   └── logs/
├── nginx/
│   └── nginx.conf
└── docker-compose.yml
```



---

## Configuração
```
Domain Controller
└─ host1
   └─ server-one (main-server-group)
   └─ host2
   └─ server-two (main-server-group)
```
### Domain Controller

- Porta de management: `9990`  
- Porta de domínio: `9999`  
- Host master: `host-master.xml`  
- Socket bindings e profiles configurados em `domain.xml`

### Hosts Gerenciados

| Host  | Socket Binding Offset | HTTP Port |
|-------|--------------------|-----------|
| host1 | 0                  | 8080      |
| host2 | 150                | 8230      |

- Configurações principais nos arquivos:
  - `host1/config/host1.xml`
  - `host2/config/host2.xml`

---



## Deploy de Aplicações

- WAR de exemplo: `domain-controller/app/app.war`  
- Deploy via CLI do Domain Controller:

```bash
# Deploy em todos os hosts do grupo
/host=*/server=*/:deploy(path=domain-controller/app/app.war, enabled=true)


Teste de aplicação:

curl http://host1:8080/app/
curl http://host2:8230/app/


Undeploy:

undeploy app.war --server-groups=main-server-group

Testes e Observabilidade

Verificar qual host respondeu:

curl -I http://localhost/app/  # X-Backend pode mostrar host


Monitorar estado de hosts e deployments via CLI:

# Status do server
/host=*/server=*/:read-attribute(name=status)

# Recursos de deployment
/host=*/server=*/deployment=app.war:read-resource(include-runtime=true)

# Boot errors
/host=host1/server=server-one:read-boot-errors()


Parar e iniciar hosts:

/host=host1/server=server-one:stop
/host=host1/server=server-one:start

Undertow e Context Paths

WAR /app → http://host1:8080/app/

WAR raiz / → http://host1:8080/ (WildFly welcome page)

Experimentos sugeridos:

Deploy de múltiplos WARs em hosts diferentes

Ajustar context path e observar comportamento do console e links internos

Testar failover e balanceamento de hosts

Proxy Reverso (NGINX)

O NGINX é opcional, serve apenas para balancear requests HTTP externos.

Configuração simplificada:

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

Pontos de Aprendizado

Entender Domain Mode e como o master gerencia múltiplos hosts

Deploy via server-groups vs deploy por host

Uso de socket-binding-port-offset e interfaces

Observabilidade via CLI e logs de boot/runtime

Testar failover e comportamento de múltiplos deployments

Referências

WildFly Documentation

WildFly CLI Guide

Domain Mode Overview
