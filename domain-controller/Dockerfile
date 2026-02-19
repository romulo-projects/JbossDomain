FROM quay.io/wildfly/wildfly:latest

USER root

# Admin do console (opcional)
RUN /opt/jboss/wildfly/bin/add-user.sh admin Admin#123 --silent

# User usado pelo Host Controller para conectar no Domain Controller
RUN /opt/jboss/wildfly/bin/add-user.sh jboss Jboss@123 --silent

USER jboss
