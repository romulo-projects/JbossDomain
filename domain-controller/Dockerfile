<<<<<<< HEAD
FROM quay.io/wildfly/wildfly:latest

USER root

# Criação do diretório do módulo GELF no caminho correto
RUN mkdir -p /opt/jboss/wildfly/modules/system/layers/base/biz/paluch/logging/main

# Copia os JARs necessários para o módulo
COPY logstash-gelf-1.15.1.jar \
     /opt/jboss/wildfly/modules/system/layers/base/biz/paluch/logging/main/

COPY jedis-2.9.0.jar \
     /opt/jboss/wildfly/modules/system/layers/base/biz/paluch/logging/main/

COPY commons-pool2-2.4.3.jar \
     /opt/jboss/wildfly/modules/system/layers/base/biz/paluch/logging/main/

# Copia o module.xml que define o módulo
COPY module.xml \
     /opt/jboss/wildfly/modules/system/layers/base/biz/paluch/logging/main/

# Copia o domain.xml com a configuração de logging ajustada
COPY domain.xml \
    /opt/jboss/wildfly/domain/configuration/domain.xml

# Script auxiliar de espera
COPY wait-for-it.sh /opt/wait-for-it.sh

# Criação de usuários administrativos e de domínio
RUN /opt/jboss/wildfly/bin/add-user.sh admin Admin#123 --silent
RUN /opt/jboss/wildfly/bin/add-user.sh jboss Jboss@123 --silent

# Permissões para o script
RUN chmod +x /opt/wait-for-it.sh

USER jboss
=======
FROM quay.io/wildfly/wildfly:latest

USER root

# Admin do console (opcional)
RUN /opt/jboss/wildfly/bin/add-user.sh admin Admin#123 --silent

# User usado pelo Host Controller para conectar no Domain Controller
RUN /opt/jboss/wildfly/bin/add-user.sh jboss Jboss@123 --silent

USER jboss
>>>>>>> 8b5d76762fa12e297e1badeb24e5b0c522120d2f
