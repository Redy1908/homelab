FROM mcr.microsoft.com/devcontainers/base:debian@sha256:30b0a0c004ca94d36c323ee993361a7e0ae25ea255ea125201e8a9587501c324

# Install additional packages
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
    curl \
    git \
    gnupg2 \
    lsb-release \
    ca-certificates \
    bash-completion \
    direnv \
    yamllint \
    ansible

# Update user to Redy1908
RUN usermod -l Redy1908 -d /home/Redy1908 -m vscode \
    && groupmod -n Redy1908 vscode \
    && echo Redy1908 ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/Redy1908 \
    && chmod 0440 /etc/sudoers.d/Redy1908

# Install kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install talosctl
RUN curl -sL https://talos.dev/install | sh

# Install flux
RUN curl -s https://fluxcd.io/install.sh | bash

# Install sops
RUN curl -LO https://github.com/getsops/sops/releases/download/v3.11.0/sops-v3.11.0.linux.amd64 \
    && mv sops-v3.11.0.linux.amd64 /usr/local/bin/sops \
    && chmod +x /usr/local/bin/sops

# Install age
RUN curl -LO https://github.com/FiloSottile/age/releases/download/v1.3.1/age-v1.3.1-linux-amd64.tar.gz \
    && tar xf age-v1.3.1-linux-amd64.tar.gz \
    && mv age/age /usr/local/bin/age \
    && mv age/age-keygen /usr/local/bin/age-keygen \
    && rm -rf age age-v1.3.1-linux-amd64.tar.gz

# Install kubeconform
RUN curl -L https://github.com/yannh/kubeconform/releases/download/v0.7.0/kubeconform-linux-amd64.tar.gz \
    | tar xz -C /usr/local/bin kubeconform

# Install helm
RUN curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Install yq
RUN wget -O /usr/local/bin/yq "https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64" \
    && chmod +x /usr/local/bin/yq

# Setup aliases and completion and tools
RUN echo "source <(kubectl completion bash)" >> /home/Redy1908/.bashrc \
    && echo "alias k=kubectl" >> /home/Redy1908/.bashrc \
    && echo "complete -F __start_kubectl k" >> /home/Redy1908/.bashrc \
    && echo "source <(flux completion bash)" >> /home/Redy1908/.bashrc \
    && echo "source <(talosctl completion bash)" >> /home/Redy1908/.bashrc \
    && echo "eval '$(direnv hook bash)'" >> /home/Redy1908/.bashrc
