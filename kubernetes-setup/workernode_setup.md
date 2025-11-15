# Boot the worker nodes

Here we will join the worker nodes to the cluster. We will need the `kubeadm join` command from the previous step

Ref: https://stackoverflow.com/questions/59629319/unable-to-upgrade-connection-pod-does-not-exist

## Join Workers

On each of `node01` and `node02` do the following

1.  Become root

    ```
    sudo su
    ```

1.  Join the node

    > Paste the `kubeadm join` command output by `kubeadm init` on the control plane

### Verify

On `controlplane` run the following. After a few seconds both nodes should be ready

```
kubectl get nodes
```