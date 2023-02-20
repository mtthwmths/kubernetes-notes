notes on setting up minikube in CentOS (Stream 9):
    you're going to need docker, because that's what minikube runs within.
    Go to https://docs.docker.com/engine/install/centos/ and follow the install notes.
        $sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        $sudo usermod -aG docker $USER && newgrp docker #https://docs.docker.com/engine/install/linux-postinstall/
    Then go to https://minikube.sigs.k8s.io/docs/start/ and follow the install notes.
        $curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
        $sudo rpm -Uvh minikube-latest.x86_64.rpm
    Now starting minikube is as easy as this:
        $systemctl start docker
        $minikube start
    To use kubens or kubectx for namespace handling:
        $sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
        $sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
        $sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens

Important docs links for getting started:
    https://kubernetes.io/docs/concepts/overview/components/

Other links of interest:
    https://kind.sigs.k8s.io/docs/user/quick-start/
    https://www.redhat.com/en/topics/virtualization/what-is-KVM
    https://minikube.sigs.k8s.io/docs/drivers/kvm2/