# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.140.1/containers/ubuntu/.devcontainer/base.Dockerfile

# [Choice] Ubuntu version: bionic, focal
ARG VARIANT="bionic"
FROM mcr.microsoft.com/vscode/devcontainers/base:0-${VARIANT}

# [Optional] Uncomment this section to install additional OS packages.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive && apt-get -y install --no-install-recommends \
  build-essential \
  dnsutils \
  iputils-ping \
  nmap \
  python3-dev \
  python3-venv \
  software-properties-common \
  tcpdump \
  tmux \
  traceroute \
  vim

# Powershell Core & Az Modules
RUN \
  wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb && \
  dpkg -i packages-microsoft-prod.deb && \
  apt-get update && export DEBIAN_FRONTEND=noninteractive && \
  apt-get install -y powershell && \
  pwsh -Command Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted && \
  pwsh -Command "Set-Variable -Name 'ProgressPreference' -Value 'SilentlyContinue' && Install-Module -Name Az -AllowClobber" && \
  rm packages-microsoft-prod.deb

# Terraform
RUN \
  TF_VERSION=$(curl -s https://checkpoint-api.hashicorp.com/v1/check/terraform | jq -r -M ".current_version") && \
  wget -nv -O terraform.zip https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_amd64.zip && \
  unzip terraform.zip && \
  mv terraform /usr/local/bin && \
  rm terraform.zip

# AWS CLI
RUN \
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
  unzip -q awscliv2.zip && \
  ./aws/install && \
  rm aws* -rf

# Azure CLI
RUN \
  curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && \
  AZ_REPO=$(lsb_release -cs) && \
  echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | tee /etc/apt/sources.list.d/azure-cli.list

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive && apt-get -y install \
  azure-cli

# GitHub CLI
RUN \
  export DEBIAN_FRONTEND=noninteractive && \
  export APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=DontWarn && \
  apt-key adv --keyserver keyserver.ubuntu.com --recv-key C99B11DEB97541F0 && \
  apt-add-repository https://cli.github.com/packages && \
  apt-get update && \
  apt-get -y install gh

# Create Python VENV and activate it
COPY requirements.txt /root
RUN \
  python3 -m venv /root/venv && \
  . /root/venv/bin/activate && \
  pip3 install --no-cache-dir -r /root/requirements.txt

# Adjust .bashrc
RUN \
  echo "source /root/venv/bin/activate" >> /root/.bashrc

# Install Ansible Collections
RUN \
  . /root/venv/bin/activate && \
  ansible-galaxy collection install clay584.genie

# Download NTC-Templates
RUN \
  cd /root && \
  git clone https://github.com/networktocode/ntc-templates.git
