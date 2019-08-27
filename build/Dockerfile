FROM quay.io/operator-framework/ansible-operator:v0.9.0

USER 0 
ARG OCP_VERSION=4.1.9
RUN yum install -y tar gcc wget && yum clean all \
    && wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCP_VERSION}/openshift-client-linux-${OCP_VERSION}.tar.gz && tar xzvf openshift-client-linux-${OCP_VERSION}.tar.gz -C /usr/bin && rm -f openshift-client-linux-${OCP_VERSION}.tar.gz /usr/bin/README.md && chmod +x /usr/bin/oc && chmod +x /usr/bin/kubectl

USER 1001

COPY watches.yaml ${HOME}/watches.yaml

COPY roles/ ${HOME}/roles/
COPY playbook.yml ${HOME}/playbook.yml

