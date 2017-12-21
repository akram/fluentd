FROM centos:centos7

MAINTAINER OpenShift Development <dev@lists.openshift.redhat.com>

ENV DATA_VERSION=1.6.0 \
    FLUENTD_VERSION=0.12.29 \
    FLUENTD_PLUGIN_ELASTICSEARCH_VERSION=1.9.2 \
    FLUENTD_PLUGIN_REWRITE_TAG_FILTER_VERSION=1.6.0 \
    GEM_HOME=/opt/app-root/src \
    HOME=/opt/app-root/src \
    PATH=/opt/app-root/src/bin:/opt/app-root/bin:$PATH \
    RUBY_VERSION=2.0

LABEL io.k8s.description="Fluentd container for collecting of docker container logs" \
      io.k8s.display-name="Fluentd ${FLUENTD_VERSION}" \
      io.openshift.expose-services="9200:http, 9300:http" \
      io.openshift.tags="logging,elk,fluentd"

# activesupport version 5.x requires ruby 2.2
# iproute needed for ip command to get ip addresses
RUN rpmkeys --import file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 && \
    yum makecache fast && \
    yum install -y --setopt=tsflags=nodocs \
      gcc-c++ \
      libcurl-devel \
      make \
      bc \
      iproute \
      libyaml \
      openssl \
      wget && \
    yum clean all && \
    wget https://github.com/feedforce/ruby-rpm/releases/download/2.2.4/ruby-2.2.4-1.el7.centos.x86_64.rpm -O /tmp/ruby-2.2.4-1.el7.centos.x86_64.rpm && \
    rpm -Uhv /tmp/ruby-2.2.4-1.el7.centos.x86_64.rpm && \
    rm /tmp/ruby-2.2.4-1.el7.centos.x86_64.rpm

RUN mkdir -p ${HOME} && \
     gem install -N --conservative --minimal-deps --no-ri \
     'activesupport:<5' \
     'elasticsearch-transport:<5' \
     'elasticsearch-api:<5' \
     'elasticsearch:<5' \
      fluentd:${FLUENTD_VERSION} \
      fluent-plugin-elasticsearch:${FLUENTD_PLUGIN_ELASTICSEARCH_VERSION} \
      fluent-plugin-kubernetes_metadata_filter \
      fluent-plugin-rewrite-tag-filter:${FLUENTD_PLUGIN_REWRITE_TAG_FILTER_VERSION} \
      fluent-plugin-secure-forward \
     'fluent-plugin-systemd:<0.1.0' \
      fluent-plugin-viaq_data_model \
      fluent-plugin-gelf-hs \
      systemd-journal \
      json \
      io-console \
      rdoc \
      bigdecimal \
      psych 

ADD configs.d/ /etc/fluent/configs.d/
ADD run.sh generate_throttle_configs.rb ${HOME}/

RUN mkdir -p /etc/fluent/configs.d/{dynamic,user} && \
    mkdir /etc/fluent/plugin && \
    chmod 777 /etc/fluent/configs.d/dynamic /etc/fluent/plugin && \
    ln -s /etc/fluent/configs.d/user/fluent.conf /etc/fluent/fluent.conf

ADD filter-common-data-model.rb /etc/fluent/plugin

WORKDIR ${HOME}
USER 0
CMD ["sh", "run.sh"]
