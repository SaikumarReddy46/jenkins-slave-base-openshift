##############################################
# Stage 1 : Build go-init
##############################################
#FROM registry.ci.openshift.org/ocp/builder:rhel-8-golang-1.18-openshift-4.11 AS go-init-builder
FROM registry.ci.openshift.org/openshift/release:golang-1.17 AS go-init-builder
WORKDIR  /go/src/github.com/openshift/jenkins
COPY . .
WORKDIR  /go/src/github.com/openshift/jenkins/go-init
RUN GO111MODULE=off go build . && cp go-init /usr/bin

##############################################
# Stage 2 : Build slave-base with go-init
##############################################
#FROM registry.ci.openshift.org/ocp/4.11:cli
FROM registry.ci.openshift.org/origin/4.10:base
MAINTAINER OpenShift Developer Services <openshift-dev-services+jenkins@redhat.com>
COPY --from=go-init-builder /usr/bin/go-init /usr/bin/go-init

# Labels consumed by Red Hat build service
LABEL com.redhat.component="jenkins-slave-base-rhel7-container" \
      name="openshift4/jenkins-slave-base-rhel7" \
      architecture="x86_64" \
      io.k8s.display-name="Jenkins Slave Base" \
      io.k8s.description="The jenkins slave base image is intended to be built on top of, to add your own tools that your jenkins job needs. The slave base image includes all the jenkins logic to operate as a slave, so users just have to yum install any additional packages their specific jenkins job will need" \
      io.openshift.tags="openshift,jenkins,slave" \
      maintainer="openshift-dev-services+jenkins@redhat.com"


ENV HOME=/home/jenkins \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

USER root
# Install headless Java
#RUN localectl set-locale LANG=en_US.UTF-8
RUN yum install langpacks-en glibc-all-langpacks -y --setopt=tsflags=nodocs --disableplugin=subscription-manager
RUN yum install -y --setopt=tsflags=nodocs --disableplugin=subscription-manager java-1.8.0-openjdk-headless
RUN yum -y update
RUN INSTALL_PKGS="glibc-langpack-en bc gettext git java-11-openjdk-headless java-1.8.0-openjdk-headless lsof rsync tar unzip which zip bzip2 jq" && \
    yum install -y --setopt=tsflags=nodocs --disableplugin=subscription-manager $INSTALL_PKGS && \
    yum install -y mailx && \
    yum install -y sendmail && \
    rpm -V  $INSTALL_PKGS && \
    yum clean all && \
    mkdir -p /home/jenkins && \
    chown -R 1001:0 /home/jenkins && \
    chmod -R g+w /home/jenkins && \
    chmod -R 775 /etc/alternatives && \
    chmod -R 775 /var/lib/alternatives && \
    chmod -R 775 /usr/lib/jvm && \
    chmod 775 /usr/bin && \
    chmod 775 /usr/share/man/man1 && \
    mkdir -p /var/lib/origin && \
    chmod 775 /var/lib/origin && \
    unlink /usr/bin/java && \
    unlink /usr/bin/jjs && \
    unlink /usr/bin/keytool && \
    unlink /usr/bin/pack200 && \
    unlink /usr/bin/rmid && \
    unlink /usr/bin/rmiregistry && \
    unlink /usr/bin/unpack200 && \
    unlink /usr/share/man/man1/java.1.gz && \
    unlink /usr/share/man/man1/jjs.1.gz && \
    unlink /usr/share/man/man1/keytool.1.gz && \
    unlink /usr/share/man/man1/pack200.1.gz && \
    unlink /usr/share/man/man1/rmid.1.gz && \
    unlink /usr/share/man/man1/rmiregistry.1.gz && \
    unlink /usr/share/man/man1/unpack200.1.gz

# Copy the entrypoint
ADD contrib/bin/* /usr/local/bin/

# Run the Jenkins JNLP client
ENTRYPOINT ["/usr/bin/go-init", "-main", "/usr/local/bin/run-jnlp-client"]

##############################################
# End
##############################################