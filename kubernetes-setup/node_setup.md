# Node Setup

**NOTE**<br/>
In this section, there are some shell commands which are used to setup the nodes created from Vagrant using `kubeadm`.

---

In this section we will configure the nodes and install prerequisites such as the container runtime `containerd`.

1. Update the apt packages and install packages needed to use the Kubernetes apt repository:
    ```bash
    {
        sudo apt-get update
        sudo apt-get install -y apt-transport-https ca-certificates curl
    }
    ```

1. Set up the required kernel modules and make them persistent
    ```bash
    {
        cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

        sudo modprobe overlay
        sudo modprobe br_netfilter
    }
    ```

1.  Set the required kernel parameters and make them persistent
    ```bash
    {
        cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

        sudo sysctl --system
    }
    ```

1. Install the container runtime
    ```bash
    sudo apt-get install -y containerd
    ```

1.  Configure the container runtime to use systemd Cgroups.

    1. Create default configuration

        ```bash
        {
            sudo mkdir -p /etc/containerd
            containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sudo tee /etc/containerd/config.toml
        }
        ```

    1.  Restart containerd

        ```bash
        sudo systemctl restart containerd
        ```

1.  Determine latest version of Kubernetes and store in a shell variable

    ```bash
    KUBE_LATEST=$(curl -L -s https://dl.k8s.io/release/stable.txt | awk 'BEGIN { FS="." } { printf "%s.%s", $1, $2 }')
    ```

1. Download the Kubernetes public signing key
    ```bash
    {
        sudo mkdir -p /etc/apt/keyrings
        curl -fsSL https://pkgs.k8s.io/core:/stable:/${KUBE_LATEST}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    }
    ```

1. Add the Kubernetes apt repository
    ```bash
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/${KUBE_LATEST}/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```

1. Update apt packages, install kubelet, kubeadm and kubectl, and pin their version
    ```bash
    {
        sudo apt-get update
        sudo apt-get install -y kubelet kubeadm kubectl
        sudo apt-mark hold kubelet kubeadm kubectl
    }
    ```

1.  Configure `crictl` to examine running containers
    ```bash
    sudo crictl config \
        --set runtime-endpoint=unix:///run/containerd/containerd.sock \
        --set image-endpoint=unix:///run/containerd/containerd.sock
    ```

1.  Prepare extra arguments for kubelet such that when it starts, it listens on the VM's primary network address and not any NAT one that may be present. This uses the predefined `PRIMARY_IP` environment variable

    ```bash
    cat <<EOF | sudo tee /etc/default/kubelet
    KUBELET_EXTRA_ARGS='--node-ip ${PRIMARY_IP}'
    EOF
    ```

Next: [Control Plane setup](./05-controlplane.md)