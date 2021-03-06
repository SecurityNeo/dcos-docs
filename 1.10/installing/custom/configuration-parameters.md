---
post_title: Install Configuration Parameters
nav_title: Config
menu_order: 600
---

These configuration parameters are specified in [YAML][1] format in your config.yaml file. During DC/OS installation the configuration file is used to generate a customized DC/OS build.

*  [Cluster Setup](#cluster-setup)
*  [Security and Authentication](#security-and-authentication)
*  [Networking](#networking)
*  [Performance and Tuning](#performance-and-tuning)
*  [Example Configurations](#examples1)

# <a name="cluster-setup"></a>Cluster Setup

### agent_list
This parameter specifies a YAML nested list (`-`) of IPv4 addresses to your [private agent](/docs/1.10/overview/concepts/#private) host names.

### bootstrap_url
This required parameter specifies the URI path for the DC/OS installer to store the customized DC/OS build files. If you are using the automated DC/OS installer, you should specify `bootstrap_url: file:///opt/dcos_install_tmp` unless you have moved the installer assets. By default the automated DC/OS installer places the build files in `file:///opt/dcos_install_tmp`.

### cluster_docker_credentials
This parameter specifies a dictionary of Docker credentials to pass. 

- If unset, a default empty credentials file is created at `/etc/mesosphere/docker_credentials` during DC/OS install. A sysadmin can change credentials as needed. A `systemctl restart dcos-mesos-slave` or `systemctl restart dcos-mesos-slave-public` is required for changes to take effect.
- You can also specify by using the `--docker_config` JSON [format](http://mesos.apache.org/documentation/latest/configuration/). You can write as YAML in the `config.yaml` file and it will automatically be mapped to the JSON format for you. This will store the Docker credentials in the same location as the DC/OS internal configuration (`/opt/mesosphere`). If you need to update or change the configuration, you will have to create a new DC/OS internal configuration.

You can use the following options to further configure the Docker credentials:

*  **cluster_docker_credentials_dcos_owned** This parameter specifies whether to store the credentials file in `/opt/mesosphere` or `/etc/mesosphere/docker_credentials`. A sysadmin cannot edit `/opt/mesosphere` directly.

    *  `cluster_docker_credentials_dcos_owned: 'true'` The credentials file is stored in `/opt/mesosphere`.
    
        *  **cluster_docker_credentials_write_to_etc** This parameter specifies whether to write a cluster credentials file.
        
            *  `cluster_docker_credentials_write_to_etc: 'true'` Write a credentials file. This can be useful if overwriting your credentials file will cause problems (e.g. if it is part of a machine image or AMI). This is the default value.
            *  `cluster_docker_credentials_write_to_etc: 'false'` Do not write a credentials file.
            
    *  `cluster_docker_credentials_dcos_owned: 'false'` The credentials file is stored in `/etc/mesosphere/docker_credentials`.

*  **cluster_docker_credentials_enabled** This parameter specifies whether to pass the Mesos `--docker_config` option to Mesos. 

    *  `cluster_docker_credentials_enabled: 'true'` Pass the Mesos `--docker_config` option to Mesos. It will point to a file that contains the provided `cluster_docker_credentials` data.
    *  `cluster_docker_credentials_enabled: 'false'` Do not pass the Mesos `--docker_config` option to Mesos. 
    
For more information, see the [examples](#docker-credentials).

### cluster_docker_registry_url
This parameter specifies a custom URL that Mesos uses to pull Docker images from. If set, it will configure the Mesos' `--docker_registry` flag to the specified URL. This changes the default URL Mesos uses for pulling Docker images. By default `https://registry-1.docker.io` is used.

### cluster_name
This parameter specifies the name of your cluster.

### cosmos_config
This parameter specifies a dictionary of packaging configuration to pass to the [DC/OS package manager](https://github.com/dcos/cosmos). If set, the following options must also be
specified.

* **staged_package_storage_uri**
  This parameter specifies where to temporarily store DC/OS packages while they are being added.
  The value must be a file URL, for example, `file:///var/lib/dcos/cosmos/staged-packages`.
* **package_storage_uri**
  This parameter specifies where to permanently store DC/OS packages. The value must be a file URL,
  for example, `file:///var/lib/dcos/cosmos/packages`.

### exhibitor_storage_backend
This parameter specifies the type of storage backend to use for Exhibitor. You can use internal DC/OS storage (`static`) or specify an external storage system (`zookeeper`, `aws_s3`, and `azure`) for configuring and orchestrating ZooKeeper with Exhibitor on the master nodes. Exhibitor automatically configures your ZooKeeper installation on the master nodes during your DC/OS installation.

*   `exhibitor_storage_backend: static`
    This option specifies that the Exhibitor storage backend is managed internally within your cluster.
*   `exhibitor_storage_backend: zookeeper`
    This option specifies a ZooKeeper instance for shared storage. If you use a ZooKeeper instance to bootstrap Exhibitor, this ZooKeeper instance must be separate from your DC/OS cluster. You must have at least 3 ZooKeeper instances running at all times for high availability. If you specify `zookeeper`, you must also specify these parameters.
    *   **exhibitor_zk_hosts**
        This parameter specifies a comma-separated list (`<ZK_IP>:<ZK_PORT>, <ZK_IP>:<ZK_PORT>, <ZK_IP:ZK_PORT>`) of one or more ZooKeeper node IP and port addresses to use for configuring the internal Exhibitor instances. Exhibitor uses this ZooKeeper cluster to orchestrate it's configuration. Multiple ZooKeeper instances are recommended for failover in production environments.
    *   **exhibitor_zk_path**
        This parameter specifies the filepath that Exhibitor uses to store data.
*   `exhibitor_storage_backend: aws_s3`
    This option specifies an Amazon Simple Storage Service (S3) bucket for shared storage. If you specify `aws_s3`, you must also specify these parameters:
    *  **aws_access_key_id**
       This parameter specifies AWS key ID.
    *  **aws_region**
       This parameter specifies AWS region for your S3 bucket.
    *  **aws_secret_access_key**
       This parameter specifies AWS secret access key.
    *  **exhibitor_explicit_keys**
       This parameter specifies whether you are using AWS API keys to grant Exhibitor access to S3.
        *  `exhibitor_explicit_keys: 'true'`
           If you're  using AWS API keys to manually grant Exhibitor access.
        *  `exhibitor_explicit_keys: 'false'`
           If you're using AWS Identity and Access Management (IAM) to grant Exhibitor access to s3.
    *  **s3_bucket**
       This parameter specifies name of your S3 bucket.
    *  **s3_prefix**
       This parameter specifies S3 prefix to be used within your S3 bucket to be used by Exhibitor.

       **Tip:** AWS EC2 Classic is not supported.
*   `exhibitor_storage_backend: azure`
   This option specifies an Azure Storage Account for shared storage. The data will be stored under the container named `dcos-exhibitor`. If you specify `azure`, you must also specify these parameters:
    *  **exhibitor_azure_account_name**
       This parameter specifies the Azure Storage Account Name.
    *  **exhibitor_azure_account_key**
       This parameter specifies a secret key to access the Azure Storage Account.
    *  **exhibitor_azure_prefix**
       This parameter specifies the blob prefix to be used within your Storage Account to be used by Exhibitor.


### <a name="master"></a>master_discovery
This required parameter specifies the Mesos master discovery method. The available options are `static` or `master_http_loadbalancer`.

*  `master_discovery: static`
This option specifies that Mesos agents are used to discover the masters by giving each agent a static list of master IPs. The masters must not change IP addresses, and if a master is replaced, the new master must take the old master's IP address. If you specify `static`, you must also specify this parameter:

    *  **master_list**
       This required parameter specifies a list of your static master IP addresses as a YAML nested series (`-`).

*   `master_discovery: master_http_loadbalancer` This option specifies that the set of masters has an HTTP load balancer in front of them. The agent nodes will know the address of the load balancer. They use the load balancer to access Exhibitor on the masters to get the full list of master IPs. If you specify `master_http_load_balancer`, you must also specify these parameters:

    *  **exhibitor_address** This required parameter specifies the location (preferably an IP address) of the load balancer in front of the masters. The load balancer must accept traffic on ports 80, 443, 2181, 5050, 8080, 8181. The traffic must also be forwarded to the same ports on the master. For example, Mesos port 5050 on the load balancer should forward to port 5050 on the master. The master should forward any new connections via round robin, and should avoid machines that do not respond to requests on Mesos port 5050 to ensure the master is up.
    *  **num_masters**
       This required parameter specifies the number of Mesos masters in your DC/OS cluster. It cannot be changed later. The number of masters behind the load balancer must never be greater than this number, though it can be fewer during failures.

*Note*: On platforms like AWS where internal IPs are allocated dynamically, you should not use a static master list. If a master instance were to terminate for any reason, it could lead to cluster instability.

### <a name="public-agent"></a>public_agent_list
This parameter specifies a YAML nested list (`-`) of IPv4 addresses to your [public agent](/docs/1.10/overview/concepts/#public-agent-node) host names.

### <a name="platform"></a>platform
This parameter specifies the infrastructure platform. The value is optional, free-form with no content validation, and used for telemetry only. Please supply an appropriate value to help inform DC/OS platform prioritization decisions. Example values: `aws`, `azure`, `oneview`, `openstack`, `vsphere`, `vagrant-virtualbox`, `onprem` (default).

## <a name="networking"></a>Networking

### <a name="dcos-overlay-enable"></a>dcos_overlay_enable

This parameter specifies whether to enable DC/OS virtual networks.

**Important:** Virtual networks require minimum Docker version 1.11. If you are using Docker 1.10 or earlier, you must specify `dcos_overlay_enable: 'false'`. For more information, see the [system requirements](/docs/1.10/installing/custom/system-requirements/).

*  `dcos_overlay_enable: 'false'` Do not enable the DC/OS virtual network.
*  `dcos_overlay_enable: 'true'` Enable the DC/OS virtual network. This is the default value. When the virtual network is enabled you can also specify the following parameters:

    *  `dcos_overlay_config_attempts` This parameter specifies how many failed configuration attempts are allowed before the overlay configuration modules stop trying to configure an virtual network.

        __Tip:__ The failures might be related to a malfunctioning Docker daemon.

    *  `dcos_overlay_mtu` This parameter specifies the maximum transmission unit (MTU) of the Virtual Ethernet (vEth) on the containers that are launched on the overlay.

    *  `dcos_overlay_network` This group of parameters define an virtual network for DC/OS.  The default configuration of DC/OS provides an virtual network named `dcos` whose YAML configuration is as follows:

        ```
        dcos_overlay_network:
            vtep_subnet: 44.128.0.0/20
            vtep_mac_oui: 70:B3:D5:00:00:00
            overlays:
              - name: dcos
                subnet: 9.0.0.0/8
                prefix: 26
        ```

        *  `vtep_subnet` This parameter specifies a dedicated address space that is used for the VxLAN backend for the virtual network. This address space should not be routeable from outside the agents or master.
        *  `vtep_mac_oui` This parameter specifies the MAC address of the interface connecting to it in the public node.
            
            **Important:** The last 3 bytes must be `00`.
        *  __overlays__
            *  `name` This parameter specifies the canonical name (see [limitations](/docs/1.10/networking/virtual-networks/) for constraints on naming virtual networks).
            *  `subnet` This parameter specifies the subnet that is allocated to the virtual network.
            *  `prefix` This parameter specifies the size of the subnet that is allocated to each agent and thus defines the number of agents on which the overlay can run. The size of the subnet is carved from the overlay subnet.

 For more information see the [example](#overlay) and [documentation](/docs/1.10/networking/virtual-networks/).

### <a name="dns-search"></a>dns_search
This parameter specifies a space-separated list of domains that are tried when an unqualified domain is entered (e.g. domain searches that do not contain &#8216;.&#8217;). The Linux implementation of `/etc/resolv.conf` restricts the maximum number of domains to 6 and the maximum number of characters the setting can have to 256. For more information, see <a href="http://man7.org/linux/man-pages/man5/resolv.conf.5.html">man /etc/resolv.conf</a>.

A `search` line with the specified contents is added to the `/etc/resolv.conf` file of every cluster host. `search` can do the same things as `domain` and is more extensible because multiple domains can be specified.

In this example, `example.com` has public website `www.example.com` and all of the hosts in the datacenter have fully qualified domain names that end with `dc1.example.com`. One of the hosts in your datacenter has the hostname `foo.dc1.example.com`. If `dns_search` is set to &#8216;dc1.example.com example.com&#8217;, then every DC/OS host which does a name lookup of foo will get the A record for `foo.dc1.example.com`. If a machine looks up `www`, first `www.dc1.example.com` would be checked, but it does not exist, so the search would try the next domain, lookup `www.example.com`, find an A record, and then return it.

```yaml
dns_search: dc1.example.com dc1.example.com example.com dc1.example.com dc2.example.com example.com
```
### <a name="#resolvers"></a>resolvers

This required parameter specifies a YAML nested list (`-`) of DNS resolvers for your DC/OS cluster nodes. You can specify a maximum of 3 resolvers. Set this parameter to the most authoritative nameservers that you have.

-  If you want to resolve internal hostnames, set it to a nameserver that can resolve them.
-  If you do not have internal hostnames to resolve, you can set this to a public nameserver like Google or AWS. For example, you can specify the [Google Public DNS IP addresses (IPv4)](https://developers.google.com/speed/public-dns/docs/using):

    ```bash
    resolvers:
    - 8.8.4.4
    - 8.8.8.8
    ```
-  If you do not have a DNS infrastructure and do not have access to internet DNS servers, you can specify `resolvers: []`. By specifying this setting, all requests to non-`.mesos` will return an error. For more information, see the Mesos-DNS [documentation](/docs/1.10/networking/mesos-dns/).

**Caution:** If you set the `resolvers` parameter incorrectly, you will permanently damage your configuration and have to reinstall DC/OS.

### dns_bind_ip_blacklist
This parameter specifies a list of IP addresses that DC/OS DNS resolvers cannot bind to.

### dns_forward_zones
This parameter specifies a nested list of DNS zones, IP addresses and ports that configure custom forwarding behavior of DNS queries. A DNS zone is mapped to a set of DNS resolvers.

A sample definition is as follows:
```
dns_forward_zones:
- - "a.contoso.com"
  - - - "1.1.1.1"
      - 53
    - - "2.2.2.2"
      - 53
- - "b.contoso.com"
  - - - "3.3.3.3"
      - 53
    - - "4.4.4.4"
      - 53
```

In the above example, a DNS query to `myapp.a.contoso.com` will be directed to `1.1.1.1:53` or `2.2.2.2:53`. Likewise, a DNS query to `myapp.b.contoso.com` will be directed to `3.3.3.3:53` or `4.4.4.4:53`.

### use_proxy

This parameter specifies whether to enable the DC/OS proxy. 

*  `use_proxy: 'false'` Do not configure DC/OS [components](/docs/1.10/overview/architecture/components/) to use a custom proxy. This is the default value.
*  `use_proxy: 'true'` Configure DC/OS [components](/docs/1.10/overview/architecture/components/) to use a custom proxy. If you specify `use_proxy: 'true'`, you can also specify these parameters:
    **Important:** The specified proxies must be resolvable from the provided list of [resolvers](#resolvers).
    *  `http_proxy: http://<user>:<pass>@<proxy_host>:<http_proxy_port>` This parameter specifies the HTTP proxy.
    *  `https_proxy: https://<user>:<pass>@<proxy_host>:<https_proxy_port>` This parameter specifies the HTTPS proxy.
    *  `no_proxy: - .<(sub)domain>` This parameter specifies YAML nested list (-) of addresses to exclude from the proxy.

For more information, see the [examples](#http-proxy).

**Important:** You should also configure an HTTP proxy for [Docker](https://docs.docker.com/engine/admin/systemd/#/http-proxy). 

## <a name="performance-and-tuning"></a>Performance and Tuning

### <a name="check-time"></a>check_time
This parameter specifies whether to check if Network Time Protocol (NTP) is enabled during DC/OS startup. It recommended that NTP is enabled for a production environment.

*  `check_time: 'true'` Check if NTP is enabled during startup. You will receive an error if this is not enabled. This is the default value.
*  `check_time: 'false'` Do not check if NTP is enabled during startup.

### <a name="docker-remove"></a>docker_remove_delay
This parameter specifies the amount of time to wait before removing stale Docker images stored on the agent nodes and the Docker image generated by the installer. It is recommended that you accept the default value 1 hour.

### <a name="enable-docker-gc"></a>enable_docker_gc
This parameter specifies whether to run the [docker-gc](https://github.com/spotify/docker-gc#excluding-images-from-garbage-collection) script, a simple Docker container and image garbage collection script, once every hour to clean up stray Docker containers. You can configure the runtime behavior by using the `/etc/` config. For more information, see the [documentation](https://github.com/spotify/docker-gc#excluding-images-from-garbage-collection)

*  `enable_docker_gc: 'true'` Run the docker-gc scripts once every hour. This is the default value for [cloud](/docs/1.10/installing/cloud/) template installations.
*  `enable_docker_gc: 'false'` Do not run the docker-gc scripts once every hour. This is the default value for [custom](/docs/1.10/installing/custom/) installations.

### <a name="gc-delay"></a>gc_delay
This parameter specifies the maximum amount of time to wait before cleaning up the executor directories. It is recommended that you accept the default value of 2 days.

### <a name="log_directory"></a>log_directory
This parameter specifies the path to the installer host logs from the SSH processes. By default this is set to `/genconf/logs`. In most cases this should not be changed because `/genconf` is local to the container that is running the installer, and is a mounted volume.

### <a name="process_timeout"></a>process_timeout
This parameter specifies the allowable amount of time, in seconds, for an action to begin after the process forks. This parameter is not the complete process time. The default value is 120 seconds.

**Tip:** If have a slower network environment, consider changing to `process_timeout: 600`.

## <a name="security-and-authentication"></a>Security And Authentication

### oauth_enabled
This parameter specifies whether to enable authentication for your cluster. <!-- DC/OS auth -->

- `oauth_enabled: 'true'` Enable authentication for your cluster. This is the default value.
- `oauth_enabled: 'false'` Disable authentication for your cluster.

If you’ve already installed your cluster and would like to disable this in-place, you can go through an upgrade with the same parameter set.

### telemetry_enabled
This parameter specifies whether to enable sharing of anonymous data for your cluster. <!-- DC/OS auth -->

- `telemetry_enabled: 'true'` Enable anonymous data sharing. This is the default value.
- `telemetry_enabled: 'false'` Disable anonymous data sharing.

If you’ve already installed your cluster and would like to disable this in-place, you can go through an [upgrade][3] with the same parameter set.

# <a name="examples1"></a>Example Configurations

#### DC/OS cluster with three masters, five private agents, and Exhibitor/ZooKeeper managed internally.

```yaml
---
agent_list:
- <agent-private-ip-1>
- <agent-private-ip-2>
- <agent-private-ip-3>
- <agent-private-ip-4>
- <agent-private-ip-5>
bootstrap_url: 'file:///opt/dcos_install_tmp'
cluster_name: '<cluster-name>'
log_directory: /genconf/logs
master_discovery: static
master_list:
- <master-private-ip-1>
- <master-private-ip-2>
- <master-private-ip-3>
process_timeout: 120
resolvers:
- <dns-resolver-1>
- <dns-resolver-2>
ssh_key_path: /genconf/ssh-key
ssh_port: '<port-number>'
ssh_user: <username>
```

#### <a name="aws"></a>DC/OS cluster with three masters, an Exhibitor/ZooKeeper backed by an AWS S3 bucket, AWS DNS, five private agents, and one public agent node

```yaml
---
agent_list:
- <agent-private-ip-1>
- <agent-private-ip-2>
- <agent-private-ip-3>
- <agent-private-ip-4>
- <agent-private-ip-5>
aws_access_key_id: AKIAIOSFODNN7EXAMPLE
aws_region: us-west-2
aws_secret_access_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
bootstrap_url: file:///tmp/dcos
cluster_name: s3-example
exhibitor_storage_backend: aws_s3
exhibitor_explicit_keys: 'true'
log_directory: /genconf/logs
master_discovery: static
master_list:
- <master-private-ip-1>
- <master-private-ip-2>
- <master-private-ip-3>
process_timeout: 120
resolvers:
- <dns-resolver-1>
- <dns-resolver-2>
s3_bucket: mybucket
s3_prefix: s3-example
ssh_key_path: /genconf/ssh-key
ssh_port: '<port-number>'
ssh_user: <username>
```

#### <a name="zk"></a>DC/OS cluster with three masters, an Exhibitor/ZooKeeper backed by ZooKeeper, masters that have an HTTP load balancer in front of them, one public agent node, five private agents, and Google DNS

```yaml
---
agent_list:
- <agent-private-ip-1>
- <agent-private-ip-2>
- <agent-private-ip-3>
- <agent-private-ip-4>
- <agent-private-ip-5>
bootstrap_url: file:///tmp/dcos
cluster_name: zk-example
exhibitor_storage_backend: zookeeper
exhibitor_zk_hosts: 10.0.0.1:2181, 10.0.0.2:2181, 10.0.0.3:2181
exhibitor_zk_path: /zk-example
log_directory: /genconf/logs
master_discovery: master_http_loadbalancer
num_masters: 3
public_agent_list:
- 10.10.0.139
exhibitor_address: 67.34.242.55
process_timeout: 120
resolvers:
- <dns-resolver-1>
- <dns-resolver-2>
ssh_key_path: /genconf/ssh-key
ssh_port: '<port-number>'
ssh_user: <username>
```

#### <a name="overlay"></a>DC/OS cluster with three masters, an Exhibitor/ZooKeeper managed internally, two DC/OS virtual networks, two private agents, and Google DNS

```yaml
    agent_list:
    - <agent-private-ip-1>
    - <agent-private-ip-2>
    # Use this bootstrap_url value unless you have moved the DC/OS installer assets.
    bootstrap_url: file:///opt/dcos_install_tmp
    cluster_name: <cluster-name>
    master_discovery: static
    master_list:
    - <master-private-ip-1>
    - <master-private-ip-2>
    - <master-private-ip-3>
    resolvers:
    # You probably do not want to use these values since they point to public DNS servers.
    # Instead use values that are more specific to your particular infrastructure.
    - 8.8.4.4
    - 8.8.8.8
    ssh_port: 22
    ssh_user: centos
    dcos_overlay_enable: true
    dcos_overlay_mtu: 9001
    dcos_overlay_config_attempts: 6
    dcos_overlay_network:
      vtep_subnet: 44.128.0.0/20
      vtep_mac_oui: 70:B3:D5:00:00:00
      overlays:
        - name: dcos
          subnet: 9.0.0.0/8
          prefix: 26
        - name: dcos-1
          subnet: 192.168.0.0/16
          prefix: 24
```

#### <a name="http-proxy"></a>DC/OS cluster with three masters, an Exhibitor/ZooKeeper managed internally, a custom HTTP proxy, two private agents, and Google DNS

```yaml
    agent_list:
    - <agent-private-ip-1>
    - <agent-private-ip-2>
    # Use this bootstrap_url value unless you have moved the DC/OS installer assets.
    bootstrap_url: file:///opt/dcos_install_tmp
    cluster_name: <cluster-name>
    master_discovery: static
    master_list:
    - <master-private-ip-1>
    - <master-private-ip-2>
    - <master-private-ip-3>
    resolvers:
    # You probably do not want to use these values since they point to public DNS servers.
    # Instead use values that are more specific to your particular infrastructure.
    - 8.8.4.4
    - 8.8.8.8
    ssh_port: 22
    ssh_user: centos
    use_proxy: 'true'
    http_proxy: http://<proxy_host>:<http_proxy_port>
    https_proxy: https://<proxy_host>:<https_proxy_port>
    no_proxy:
    - 'foo.bar.com'
    - '.baz.com'
```

#### <a name="docker-credentials"></a>DC/OS cluster with three masters, an Exhibitor/ZooKeeper managed internally, custom Docker credentials, two private agents, and Google DNS

```yaml
    agent_list:
    - <agent-private-ip-1>
    - <agent-private-ip-2>
    # Use this bootstrap_url value unless you have moved the DC/OS installer assets.
    bootstrap_url: file:///opt/dcos_install_tmp
    cluster_docker_credentials:
      auths:
        'https://registry.example.com/v1/':
          auth: foo
          email: user@example.com
    cluster_docker_credentials_dcos_owned: false
    cluster_docker_registry_url: https://registry.example.com
    cluster_name: <cluster-name>
    master_discovery: static
    master_list:
    - <master-private-ip-1>
    - <master-private-ip-2>
    - <master-private-ip-3>
    resolvers:
    # You probably do not want to use these values since they point to public DNS servers.
    # Instead use values that are more specific to your particular infrastructure.
    - 8.8.4.4
    - 8.8.8.8
    ssh_port: 22
    ssh_user: centos
```

#### <a name="cosmos-config"></a>DC/OS cluster with one master, an Exhibitor/ZooKeeper managed internally, three private agents, Google DNS, and the package manager (Cosmos) configured with persistent storage.

```yaml
    agent_list:
    - <agent-private-ip-1>
    - <agent-private-ip-2>
    - <agent-private-ip-3>
    # Use this bootstrap_url value unless you have moved the DC/OS installer assets.
    bootstrap_url: file:///opt/dcos_install_tmp
    cluster_name: <cluster-name>
    master_discovery: static
    master_list:
    - <master-private-ip-1>
    resolvers:
    # You probably do not want to use these values since they point to public DNS servers.
    # Instead use values that are more specific to your particular infrastructure.
    - 8.8.4.4
    - 8.8.8.8
    ssh_port: 22
    ssh_user: centos
    cosmos_config:
      staged_package_storage_uri: file:///var/lib/dcos/cosmos/staged-packages
      package_storage_uri: file:///var/lib/dcos/cosmos/packages
```

 [1]: https://en.wikipedia.org/wiki/YAML
