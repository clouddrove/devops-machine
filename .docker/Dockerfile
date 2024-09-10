# Pull base image.
FROM phusion/baseimage:jammy-1.0.4

#####################################
ENV LANG=C.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV DEBIAN_FRONTEND=noninteractive

###
ENV PACKER_VERSION=1.11.2
ENV TERRAFORM_VERSION=1.9.5
ENV ANSIBLE_VERSION=10.3.0
ENV LINT_VERSION=4.1.0
ENV KUBECTL_VERSION=1.31.0
ENV HELM_VERSION=3.15.4
ENV AZURE_CLI_VERSION=2.64.0-1~jammy
ENV AWS_CLI_VERSION=2.17.47
ENV K9s_Version=0.32.5
ENV GCLOUD_VERSION=490.0.0
ENV KUI_Version=13.1.4
ENV KUBECTX_VERSION=0.9.4
ENV KUBENS_VERSION=0.9.4

####################################

#User
RUN useradd ubuntu
RUN passwd -d ubuntu
RUN passwd -d root
RUN echo "PermitEmptyPasswords yes" >> /etc/ssh/sshd_config && echo "ubuntu ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers 
RUN echo "ChallengeResponseAuthentication no" >> /etc/ssh/sshd_config
RUN echo "PubkeyAuthentication yes" >> /etc/ssh/sshd_config
RUN chsh -s `which bash` ubuntu
RUN usermod -d /home/ubuntu ubuntu
RUN mkdir -p /home/ubuntu
RUN mkdir -p /home/ubuntu/.ssh
RUN mkdir -p /home/ubuntu/.aws && \
    touch /home/ubuntu/.aws/credentials
RUN mkdir -p /home/ubuntu/.azure && \
    mkdir -p /home/ubuntu/.config/gcloud/
RUN chmod 755 /home/ubuntu/.ssh
RUN chown -R ubuntu:ubuntu /home/ubuntu
RUN chmod 755 /home/ubuntu
ENV BOOT2DOCKER_ID=501
ENV BOOT2DOCKER_GID=20
# Tweaks to give write permissions to the app
RUN usermod -u ${BOOT2DOCKER_ID} ubuntu && \
    usermod -G staff ubuntu
RUN groupmod -g $(($BOOT2DOCKER_GID + 10000)) $(getent group $BOOT2DOCKER_GID | cut -d: -f1) && groupmod -g ${BOOT2DOCKER_GID} staff
RUN add-apt-repository ppa:longsleep/golang-backports

#General
RUN apt remove -y python3
RUN apt update && apt upgrade -y
RUN apt install -y      \
    curl                   \
    git                    \
    zip                    \
    unzip                  \
    vim                    \
    ruby-full              \
    openssh-server         \
    zsh                    \
    figlet                 \
    sysvbanner             \
    htop                   \
    python3-pip            \
    wget                   \
    ca-certificates        \
    openssl                \
    yamllint               \
    inetutils-ping         \
    telnet                 \
    golang-go              \
    libssl-dev             \
    apt-transport-https    \
    ca-certificates        \
    gnupg


# ZSH
ADD .docker/etc/install-zsh.sh /root/install-zsh.sh
ADD .docker/etc/install-zsh.sh /home/ubuntu/install-zsh.sh

RUN chmod +x /root/install-zsh.sh
RUN chmod +x /home/ubuntu/install-zsh.sh
RUN sh /root/install-zsh.sh
RUN su - ubuntu -c "sh /home/ubuntu/install-zsh.sh"
RUN rm /root/.zshrc && chsh -s `which zsh` && chsh -s `which zsh` ubuntu && chmod -R 755 /usr/local/share/zsh*

##Python with Packages
RUN pip install ansible==${ANSIBLE_VERSION} \
                 ansible-lint==${LINT_VERSION} \
                 mongotail \
                 bumpversion \
                 pip \
                 yamllint

#terraform  
RUN cd /tmp && \
     wget https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip && \
     unzip -o terraform_${TERRAFORM_VERSION}_linux_amd64.zip -d /usr/bin && \
     rm terraform_${TERRAFORM_VERSION}_linux_amd64.zip

# Packer
RUN cd /tmp && \
    wget https://releases.hashicorp.com/packer/${PACKER_VERSION}/packer_${PACKER_VERSION}_linux_amd64.zip && \
    unzip -o packer_${PACKER_VERSION}_linux_amd64.zip -d /usr/bin && \
    rm packer_${PACKER_VERSION}_linux_amd64.zip

#Kubectl
RUN cd /usr/bin && \
    curl -LO https://storage.googleapis.com/kubernetes-release/release/v${KUBECTL_VERSION}/bin/linux/amd64/kubectl && \
    chmod +x ./kubectl

# Helm
RUN cd /tmp && \
    wget https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz && \
    tar -zxvf helm* && \
    mv linux-amd64/helm /usr/local/bin/helm && \
    rm helm* && \
    rm -rf linux-amd64 helm*

#Azure Cli
RUN apt-get install -y lsb-release && \
    mkdir -p /etc/apt/keyrings && \
    curl -sLS https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/keyrings/microsoft.gpg && \
    chmod go+r /etc/apt/keyrings/microsoft.gpg && \
    AZ_DIST=$(lsb_release -cs) && \
    echo "Types: deb\nURIs: https://packages.microsoft.com/repos/azure-cli/\nSuites: ${AZ_DIST}\nComponents: main\nArchitectures: $(dpkg --print-architecture)\nSigned-by: /etc/apt/keyrings/microsoft.gpg" | tee /etc/apt/sources.list.d/azure-cli.sources && \
    apt-get update && \
    apt-get install azure-cli=${AZURE_CLI_VERSION} && \
    rm -rf /var/lib/apt/lists/*

#Aws Cli
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64-${AWS_CLI_VERSION}.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install -i /usr/local/aws-cli -b /usr/local/bin && \
    mv /usr/local/bin/aws /bin
    
#gcloud
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \ 
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg && \ 
    apt-get update -y && apt-get install google-cloud-cli=${GCLOUD_VERSION}-0 -y

#k9s
RUN curl -LO https://github.com/derailed/k9s/releases/latest/download/k9s_Linux_amd64.tar.gz && \
    tar -xzf k9s_Linux_amd64.tar.gz && \
    mv k9s /usr/local/bin/ && \
    rm k9s_Linux_amd64.tar.gz

#Kubectx
RUN git clone https://github.com/ahmetb/kubectx /opt/kubectx \
    && ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx \
    && ln -s /opt/kubectx/kubens /usr/local/bin/kubens

#tfswitch
RUN curl -L https://raw.githubusercontent.com/warrensbox/terraform-switcher/release/install.sh | bash -s -- -b /usr/local/bin

#kubectx
RUN curl -L https://github.com/ahmetb/kubectx/releases/download/v${KUBECTX_VERSION}/kubectx -o /usr/local/bin/kubectx && \
    chmod +x /usr/local/bin/kubectx

#kubens
RUN curl -L https://github.com/ahmetb/kubectx/releases/download/v${KUBENS_VERSION}/kubens -o /usr/local/bin/kubens && \
    chmod +x /usr/local/bin/kubens && \
    mkdir -p /home/ubuntu/.kube/config && \
    chmod 644 /home/ubuntu/.kube/config

#Opentofu
RUN cd /tmp
RUN curl --proto '=https' --tlsv1.2 -fsSL https://get.opentofu.org/install-opentofu.sh -o install-opentofu.sh && \
     chmod +x install-opentofu.sh && \
     ./install-opentofu.sh --install-method standalone && \
     rm -f install-opentofu.sh

# Clean up APT when done.
RUN apt clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/ /var/cache/apk/**
WORKDIR /home/ubuntu/workspace

# Define default command.
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]