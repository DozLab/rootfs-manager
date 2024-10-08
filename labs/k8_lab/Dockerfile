ARG TAG=test
FROM dozman99/lab-base_image:${TAG}

ARG ARCH="amd64"
ARG CONTAINERD_VERSION=1.7.19
# Kubernetes version will be used to keep the kubeadm, kubelet, and kubectl version consistent
ARG KUBERNETES_VERSION=1.30

RUN apt-get update && apt-get install -y \
        dnsutils \
        cloud-init \
        linux-image-virtual \
        less \
        apt-transport-https \
        ca-certificates \
        gnupg2 \
        software-properties-common \
        libseccomp2 \
        jq && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* 




#### Let iptables see bridged traffic ###
RUN echo "overlay \n\
br_netfilter" >> /etc/modules-load.d/containerd.conf

# Setup required sysctl params, these persist across reboots.
RUN echo "net.bridge.bridge-nf-call-iptables  = 1 \n\
net.bridge.bridge-nf-call-ip6tables = 1 \n\
net.ipv4.ip_forward                 = 1 \n\
net.ipv6.conf.all.forwarding = 1 \n\
net.ipv6.conf.all.disable_ipv6 = 0 \n\
net.ipv4.tcp_congestion_control = bbr \n\
vm.overcommit_memory = 1 \n\
kernel.panic = 10 \n\
net.ipv4.conf.all.rp_filter = 1 \n\
kernel.panic_on_oops = 1" >> /etc/sysctl.d/99-kubernetes-cri.conf

# Apply sysctl params without reboot
RUN sysctl --system

# Install Containerd
RUN wget https://github.com/containerd/containerd/releases/download/v${CONTAINERD_VERSION}/cri-containerd-cni-${CONTAINERD_VERSION}-linux-${ARCH}.tar.gz
RUN tar --no-overwrite-dir -C / -xzf cri-containerd-cni-${CONTAINERD_VERSION}-linux-${ARCH}.tar.gz && rm -f cri-containerd-cni-${CONTAINERD_VERSION}-linux-${ARCH}.tar.gz

### Add apt repos
# Kubeadm, Kubelet, and Kubectl
RUN sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gpg 
RUN mkdir -p -m 755 /etc/apt/keyrings && curl -fsSL https://pkgs.k8s.io/core:/stable:/v${KUBERNETES_VERSION}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg 
RUN echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v'${KUBERNETES_VERSION}'/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list



RUN apt-get update && apt-get install -y \
        kubeadm\
        kubelet \
        kubectl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    apt-mark hold kubelet kubeadm kubectl

RUN kubeadm version

RUN mkdir -p /etc/containerd && containerd config default > /etc/containerd/config.toml
RUN rm /etc/cni/net.d/10-containerd-net.conflist
RUN systemctl enable containerd


RUN mkdir -p  /etc/systemd/system/kubelet.service.d/
RUN echo '[Service] \nEnvironment="KUBELET_EXTRA_ARGS=--container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock"' | tee /etc/systemd/system/kubelet.service.d/0-containerd.conf


RUN systemctl enable kubelet


# Create the following files, but unset them
RUN echo "" > /etc/machine-id && echo "" > /var/lib/dbus/machine-id

RUN mkdir -m 0700 -p /root/.ssh

# Set the root password to root when logging in through the VM's ttyS0 console
RUN echo "root:root" | chpasswd

