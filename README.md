## Magnum-Senlin ##

### Environment ###

Our test server environment is described as follows:

- **OS**: Ubuntu Server 14.04
- **Memory**: 256 GB
- **NIC**: Single network interface

### Configuration ###

First, you should `clone` the **devstack**:

`sudo mkdir -p /opt/stack`

`sudo chown $USER /opt/stack`

`git clone https://git.openstack.org/openstack-dev/devstack /opt/stack/devstack`

Then, create the `local.conf` file in the root directory of devstack. Our local.conf is:

    [[local|localrc]]

    SERVICE_TOKEN=stacktoken
    ADMIN_PASSWORD=stack
    DATABASE_PASSWORD=stack
    RABBIT_PASSWORD=stack
    SERVICE_PASSWORD=$ADMIN_PASSWORD
    
    enable_plugin magnum https://git.openstack.org/openstack/magnum
    VOLUME_BACKING_FILE_SIZE=20G

    enable_plugin senlin https://git.openstack.org/stackforge/senlin

*Note:* If you just have single network interface, you should not set something like `PUBLIC_INTERFACE=eth1`.

After that, you can `cd /opt/stack/devstack`, then `./stack.sh`

### Checkout our test code ###

We maintain a test branch of **Magnum** on Github. If you can use devstack with the local.conf above, it proves that you environment fits the latest magnum.

Then you can `./unstack.sh`, and checkout our code with the following steps:

`cd /opt/stack/magnum`

`git checkout -b surmagnum`

`git remote add sur https://github.com/Tennyson53/magnum`

`git pull sur master:surmagnum`

Finally, you can just re-run `./stack.sh` to re-install all services.

### Have a try ###

To be continued...


### Bugs and Problems ###

*2015.9.23*

- We cannot use the q-lbaasv2 as the load-balance service due to magnum will enable the q-lbaasv1 as default. If we comment out the `enable_service q-lbaas` in the `settings` of magnum, and enable the q-lbaasv2, magnum will fail to create bay.
- For the auto-scaling,  we have found that cAdvisor was integrated into the Kublete binary in kubernetes 1.0. It collects the CPU, memory, filesystem, and network usage of containers.
- [The doc of kubernetes](http://kubernetes.io/v1.0/docs/user-guide/monitoring.html) claims it can provide users with detailed resource usage information about running applications at containers, pods, services, and **whole cluster**. We need to see what it really gives us. *Maybe Zhenqi knows more about that*
- The *KubernetesClientCode* is 
[here](https://github.com/bolan2014/KubernetesClientCode).
- We need a definite user case to deep understand when we should trigger the VMs auto-scaling that is based on the information collected from containers (container to VM).
- We have imaged a case as follow: the cluster's free memory is not enough to create a new container, and ceilometer cannot get the instance's resource usage in real-time, as a result, if a user create the container before ceilometer knows the actual resource usage, the instance may crash. Therefore we need to use the information collected in container layer to trigger the auto-scaling in VM layer, and then the new container will be created in the new instance. 

*2015.9.25*

- The *KubernetesClientCode* is 
[here](https://github.com/bolan2014/KubernetesClientCode).
- We are really stuck on the ERROR that **Too many connections** raised by the mariadb in CentOS 7. It could occur at any time, such as the process of *stack.sh* or running some commands (like *magnum bay-create* or *nova list*). We can temporary solve this by restart the mariadb.service in CentOS 7, but for **magnum bay-create** which takes long time, the bug of mariadb in CentOS 7 could make it fail to process.
- We faild to install Ubuntu on our server last week. Now we will quickly re-install the server with Ubuntu with the server provider's support. Hope to have a nice weekend! 

*2015.9.27*

- We have manually tried to create two kinds of nodes (master node and minion nodes) by senlin node-create with two kinds of profiles created before. The template property in spec file of two kinds of profiles are temporarily [master.yaml](https://github.com/openstack/magnum/blob/master/magnum/templates/heat-kubernetes/kubemaster.yaml) and [minion.yaml](https://github.com/openstack/magnum/blob/master/magnum/templates/heat-kubernetes/kubeminion.yaml). And all creation is successful!
- When we create profile with type os.heat.stack, the parameters needed by template should be added to spec file. I think senlin profile-create should add a new argument -P which means parameters and pass these parameters to server side.
