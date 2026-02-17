FROM quay.io/wildfly/wildfly:latest

USER root

# Cria admin para console e CLI
RUN /opt/jboss/wildfly/bin/add-user.sh admin Admin#123 --silent

USER jboss
