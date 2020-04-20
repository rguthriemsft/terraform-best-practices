FROM mcr.microsoft.com/azure-cli:latest

RUN apk add --no-cache ca-certificates git bash curl jq vim

############### Installing Helm ###############
ARG HELM_VERSION=v3.1.0
ARG HELM_LOCATION="https://get.helm.sh"
ARG HELM_FILENAME="helm-${HELM_VERSION}-linux-amd64.tar.gz"
ARG HELM_SHA256="f0fd9fe2b0e09dc9ed190239fce892a468cbb0a2a8fffb9fe846f893c8fd09de"
RUN wget ${HELM_LOCATION}/${HELM_FILENAME} && \
    echo Verifying ${HELM_FILENAME}... && \
    sha256sum ${HELM_FILENAME} | grep -q "${HELM_SHA256}" && \
    echo Extracting ${HELM_FILENAME}... && \
    tar zxvf ${HELM_FILENAME} && mv /linux-amd64/helm /usr/local/bin/ && \
    rm ${HELM_FILENAME} && rm -r /linux-amd64

############### Installing kubectl ###############
ENV KUBECTL_VERSION="v1.17.3"
ENV KUBECTL_SHA256="ae8627adb1f0ae7bdb82ffd74a579953b8acdcd4b94aeefc7569078c3d7994c6"
RUN curl -LO "https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" && \
    sha256sum kubectl | grep ${KUBECTL_SHA256} && \
    chmod +x kubectl && \
    mv kubectl /usr/local/bin/kubectl

############### Installing terraform ###############
ENV TERRAFORM_VERSION=0.12.24
ENV TERRAFORM_SHA256=602d2529aafdaa0f605c06adb7c72cfb585d8aa19b3f4d8d189b42589e27bf11
RUN curl -fsSLo terraform.zip "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip"
RUN sha256sum *terraform.zip | grep -q "${TERRAFORM_SHA256}" && \
    unzip terraform.zip -d /usr/local/bin/ && \
    rm terraform.zip

############### Set Labels ###############
LABEL HELM_VERSION=$HELM_VERSION
LABEL KUBECTL_VERSION=$KUBECTL_VERSION
LABEL TERRAFORM_VERSION=$TERRAFORM_VERSION

############### Installing Some Helpful Packages ###############

CMD bash