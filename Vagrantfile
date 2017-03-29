# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT
add-apt-repository -y ppa:jonathonf/golang-1.8
apt-get update && apt-get install -y \
    build-essential \
    curl \
    sudo \
    gawk \
    golang-1.8 \
    iptables \
    jq \
    pkg-config \
    libaio-dev \
    libcap-dev \
    libprotobuf-dev \
    libprotobuf-c0-dev \
    libseccomp2 \
    libseccomp-dev \
    protobuf-c-compiler \
    protobuf-compiler \
    python-minimal \
    --no-install-recommends \
   && apt-get clean
echo "export GOPATH=/usr/local" >> /etc/bash.bashrc
echo "export PATH=${PATH}:/usr/lib/go-1.8/bin" >> /etc/bash.bashrc
export GOPATH=/usr/local
export PATH=${PATH}:/usr/lib/go-1.8/bin
source /etc/bash.bashrc

# Add a dummy user for the rootless integration tests. While runC does
# not require an entry in /etc/passwd to operate, one of the tests uses
# `git clone` -- and `git clone` does not allow you to clone a
# repository if the current uid does not have an entry in /etc/passwd.
useradd -u1000 -m -d/home/rootless -s/bin/bash rootless

# install bats
echo "###### Install bats"
cd /tmp \
    && git clone https://github.com/sstephenson/bats.git \
    && cd bats \
    && git reset --hard 03608115df2071fff4eaaff1605768c275e5f81f \
    && ./install.sh /usr/local \
    && rm -rf /tmp/bats
# install criu
echo "###### Install crui"
export CRIU_VERSION=1.7
mkdir -p /usr/src/criu \
    && curl -sSL https://github.com/xemul/criu/archive/v${CRIU_VERSION}.tar.gz | tar -v -C /usr/src/criu/ -xz --strip-components=1 \
    && cd /usr/src/criu \
    && patch -i /usr/local/src/github.com/docker/containerd/ptrace.patch include/ptrace.h \
    && make install-criu \
    && rm -rf /usr/src/criu
# install shfmt
echo "###### Install shfmt"
mkdir -p ${GOPATH}/src/github.com/mvdan \
    && cd ${GOPATH}/src/github.com/mvdan \
    && git clone https://github.com/mvdan/sh \
    && cd sh \
    && git checkout -f v0.4.0 \
    && go install ./cmd/shfmt
# install runC
echo "###### Install runC"
git clone https://github.com/opencontainers/runc.git /usr/local/src/github.com/opencontainers/runc \
    && cd /usr/local/src/github.com/opencontainers/runc \
    && make \
    && make install-bash install
# install busybox
export ROOTFS=/busybox
mkdir -p ${ROOTFS} \
    && curl -o- -sSL 'https://github.com/docker-library/busybox/raw/a0558a9006ce0dd6f6ec5d56cfd3f32ebeeb815f/glibc/busybox.tar.xz' | tar xfJC - ${ROOTFS}
# install containerd
echo "###### Install containerd"
cd /usr/local/src/github.com/docker/containerd \
    && make install
SCRIPT

# This defines the version of vagrant
Vagrant.configure(2) do |config|
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.hostname = "containerd"
  config.vm.synced_folder ".", "/usr/local/src/github.com/docker/containerd"
  #config.vm.network "private_network", ip: machine[:ip]
  config.vm.provision "shell", inline: $script
  config.vm.provider "virtualbox" do |vb|
      vb.memory = 4096
      vb.cpus = 2
  end
end
