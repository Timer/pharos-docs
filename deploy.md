# Deploy Kontena Pharos Cluster

> **Prerequisites:** Spin up machines for your cluster. You can use machines from any infrastructure. These machines must meet the [host system requirements](/requirements.md) and they must be accessible via SSH. Additionally, you will need `pharos` CLI tool that is part of [Kontena Pharos CLI Toolchain](/install.md).

Kontena Pharos cluster is deployed using `pharos` CLI tool. The [cluster configuration](/configuration.md) is described in `cluster.yml` file.

For example, your `cluster.yml` file might look like this:

```yaml
hosts:
  - address: "192.0.2.1"
    user: root
    ssh_key_path: ~/.ssh/my_key
    role: master
  - address: "192.0.2.2"
    role: worker
  - address: "192.0.2.3"
    role: worker
network:
  provider: weave
addons:
  ingress-nginx:
     enabled: true
```

Once you have created `cluster.yml` file with your desired [cluster configuration](/configuration.md)  options, the cluster may be deployed simply by running command:

```sh
$ pharos up -c cluster.yml
```

NOTE! The `pharos up` command is also used for changing the cluster configuration. It is safe to run this command multiple times. If there are no changes in configuration, nothing will be done to your cluster.


## Adding and removing nodes

Adding and removing nodes is highly dependent on the roles of the nodes. Following chapters covers the process of adding and removing nodes in different roles.

### Notes on master nodes and etcd

Keep in mind that the master nodes also host the control plane etcd cluster. Even more importantly, any changes to the cluster needs a working etcd cluster with the majority of peers present and working.

As normally with etcd, or any other quorum based system, it is highly advisable to run odd number of peers. As the etcd cluster only works when a majority can be formed, once you grow the control plane to have > 1 nodes, you cannot (automatically) go back to 1 node.

TODO: Add link to etcd availability docs, once I find the page...

### Adding Master nodes

Adding master nodes is as simple as adding them into the `cluster.yml`. Re-running `pharos up ...` will configure the control plane on the new node and also makes necessary changes in the etcd cluster.

### Removing Master nodes

Once you've determined that it is safe to remove a master node, and it's etcd peer, follow this process:
1. Stop node
2. Update `cluster.yml`
3. Run `pharos up ...`
4. Remove the node from Kubernetes API with `kubectl delete node <node name>`
5. Terminate/remove the node in your infrastructure

### Adding Worker nodes

Adding worker nodes is as simple as adding them into the `cluster.yml`. Re-running `pharos up ...` will configure the everything on the new node and joins the node into the cluster.

### Removing Worker nodes

Removing worker node is currently multi step process:
1. Remove host from `cluster.yml`. As re-upping a cluster would not actually do anything, you do not need to run `pharos up ...`
2. (optional) Move the workload away from the node with `kubectl drain --timeout=5m --ignore-daemonsets --delete-local-data`
3. Terminate host
4. Remove the node from Kubernetes API with `kubectl delete node <node name>`