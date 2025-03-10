# Bottlerocket OS

Welcome to Bottlerocket!

Bottlerocket is a free and open-source Linux-based operating system meant for hosting containers.

To learn more about Bottlerocket, visit the [official Bottlerocket website and documentation](https://bottlerocket.dev/).
Otherwise, if you’re ready to jump right in, read one of our setup guides for running Bottlerocket in [Amazon EKS](QUICKSTART-EKS.md), [Amazon ECS](QUICKSTART-ECS.md), or [VMware](QUICKSTART-VMWARE.md).
If you're interested in running Bottlerocket on bare metal servers, please refer to the [provisioning guide](PROVISIONING-METAL.md) to get started.

Bottlerocket focuses on security and maintainability, providing a reliable, consistent, and safe platform for container-based workloads.
This is a reflection of what we've learned building operating systems and services at Amazon.
You can read more about what drives us in [our charter](CHARTER.md).

The base operating system has just what you need to run containers reliably, and is built with standard open-source components.
Bottlerocket-specific additions focus on reliable updates and on the API.
Instead of making configuration changes manually, you can change settings with an API call, and these changes are automatically migrated through updates.

Some notable features include:

* [API access](#api) for configuring your system, with secure out-of-band [access methods](#exploration) when you need them.
* [Updates](#updates) based on partition flips, for fast and reliable system updates.
* [Modeled configuration](#settings) that's automatically migrated through updates.
* [Security](#security) as a top priority.

## Participate in the Community

There are many ways to take part in the Bottlerocket community:

- [Join us on Meetup](https://www.meetup.com/bottlerocket-community/) to hear about the latest Bottlerocket (virtual/in-person) events and community meetings.
  Community meetings are typically every other week.

  Details can be found under the [Events section on Meetup](https://www.meetup.com/bottlerocket-community/events/), and you will receive email notifications if you become a member of the Meetup group. (It's free to join!)

- [Start or join a discussion](https://github.com/bottlerocket-os/bottlerocket/discussions) if you have questions about Bottlerocket.
- If you're interested in contributing, thank you!
  Please see our [contributor's guide](CONTRIBUTING.md).

## Contact us

If you find a security issue, please [contact our security team](https://github.com/bottlerocket-os/bottlerocket/security/policy) rather than opening an issue.

We use GitHub issues to track other bug reports and feature requests.
You can look at [existing issues](https://github.com/bottlerocket-os/bottlerocket/issues) to see whether your concern is already known.

If not, you can select from a few templates and get some guidance on the type of information that would be most helpful.
[Contact us with a new issue here.](https://github.com/bottlerocket-os/bottlerocket/issues/new/choose)

We don't have other communication channels set up quite yet, but don't worry about making an issue or a discussion thread!
You can let us know about things that seem difficult, or even ways you might like to help.

## Variants

To start, we're focusing on the use of Bottlerocket as a host OS in AWS EKS Kubernetes clusters and Amazon ECS clusters.
We’re excited to get early feedback and to continue working on more use cases!

Bottlerocket is architected such that different cloud environments and container orchestrators can be supported in the future.
A build of Bottlerocket that supports different features or integration characteristics is known as a 'variant'.
The artifacts of a build will include the architecture and variant name.
For example, an `x86_64` build of the `aws-k8s-1.24` variant will produce an image named `bottlerocket-aws-k8s-1.24-x86_64-<version>-<commit>.img`.

The following variants support EKS, as described above:

* `aws-k8s-1.23`
* `aws-k8s-1.24`
* `aws-k8s-1.25`
* `aws-k8s-1.26`
* `aws-k8s-1.27`
* `aws-k8s-1.23-nvidia`
* `aws-k8s-1.24-nvidia`
* `aws-k8s-1.25-nvidia`
* `aws-k8s-1.26-nvidia`
* `aws-k8s-1.27-nvidia`

The following variants support ECS:

* `aws-ecs-1`
* `aws-ecs-1-nvidia`

We also have variants that are designed to be Kubernetes worker nodes in VMware:

* `vmware-k8s-1.23`
* `vmware-k8s-1.24`
* `vmware-k8s-1.25`
* `vmware-k8s-1.26`
* `vmware-k8s-1.27`

The following variants are designed to be Kubernetes worker nodes on bare metal:

* `metal-k8s-1.23`
* `metal-k8s-1.24`
* `metal-k8s-1.25`
* `metal-k8s-1.26`
* `metal-k8s-1.27`

The following variants are no longer supported:

* All Kubernetes variants using Kubernetes 1.22 and earlier

We recommend users replace nodes running these variants with the [latest variant compatible with their cluster](variants/).

## Architectures

Our supported architectures include `x86_64` and `aarch64` (written as `arm64` in some contexts).

## Setup

:walking: :running:

Bottlerocket is best used with a container orchestrator.
To get started with Kubernetes in Amazon EKS, please see [QUICKSTART-EKS](QUICKSTART-EKS.md).
To get started with Kubernetes in VMware, please see [QUICKSTART-VMWARE](QUICKSTART-VMWARE.md).
To get started with Amazon ECS, please see [QUICKSTART-ECS](QUICKSTART-ECS.md).
These guides describe:

* how to set up a cluster with the orchestrator, so your Bottlerocket instance can run containers
* how to launch a Bottlerocket instance in EC2 or VMware

To see how to provision Bottlerocket on bare metal, see [PROVISIONING-METAL](PROVISIONING-METAL.md).

To build your own Bottlerocket images, please see [BUILDING](BUILDING.md).
It describes:

* how to build an image
* how to register an EC2 AMI from an image

To publish your built Bottlerocket images, please see [PUBLISHING](PUBLISHING.md).
It describes:

* how to make TUF repos including your image
* how to copy your AMI across regions
* how to mark your AMIs public or grant access to specific accounts
* how to make your AMIs discoverable using [SSM parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)

## Exploration

To improve security, there's no SSH server in a Bottlerocket image, and not even a shell.

Don't panic!

There are a couple out-of-band access methods you can use to explore Bottlerocket like you would a typical Linux system.
Either option will give you a shell within Bottlerocket.
From there, you can [change settings](#settings), manually [update Bottlerocket](#updates), debug problems, and generally explore.

**Note:** These methods require that your instance has permission to access the ECR repository where these containers live; the appropriate policy to add to your instance's IAM role is `AmazonEC2ContainerRegistryReadOnly`.

### Control container

Bottlerocket has a ["control" container](https://github.com/bottlerocket-os/bottlerocket-control-container), enabled by default, that runs outside of the orchestrator in a separate instance of containerd.
This container runs the [AWS SSM agent](https://github.com/aws/amazon-ssm-agent) that lets you run commands, or start shell sessions, on Bottlerocket instances in EC2.
(You can easily replace this control container with your own just by changing the URI; see [Settings](#settings).)

In AWS, you need to give your instance the SSM role for this to work; see the [setup guide](QUICKSTART-EKS.md#enabling-ssm).
Outside of AWS, you can use [AWS Systems Manager for hybrid environments](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-managedinstances.html).
There's more detail about hybrid environments in the [control container documentation](https://github.com/bottlerocket-os/bottlerocket-control-container/#connecting-to-aws-systems-manager-ssm).

Once the instance is started, you can start a session:

* Go to AWS SSM's [Session Manager](https://console.aws.amazon.com/systems-manager/session-manager/sessions)
* Select "Start session" and choose your Bottlerocket instance
* Select "Start session" again to get a shell

If you prefer a command-line tool, you can start a session with a recent [AWS CLI](https://aws.amazon.com/cli/) and the [session-manager-plugin](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html).
Then you'd be able to start a session using only your instance ID, like this:

```shell
aws ssm start-session --target INSTANCE_ID
```

With the [default control container](https://github.com/bottlerocket-os/bottlerocket-control-container), you can make [API calls](#api) to configure and manage your Bottlerocket host.
To do even more, read the next section about the [admin container](#admin-container).
You can access the admin container from the control container like this:

```shell
enter-admin-container
```

### Admin container

Bottlerocket has an [administrative container](https://github.com/bottlerocket-os/bottlerocket-admin-container), disabled by default, that runs outside of the orchestrator in a separate instance of containerd.
This container has an SSH server that lets you log in as `ec2-user` using your EC2-registered SSH key.
Outside of AWS, you can [pass in your own SSH keys](https://github.com/bottlerocket-os/bottlerocket-admin-container#authenticating-with-the-admin-container).
(You can easily replace this admin container with your own just by changing the URI; see [Settings](#settings).

To enable the container, you can change the setting in user data when starting Bottlerocket, for example EC2 instance user data:

```toml
[settings.host-containers.admin]
enabled = true
```

If Bottlerocket is already running, you can enable the admin container from the default [control container](#control-container) like this:

```shell
enable-admin-container
```

Or you can start an interactive session immediately like this:

```shell
enter-admin-container
```

If you're using a custom control container, or want to make the API calls directly, you can enable the admin container like this instead:

```shell
apiclient set host-containers.admin.enabled=true
```

Once you've enabled the admin container, you can either access it through SSH or execute commands from the control container like this:

```shell
apiclient exec admin bash
```

Once you're in the admin container, you can run `sheltie` to get a full root shell in the Bottlerocket host.
Be careful; while you can inspect and change even more as root, Bottlerocket's filesystem and dm-verity setup will prevent most changes from persisting over a restart - see [Security](#security).

## Updates

Rather than a package manager that updates individual pieces of software, Bottlerocket downloads a full filesystem image and reboots into it.
It can automatically roll back if boot failures occur, and workload failures can trigger manual rollbacks.

The update process uses images secured by [TUF](https://theupdateframework.github.io/).
For more details, see the [update system documentation](sources/updater/).

### Update methods

There are several ways of updating your Bottlerocket hosts.
We provide tools for automatically updating hosts, as well as an API for direct control of updates.

#### Automated updates

For EKS variants of Bottlerocket, we recommend using the [Bottlerocket update operator](https://github.com/bottlerocket-os/bottlerocket-update-operator) for automated updates.

For the ECS variant of Bottlerocket, we recommend using the [Bottlerocket ECS updater](https://github.com/bottlerocket-os/bottlerocket-ecs-updater/) for automated updates.

#### Update API

The [Bottlerocket API](#api) includes methods for checking and starting system updates.
You can read more about the update APIs in our [update system documentation](sources/updater/README.md#update-api).

apiclient knows how to handle those update APIs for you, and you can run it from the [control](#control-container) or [admin](#admin-container) containers.

To see what updates are available:

```shell
apiclient update check
```

If an update is available, it will show up in the `chosen_update` field.
The `available_updates` field will show the full list of available versions, including older versions, because Bottlerocket supports safely rolling back.

To apply the latest update:

```shell
apiclient update apply
```

The next time you reboot, you'll start up in the new version, and system configuration will be automatically [migrated](sources/api/migration/).
To reboot right away:

```shell
apiclient reboot
```

If you're confident about updating, the `apiclient update apply` command has `--check` and `--reboot` flags to combine the above actions, so you can accomplish all of the above steps like this:

```shell
apiclient update apply --check --reboot
```

See the [apiclient documentation](sources/api/apiclient/) for more details.

### Update rollback

The system will automatically roll back if it's unable to boot.
If the update is not functional for a given container workload, you can do a manual rollback:

```shell
signpost rollback-to-inactive
reboot
```

This doesn't require any external communication, so it's quicker than `apiclient`, and it's made to be as reliable as possible.

## Settings

Here we'll describe the settings you can configure on your Bottlerocket instance, and how to do it.

(API endpoints are defined in our [OpenAPI spec](sources/api/openapi.yaml) if you want more detail.)

### Interacting with settings

#### Using the API client

You can see the current settings with an API request:

```shell
apiclient get settings
```

This will return all of the current settings in JSON format.
For example, here's an abbreviated response:

```json
{"motd": "...", {"kubernetes": {}}}
```

You can change settings like this:

```shell
apiclient set motd="hi there" kubernetes.node-labels.environment=test
```

You can also use a JSON input mode to help change many related settings at once, and a "raw" mode if you want more control over how the settings are committed and applied to the system.
See the [apiclient README](sources/api/apiclient/) for details.

#### Using user data

If you know what settings you want to change when you start your Bottlerocket instance, you can send them in the user data.

In user data, we structure the settings in TOML form to make things a bit simpler.
Here's the user data to change the message of the day setting, as we did in the last section:

```toml
[settings]
motd = "my own value!"
```

If your user data is over the size limit of the platform (e.g. 16KiB for EC2) you can compress the contents with gzip.
(With [aws-cli](https://aws.amazon.com/cli/), you can use `--user-data fileb:///path/to/gz-file` to pass binary data.)

### Description of settings

Here we'll describe each setting you can change.

**Note:** You can see the default values (for any settings that are not generated at runtime) by looking in the `defaults.d` directory for a variant, for example [aws-ecs-1](sources/models/src/aws-ecs-1/defaults.d/).

When you're sending settings to the API, or receiving settings from the API, they're in a structured JSON format.
This allows modification of any number of keys at once.
It also lets us ensure that they fit the definition of the Bottlerocket data model - requests with invalid settings won't even parse correctly, helping ensure safety.

Here, however, we'll use the shortcut "dotted key" syntax for referring to keys.
This is used in some API endpoints with less-structured requests or responses.
It's also more compact for our needs here.

In this format, "settings.kubernetes.cluster-name" refers to the same key as in the JSON `{"settings": {"kubernetes": {"cluster-name": "value"}}}`.

#### Top-level settings

* `settings.motd`: This setting is just written out to /etc/motd. It's useful as a way to get familiar with the API! Try changing it.

#### Kubernetes settings

See the [EKS setup guide](QUICKSTART-EKS.md) for much more detail on setting up Bottlerocket and Kubernetes in AWS EKS.
For more details about running Bottlerocket as a Kubernetes worker node in VMware, see the [VMware setup guide](QUICKSTART-VMWARE.md).

The following settings must be specified in order to join a Kubernetes cluster.
You should [specify them in user data](#using-user-data).

* `settings.kubernetes.api-server`: This is the cluster's Kubernetes API endpoint.
* `settings.kubernetes.cluster-certificate`: This is the base64-encoded certificate authority of the cluster.

For Kubernetes variants in AWS, you must also specify:

* `settings.kubernetes.cluster-name`: The cluster name you chose during setup; the [setup guide](QUICKSTART-EKS.md) uses "bottlerocket".

For Kubernetes variants in VMware, you must specify:

* `settings.kubernetes.bootstrap-token`: The token used for [TLS bootstrapping](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/).

The following settings can be optionally set to customize the node labels and taints. Remember to quote keys (since they often contain ".") and to quote all values.

* `settings.kubernetes.cluster-dns-ip`: The IP of the DNS service running in the cluster.

  This value can be set as a string containing a single IP address, or as a list containing multiple IP addresses.

  Examples:

  ```toml
  # Valid, single IP
  [settings.kubernetes]
  "cluster-dns-ip" = "10.0.0.1"

  # Also valid, multiple nameserver IPs
  [settings.kubernetes]
  "cluster-dns-ip" = ["10.0.0.1", "10.0.0.2"]
  ```

* `settings.kubernetes.node-labels`: [Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) in the form of key, value pairs added when registering the node in the cluster.
* `settings.kubernetes.node-taints`: [Taints](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) in the form of key, values and effects entries added when registering the node in the cluster.

  Example user data for setting up labels and taints:

  ```toml
  [settings.kubernetes.node-labels]
  "label1" = "foo"
  "label2" = "bar"
  [settings.kubernetes.node-taints]
  "dedicated" = ["experimental:PreferNoSchedule", "experimental:NoExecute"]
  "special" = ["true:NoSchedule"]
  ```

The following settings are optional and allow you to further configure your cluster.

* `settings.kubernetes.allowed-unsafe-sysctls`: Enables specified list of unsafe sysctls.

  Example user data for setting up allowed unsafe sysctls:

  ```toml
  allowed-unsafe-sysctls = ["net.core.somaxconn", "net.ipv4.ip_local_port_range"]
  ```

* `settings.kubernetes.authentication-mode`: Which authentication method the kubelet should use to connect to the API server, and for incoming requests. Defaults to `aws` for AWS variants, and `tls` for other variants.
* `settings.kubernetes.bootstrap-token`: The token to use for [TLS bootstrapping](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/). This is only used with the `tls` authentication mode, and is otherwise ignored.
* `settings.kubernetes.cloud-provider`: The cloud provider for this cluster. Defaults to `aws` for AWS variants, and `external` for other variants.
* `settings.kubernetes.cluster-domain`: The DNS domain for this cluster, allowing all Kubernetes-run containers to search this domain before the host's search domains. Defaults to `cluster.local`.
* `settings.kubernetes.container-log-max-files`: The maximum number of container log files that can be present for a container.
* `settings.kubernetes.container-log-max-size`: The maximum size of container log file before it is rotated.
* `settings.kubernetes.cpu-cfs-quota-enforced`: Whether CPU CFS quotas are enforced. Defaults to `true`.
* `settings.kubernetes.cpu-manager-policy`: Specifies the CPU manager policy. Possible values are `static` and `none`. Defaults to `none`. If you want to allow pods with certain resource characteristics to be granted increased CPU affinity and exclusivity on the node, you can set this setting to `static`. You should reboot if you change this setting after startup - try `apiclient reboot`.
* `settings.kubernetes.cpu-manager-policy-options`: Policy options to apply when `cpu-manager-policy` is set to `static`. Currently `full-pcpus-only` is the only option.

  For example:

  ```toml
  [settings.kubernetes]
  cpu-manager-policy = "static"
  cpu-manager-policy-options = [
    "full-pcpus-only"
  ]
  ```

* `settings.kubernetes.cpu-manager-reconcile-period`: Specifies the CPU manager reconcile period, which controls how often updated CPU assignments are written to cgroupfs. The value is a duration like `30s` for 30 seconds or `1h5m` for 1 hour and 5 minutes.
* `settings.kubernetes.credential-providers`: Contains a collection of Kubelet image credential provider settings.
  Each name under `credential-providers` is the name of the plugin to configure.

  Example user data for configuring the `ecr-credential-provider` credential provider plugin:

  ```toml
  [settings.kubernetes.credential-providers.ecr-credential-provider]
  enabled = true
  # (optional - defaults to "12h")
  cache-duration = "30m"
  image-patterns = [
    # One or more URL paths to match an image prefix. Supports globbing of subdomains.
    "*.dkr.ecr.us-east-2.amazonaws.com",
    "*.dkr.ecr.us-west-2.amazonaws.com"
  ]

  [settings.kubernetes.credential-providers.ecr-credential-provider.environment]
  # The following are not used with ecr-credential-provider, but are provided for illustration
  "KEY" = "abc123xyz"
  "GOMAXPROCS" = "2"
  ```

  **Note:** `ecr-credential-provider` is currently the only supported provider.
  To manage its AWS credentials, see the `settings.aws.config` and `settings.aws.credentials` settings.

  The `ecr-credential-provider` plugin can also be used for AWS IAM Roles Anywhere support.
  IAM Roles Anywhere is configured using the `settings.aws.config` setting.
  The content of that setting needs to configure the `credential_process` using the `aws_signing_helper` using your IAM Roles Anywhere settings, similar to the following:

  ```ini
  [default]
  region = us-west-2
  credential_process = aws_signing_helper credential-process \
     --certificate /var/lib/kubelet/pki/kubelet-client-current.pem \
     --private-key /var/lib/kubelet/pki/kubelet-client-current.pem \
     --profile-arn [profile ARN]
     --role-arn [role ARN]
     --trust-anchor-arn [trust anchor ARN]
  ```

  See the [Roles Anywhere documentation](https://docs.aws.amazon.com/rolesanywhere/latest/userguide/credential-helper.html) for more details on the `aws_signing_helper` arguments.

* `settings.kubernetes.event-burst`: The maximum size of a burst of event creations.
* `settings.kubernetes.event-qps`: The maximum event creations per second.
* `settings.kubernetes.eviction-hard`: The signals and thresholds that trigger pod eviction.
* `settings.kubernetes.eviction-max-pod-grace-period`: Maximum grace period, in seconds, to wait for pod termination before soft eviction. Default is `0`.
* `settings.kubernetes.eviction-soft`: The signals and thresholds that trigger pod eviction with a provided grace period.
* `settings.kubernetes.eviction-soft-grace-period`: Delay for each signal to wait for pod termination before eviction.

  Remember to quote signals (since they all contain ".") and to quote all values.

  Example user data for setting up eviction values:

  ```toml
  [settings.kubernetes.eviction-hard]
  "memory.available" = "15%"

  [settings.kubernetes.eviction-soft]
  "memory.available" = "12%"

  [settings.kubernetes.eviction-soft-grace-period]
  "memory.available" = "30s"

  [settings.kubernetes]
  "eviction-max-pod-grace-period" = 40
  ```

* `settings.kubernetes.image-gc-high-threshold-percent`: The percent of disk usage after which image garbage collection is always run, expressed as an integer from 0-100 inclusive.
* `settings.kubernetes.image-gc-low-threshold-percent`: The percent of disk usage before which image garbage collection is never run, expressed as an integer from 0-100 inclusive.

  Since v1.14.0 `image-gc-high-threshold-percent` and `image-gc-low-threshold-percent` can be represented as numbers.
  For example:

  ```toml
  [settings.kubernetes]
  image-gc-high-threshold-percent = 85
  image-gc-low-threshold-percent = 80
  ```

  For backward compatibility, both string and numeric representations are accepted since v1.14.0.
  Prior to v1.14.0 these needed to be represented as strings, for example:

  ```toml
  [settings.kubernetes]
  image-gc-high-threshold-percent = "85"
  image-gc-low-threshold-percent = "80"
  ```

  If you downgrade from v1.14.0 to an earlier version, and you have these values set as numbers, they will be converted to strings on downgrade.

* `settings.kubernetes.kube-api-burst`: The burst to allow while talking with kubernetes.
* `settings.kubernetes.kube-api-qps`: The QPS to use while talking with kubernetes apiserver.
* `settings.kubernetes.log-level`: Adjust the logging verbosity of the `kubelet` process.
  The default log level is 2, with higher numbers enabling more verbose logging.
* `settings.kubernetes.memory-manager-policy`: The memory management policy to use: `None` (default) or `Static`.
  Note, when using the `Static` policy you should also set `settings.kubernetes.memory-manager-reserved-memory` values.
* `settings.kubernetes.memory-manager-reserved-memory`: Used to set the total amount of reserved memory for a node.
  These settings are used to configure memory manager policy when `settings.kubernetes.memory-manager-policy` is set to `Static`.

  `memory-manager-reserved-memory` is set per NUMA node. For example:

  ```toml
  [settings.kubernetes]
  "memory-manager-policy" = "Static"

  [settings.kubernetes.memory-manager-reserved-memory.0]
  # Reserve a single 1GiB huge page along with 674MiB of memory
  "enabled" = true
  "memory" = "674Mi"
  "hugepages-1Gi" = "1Gi"

  [settings.kubernetes.memory-manager-reserved-memory.1]
  # Reserve 1,074 2MiB huge pages
  "enabled" = true
  "hugepages-2Mi" = "2148Mi"
  ```

  **Warning:** `memory-manager-reserved-memory` settings are an advanced configuration and requires a clear understanding of what you are setting.
  Misconfiguration of reserved memory settings may cause the Kubernetes `kubelet` process to fail.
  It can be very difficult to recover from configuration errors.
  Use the memory reservation information from `kubectl describe node` and make sure you understand the Kubernetes documentation related to the [memory manager](https://kubernetes.io/docs/tasks/administer-cluster/memory-manager/) and how to [reserve compute resources for system daemons](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/).

* `settings.kubernetes.pod-pids-limit`: The maximum number of processes per pod.
* `settings.kubernetes.provider-id`: This sets the unique ID of the instance that an external provider (i.e. cloudprovider) can use to identify a specific node.
* `settings.kubernetes.registry-burst`: The maximum size of bursty pulls.
* `settings.kubernetes.registry-qps`: The registry pull QPS.
* `settings.kubernetes.server-tls-bootstrap`: Enables or disables server certificate bootstrap. When enabled, the kubelet will request a certificate from the certificates.k8s.io API. This requires an approver to approve the certificate signing requests (CSR). Defaults to `true`.
* `settings.kubernetes.shutdown-grace-period`: Delay the node should wait for pod termination before shutdown. Default is `0s`.
* `settings.kubernetes.shutdown-grace-period-for-critical-pods`: The portion of the shutdown delay that should be dedicated to critical pod shutdown. Default is `0s`.
* `settings.kubernetes.standalone-mode`: Whether to run the kubelet in standalone mode, without connecting to an API server. Defaults to `false`.
* `settings.kubernetes.system-reserved`: Resources reserved for system components.

  Example user data for setting up system reserved:

  ```toml
  [settings.kubernetes.system-reserved]
  cpu = "10m"
  memory = "100Mi"
  ephemeral-storage= "1Gi"
  ```

* `settings.kubernetes.server-certificate`: The base64 encoded content of an x509 certificate for the Kubelet web server, which is used for retrieving logs and executing commands.
* `settings.kubernetes.server-key`: The base64 encoded content of an x509 private key for the Kubelet web server.
* `settings.kubernetes.topology-manager-policy`: Specifies the topology manager policy. Possible values are `none`, `restricted`, `best-effort`, and `single-numa-node`. Defaults to `none`.
* `settings.kubernetes.topology-manager-scope`: Specifies the topology manager scope. Possible values are `container` and `pod`. Defaults to `container`. If you want to group all containers in a pod to a common set of NUMA nodes, you can set this setting to `pod`.

You can also optionally specify static pods for your node with the following settings.
Static pods can be particularly useful when running in standalone mode.

* `settings.kubernetes.static-pods.<custom identifier>.enabled`: Whether the static pod is enabled.
* `settings.kubernetes.static-pods.<custom identifier>.manifest`: A base64-encoded pod manifest.

For Kubernetes variants in AWS and VMware, the following are set for you automatically, but you can override them if you know what you're doing!
In AWS, [pluto](sources/api/) sets these based on runtime instance information.
In VMware and on bare metal, Bottlerocket uses [netdog](sources/api/) (for `node-ip`) or relies on default values.
(See the [VMware defaults](sources/models/src/vmware-k8s-1.23/defaults.d) or [bare metal defaults](sources/models/src/metal-k8s-1.23/defaults.d)).

* `settings.kubernetes.kube-reserved`: Resources reserved for node components.

  Bottlerocket provides default values for the resources by [schnauzer](sources/api/):

  * `cpu`: in millicores from the total number of vCPUs available on the instance.
  * `memory`: in mebibytes from the max num of pods on the instance. `memory_to_reserve = max_num_pods * 11 + 255`.
  * `ephemeral-storage`: defaults to `1Gi`.

* `settings.kubernetes.node-ip`: The IP address of this node.
* `settings.kubernetes.pod-infra-container-image`: The URI of the "pause" container.

For Kubernetes variants in AWS, the following settings are set for you automatically by [pluto](sources/api/).

* `settings.kubernetes.cluster-dns-ip`: Derived from the EKS Service IP CIDR or the CIDR block of the primary network interface.
* `settings.kubernetes.max-pods`: The maximum number of pods that can be scheduled on this node (limited by number of available IPv4 addresses)
* `settings.kubernetes.hostname-override`: The node name kubelet uses as identification instead of the hostname or the name determined by the in-tree cloud provider if that's enabled.

  **Important note for all Kubernetes variants:** Changing this setting at runtime (not via user-data) can cause issues with kubelet registration, as hostname is closely tied to the identity of the system for both registration and certificates/authorization purposes.

  Most users don't need to change this setting.
  If left unset, the system hostname will be used instead.
  The `settings.network.hostname` setting can be used to specify the value for both `kubelet` and the host.
  Only set this override if you intend for the `kubelet` to register with a different name than the host.

  For `aws-k8s-1.26` variants, which use the "external" cloud provider, a hostname override will be automatically generated by querying the EC2 API for the private DNS name of the instance.
  This is done for backwards compatibility with the deprecated "aws" cloud provider, which adjusted the hostname in a similar way.
  Future `aws-k8s-*` variants may remove this behavior.

#### Amazon ECS settings

See the [setup guide](QUICKSTART-ECS.md) for much more detail on setting up Bottlerocket and ECS.

The following settings are optional and allow you to configure how your instance joins an ECS cluster.
Since joining a cluster happens at startup, they need to be [specified in user data](#using-user-data).

* `settings.ecs.cluster`: The name or [ARN](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) of your Amazon ECS cluster.
  If left unspecified, Bottlerocket will join your `default` cluster.
* `settings.ecs.instance-attributes`: [Attributes](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-placement-constraints.html#attributes) in the form of key, value pairs added when registering the container instance in the cluster.

  Example user data for setting up attributes:

  ```toml
  [settings.ecs.instance-attributes]
  attribute1 = "foo"
  attribute2 = "bar"
  ```

The following settings are optional and allow you to further configure your cluster.
These settings can be changed at any time.

* `settings.ecs.allow-privileged-containers`: Whether launching privileged containers is allowed on the container instance.
  If this value is set to false, privileged containers are not permitted.
  Bottlerocket sets this value to false by default.
* `settings.ecs.container-stop-timeout`: Time to wait for the task's containers to stop on their own before they are forcefully stopped.
Valid time units include `s`, `m`, and `h`, e.g. `1h`, `1m1s`.
* `settings.ecs.enable-spot-instance-draining`: If the instance receives a spot termination notice, the agent will set the instance's state to `DRAINING`, so the workload can be moved gracefully before the instance is removed. Defaults to `false`.
* `settings.ecs.image-pull-behavior`: The behavior used to customize the [pull image process](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-agent-config.html#ecs-agent-availparam) for your container instances.
  Supported values are `default`, `always`, `once`, `prefer-cached`, and the default is `default`.
* `settings.ecs.logging-drivers`: The list of logging drivers available on the container instance.
  The ECS agent running on a container instance must register available logging drivers before tasks that use those drivers are eligible to be placed on the instance.
  Bottlerocket enables the `json-file`, `awslogs`, and `none` drivers by default.
* `settings.ecs.loglevel`: The level of verbosity for the ECS agent's logs.
  Supported values are `debug`, `info`, `warn`, `error`, and `crit`, and the default is `info`.
* `settings.ecs.metadata-service-rps`: The steady state rate limit of the throttling configurations set for the task metadata service.
* `settings.ecs.metadata-service-burst`: The burst rate limit of the throttling configurations set for the task metadata service.
* `settings.ecs.reserved-memory`: The amount of memory, in MiB, reserved for critical system processes.
* `settings.ecs.task-cleanup-wait`: Time to wait before the task's containers are removed after they are stopped.
Valid time units are `s`, `m`, and `h`, e.g. `1h`, `1m1s`.
* `settings.ecs.image-cleanup-wait`: Time to wait between image cleanup cycles.
Valid time units are `s`, `m`, and `h`, e.g. `1h`, `1m1s`.
* `settings.ecs.image-cleanup-delete-per-cycle`: Number of images to delete in a single image cleanup cycle.
* `settings.ecs.image-cleanup-enabled`: Enable automatic images clean up after the tasks have been removed.
Defaults to `false`
* `settings.ecs.image-cleanup-age`: Time since the image was pulled to be considered for clean up.
Valid time units are `s`, `m`, and `h`, e.g. `1h`, `1m1s`.

  **Note**: `metadata-service-rps` and `metadata-service-burst` directly map to the values set by the `ECS_TASK_METADATA_RPS_LIMIT` environment variable.

#### CloudFormation signal helper settings

For AWS variants, these settings allow you to set up CloudFormation signaling to indicate whether Bottlerocket hosts running in EC2 have been successfully created or updated:

* `settings.cloudformation.logical-resource-id`: The logical ID of the AutoScalingGroup resource that you want to signal.
* `settings.cloudformation.should-signal`: Whether to check status and send signal. Defaults to `false`. If set to `true`, both `stack-name` and `logical-resource-id` need to be specified.
* `settings.cloudformation.stack-name`: Name of the CloudFormation Stack to signal.

#### Auto Scaling group settings

* `settings.autoscaling.should-wait`: Whether to wait for the instance to reach the `InService` state before the orchestrator agent joins the cluster. Defaults to `false`. Set this to `true` only if the instance is part of an Auto Scaling group, or will be attached to one later.
  For example:

  ```toml
  [settings.autoscaling]
  should-wait = true
  ```

#### OCI Hooks settings

Bottlerocket allows you to opt-in to use additional [OCI hooks](https://github.com/opencontainers/runtime-spec/blob/main/runtime.md#lifecycle) for your orchestrated containers.
Once you opt-in to use additional OCI hooks, any new orchestrated containers will be configured with them, but existing containers won't be changed.

* `settings.oci-hooks.log4j-hotpatch-enabled`: Enables the [hotdog OCI hooks](https://github.com/bottlerocket-os/hotdog), which are used to inject the [Log4j Hot Patch](https://github.com/corretto/hotpatch-for-apache-log4j2) into containers. Defaults to `false`.

#### OCI Defaults settings

Bottlerocket allows you to customize certain parts of the default [OCI spec](https://github.com/opencontainers/runtime-spec/blob/main/config.md) that is applied to workload containers.

The following settings are available:

##### OCI Defaults: Capabilities

All of the `capabilities` settings below are boolean values (`true`/`false`).

The full list of capabilities that can be configured in Bottlerocket are as follows:

capability | setting | default value
----- | ----- | -----
`CAP_AUDIT_WRITE` | `settings.oci-defaults.capabilities.audit-write` | true
`CAP_CHOWN` | `settings.oci-defaults.capabilities.chown` | true
`CAP_DAC_OVERRIDE` | `settings.oci-defaults.capabilities.dac-override` | true
`CAP_FOWNER` | `settings.oci-defaults.capabilities.fowner` | true
`CAP_FSETID` | `settings.oci-defaults.capabilities.fsetid` | true
`CAP_KILL` | `settings.oci-defaults.capabilities.kill` | true
`CAP_MKNOD` | `settings.oci-defaults.capabilities.mknod` | true
`CAP_NET_BIND_SERVICE` | `settings.oci-defaults.capabilities.net-bind-service` | true
`CAP_SETGID` | `settings.oci-defaults.capabilities.setgid` | true
`CAP_SETFCAP` | `settings.oci-defaults.capabilities.setfcap` | true
`CAP_SETPCAP` | `settings.oci-defaults.capabilities.setpcap` | true
`CAP_SETUID` | `settings.oci-defaults.capabilities.setuid` | true
`CAP_SYS_CHROOT` | `settings.oci-defaults.capabilities.sys-chroot` | true
`CAP_AUDIT_CONTROL` | `settings.oci-defaults.capabilities.audit-control` | -
`CAP_AUDIT_READ` | `settings.oci-defaults.capabilities.audit-read` | -
`CAP_BLOCK_SUSPEND` | `settings.oci-defaults.capabilities.block-suspend` | -
`CAP_BPF` | `settings.oci-defaults.capabilities.bpf` | -
`CAP_CHECKPOINT_RESTORE` | `settings.oci-defaults.capabilities.checkpoint-restore` | -
`CAP_DAC_READ_SEARCH` | `settings.oci-defaults.capabilities.dac-read-search` | -
`CAP_IPC_LOCK` | `settings.oci-defaults.capabilities.ipc-lock` | -
`CAP_IPC_OWNER` | `settings.oci-defaults.capabilities.ipc-owner` | -
`CAP_LEASE` | `settings.oci-defaults.capabilities.lease` | -
`CAP_LINUX_IMMUTABLE` | `settings.oci-defaults.capabilities.linux-immutable` | -
`CAP_MAC_ADMIN` | `settings.oci-defaults.capabilities.mac-admin` | -
`CAP_MAC_OVERRIDE` | `settings.oci-defaults.capabilities.mac-override` | -
`CAP_NET_ADMIN` | `settings.oci-defaults.capabilities.net-admin` | -
`CAP_NET_BROADCAST` | `settings.oci-defaults.capabilities.net-broadcast` | -
`CAP_NET_RAW` | `settings.oci-defaults.capabilities.net-raw` | -
`CAP_PERFMON` | `settings.oci-defaults.capabilities.perfmon` | -
`CAP_SYS_ADMIN` | `settings.oci-defaults.capabilities.sys-admin` | -
`CAP_SYS_BOOT` | `settings.oci-defaults.capabilities.sys-boot` | -
`CAP_SYS_MODULE` | `settings.oci-defaults.capabilities.sys-module` | -
`CAP_SYS_NICE` | `settings.oci-defaults.capabilities.sys-nice` | -
`CAP_SYS_PACCT` | `settings.oci-defaults.capabilities.sys-pacct` | -
`CAP_SYS_PTRACE` | `settings.oci-defaults.capabilities.sys-ptrace` | -
`CAP_SYS_RAWIO` | `settings.oci-defaults.capabilities.sys-rawio` | -
`CAP_SYS_RESOURCE` | `settings.oci-defaults.capabilities.sys-resource` | -
`CAP_SYS_TIME` | `settings.oci-defaults.capabilities.sys-time` | -
`CAP_SYS_TTY_CONFIG` | `settings.oci-defaults.capabilities.sys-tty-config` | -
`CAP_SYSLOG` | `settings.oci-defaults.capabilities.syslog` | -
`CAP_WAKE_ALARM` | `settings.oci-defaults.capabilities.wake-alarm` | -

##### OCI Defaults: Resource Limits

Each of the `resource-limits` settings below contain two numeric fields: `hard-limit` and `soft-limit`, which are **32-bit unsigned integers**.

Please see the [`getrlimit` linux manpage](https://man7.org/linux/man-pages/man7/capabilities.7.html) for meanings of `hard-limit` and `soft-limit`.

The full list of resource limits that can be configured in Bottlerocket are:

resource limit | setting | default value
----- | ----- | -----
`RLIMIT_NOFILE` | `settings.oci-defaults.resource-limits.max-open-files.hard-limit` | 1048576
`RLIMIT_NOFILE` | `settings.oci-defaults.resource-limits.max-open-files.soft-limit` | 65536

#### Container image registry settings

The following setting is optional and allows you to configure image registry mirrors and pull-through caches for your containers.

* `settings.container-registry.mirrors`: An array of container image registry mirror settings. Each element specifies the registry and the endpoints for said registry.
When pulling an image from a registry, the container runtime will try the endpoints one by one and use the first working one.
  (Docker and containerd will still try the default registry URL if the mirrors fail.)

  Example user data for setting up image registry mirrors:

  ```toml
  [[settings.container-registry.mirrors]]
  registry = "*"
  endpoint = ["https://<example-mirror>","https://<example-mirror-2>"]

  [[settings.container-registry.mirrors]]
  registry = "docker.io"
  endpoint = [ "https://<my-docker-hub-mirror-host>", "https://<my-docker-hub-mirror-host-2>"]
  ```

  If you use a Bottlerocket variant that uses Docker as the container runtime, like `aws-ecs-1`, you should be aware that Docker only supports pull-through caches for images from Docker Hub (docker.io). Mirrors for other registries are ignored in this case.

For [host-container](#host-containers-settings) and [bootstrap-container](#bootstrap-containers-settings) images from Amazon ECR private repositories, registry mirrors are currently unsupported.

The following setting is optional and allows you to configure image registry credentials.

* `settings.container-registry.credentials`: An array of container images registry credential settings. Each element specifies the registry and the credential information for said registry.
The credential fields map to [containerd's registry credential fields](https://github.com/containerd/containerd/blob/v1.6.0/docs/cri/registry.md#configure-registry-credentials), which in turn map to the fields in `.docker/config.json`.

  To avoid storing plaintext credentials in external systems, it is recommended to programmatically apply these settings via `apiclient` using a [bootstrap container](#bootstrap-containers-settings) or [host container](#host-containers-settings).

  Example `apiclient` call to set registry credentials for `gcr.io` and `docker.io`:

  ```shell
  apiclient set --json '{
    "container-registry": {
      "credentials": [
        {
          "registry": "gcr.io",
          "username": "example_username",
          "password": "example_password"
        },
        {
          "registry": "docker.io",
          "auth": "example_base64_encoded_auth_string"
        }
      ]
    }
  }'
  ```

  Example user data for setting up image registry credentials:
  ```toml
  [[settings.container-registry.credentials]]
  registry = "docker.io"
  username = "foo"
  password = "bar"

  [[settings.container-registry.credentials]]
  registry = "gcr.io"
  auth = "example_base64_encoded_auth_string"
  ```

In addition to the container runtime daemons, these credential settings will also apply to [host-container](#host-containers-settings) and [bootstrap-container](#bootstrap-containers-settings) image pulls as well.

#### Container runtime settings

Some behavior of the container runtime (currently `containerd`) can be modified with the following settings:

* `settings.container-runtime.enable-unprivileged-icmp`: Allow unprivileged containers to open ICMP echo sockets.
* `settings.container-runtime.enable-unprivileged-ports`: Allow unprivileged containers to bind to ports < 1024.
* `settings.container-runtime.max-concurrent-downloads`: Restricts the number of concurrent layer downloads for each image.
* `settings.container-runtime.max-container-log-line-size`: Controls how long container log messages can be.
  If the log output is longer than the limit, the log message will be broken into multiple lines.

Example container runtime settings:

```toml
[settings.container-runtime]
# Set log line length to unlimited
max-container-log-line-size = -1
max-concurrent-downloads = 4
enable-unprivileged-icmp = true
enable-unprivileged-ports = true
```

#### Updates settings

* `settings.updates.ignore-waves`: Updates are rolled out in waves to reduce the impact of issues. For testing purposes, you can set this to `true` to ignore those waves and update immediately.
* `settings.updates.metadata-base-url`: The common portion of all URIs used to download update metadata.
* `settings.updates.seed`: A `u32` value that determines how far into the update schedule this machine will accept an update. We recommend leaving this at its default generated value so that updates can be somewhat randomized in your cluster.
* `settings.updates.targets-base-url`: The common portion of all URIs used to download update files.
* `settings.updates.version-lock`: Controls the version that will be selected when you issue an update request. Can be locked to a specific version like `v1.0.0`, or `latest` to take the latest available version. Defaults to `latest`.

#### Network settings

* `settings.network.hostname`: The desired hostname of the system.

  **Important note for all Kubernetes variants:** Changing this setting at runtime (not via user data) can cause issues with kubelet registration, as hostname is closely tied to the identity of the system for both registration and certificates/authorization purposes.

  Most users don't need to change this setting as the following defaults work for the majority of use cases.
  If this setting isn't set we attempt to use DNS reverse lookup for the hostname.
  If the lookup is unsuccessful, the IP of the node is used.

* `settings.network.hosts`: A mapping of IP addresses to domain names which should resolve to those IP addresses.
   This setting results in modifications to the `/etc/hosts` file  for Bottlerocket.

   Note that this setting does not typically impact name resolution for containers, which usually rely on orchestrator-specific mechanisms for configuring static resolution.
   (See [ECS](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_HostEntry.html) and [Kubernetes](https://kubernetes.io/docs/tasks/network/customize-hosts-file-for-pods/) documentation for those mechanisms.)

   Example:

   ```toml
   [settings.network]
   hosts = [
    ["10.0.0.0", ["test.example.com", "test1.example.com"]],
    ["10.1.1.1", ["test2.example.com"]]
   ]
   ```

   This example would result in an `/etc/hosts` file entries like so:

   ```txt
   10.0.0.0 test.example.com test1.example.com
   10.1.1.1 test2.example.com
   ```

   Repeated entries are merged (including loopback entries), with the first aliases listed taking precedence. e.g.:

   ```toml
   [settings.network]
   hosts = [
    ["10.0.0.0", ["test.example.com", "test1.example.com"]],
    ["10.1.1.1", ["test2.example.com"]],
    ["10.0.0.0", ["test3.example.com"]],
   ]
   ```

   Would result in `/etc/hosts` entries like so:

   ```txt
   10.0.0.0 test.example.com test1.example.com test3.example.com
   10.1.1.1 test2.example.com
   ```

The following allows for custom DNS settings, which are used to generate the `/etc/resolv.conf`.
If either DNS setting is not populated, the system will use the DHCP lease of the primary interface to gather these settings.
See the `resolv.conf` [man page](https://man7.org/linux/man-pages/man5/resolv.conf.5.html) for more detail.

* `settings.dns.name-servers`: An array of IP address strings that represent the desired name server(s).
* `settings.dns.search-list`: An array of domain strings that represent the desired domain search path(s).

  ```toml
  [settings.dns]
  name-servers = ["1.2.3.4", "5.6.7.8"]
  search-list = ["foo.bar", "baz.foo"]
  ```

##### Proxy settings

These settings will configure the proxying behavior of the following services:

* For all variants:
  * [containerd.service](packages/containerd/containerd.service)
  * [host-containerd.service](packages/host-ctr/host-containerd.service)
* For Kubernetes variants:
  * [kubelet.service](packages/kubernetes-1.18/kubelet.service)
* For the ECS variant:
  * [docker.service](packages/docker-engine/docker.service)
  * [ecs.service](packages/ecs-agent/ecs.service)

* `settings.network.https-proxy`: The HTTPS proxy server to be used by services listed above.
* `settings.network.no-proxy`: A list of hosts that are excluded from proxying.

   Example:

   ```toml
   [settings.network]
   https-proxy = "1.2.3.4:8080"
   no-proxy = ["localhost", "127.0.0.1"]
   ```

The no-proxy list will automatically include entries for localhost.

If you're running a Kubernetes variant, the no-proxy list will automatically include the Kubernetes API server endpoint and other commonly used Kubernetes DNS suffixes to facilitate intra-cluster networking.

#### Metrics settings

By default, Bottlerocket sends anonymous metrics when it boots, and once every six hours.
This can be disabled by setting `send-metrics` to false.
Here are the metrics settings:

* `settings.metrics.metrics-url`: The endpoint to which metrics will be sent. The default is `https://metrics.bottlerocket.aws/v1/metrics`.
* `settings.metrics.send-metrics`: Whether Bottlerocket will send anonymous metrics.
* `settings.metrics.service-checks`: A list of systemd services that will be checked to determine whether a host is healthy.

#### Time settings

* `settings.ntp.time-servers`: A list of NTP servers used to set and verify the system time.

#### Kernel settings

* `settings.kernel.lockdown`: This allows further restrictions on what the Linux kernel will allow, for example preventing the loading of unsigned modules.
  May be set to "none" (the default in `*-nvidia` and `*-dev` variants), "integrity" (the default for other variants), or "confidentiality".
  **Important note:** this setting cannot be lowered (toward 'none') at runtime.
  You must reboot for a change to a lower level to take effect.
* `settings.kernel.modules.<name>.allowed`: Whether the named kernel module is allowed to be loaded.
  **Important note:** this setting does not affect kernel modules that are already loaded.
  You may need to reboot for a change to disallow a kernel module to take effect.

  Example user data for blocking kernel modules:

  ```toml
  [settings.kernel.modules.sctp]
  allowed = false

  [settings.kernel.modules.udf]
  allowed = false
  ```

* `settings.kernel.sysctl`: Key/value pairs representing Linux kernel parameters.
  Remember to quote keys (since they often contain ".") and to quote all values.

  Example user data for setting up sysctl:

  ```toml
  [settings.kernel.sysctl]
  "user.max_user_namespaces" = "16384"
  "vm.max_map_count" = "262144"
  ```

#### Boot-related settings

*Please note that boot settings currently only exist for the bare metal variants and \*-k8s-1.23 variants. Boot settings will be added to any future variant introduced after Bottlerocket v1.8.0.*

Specifying any of the following settings will generate a kernel boot config file to be loaded on subsequent boots:

* `settings.boot.init-parameters`: This allows additional init parameters to be specified on the kernel command line during boot.
* `settings.boot.kernel-parameters`: This allows additional kernel parameters to be specified on the kernel command line during boot.
* `settings.boot.reboot-to-reconcile`: If set to `true`, Bottlerocket will automatically reboot again during boot if either the `settings.boot.kernel-parameters` or `settings.boot.init-parameters` were changed via user data or a bootstrap container so that these changes may take effect.

You can learn more about kernel boot configuration [here](https://www.kernel.org/doc/html/latest/admin-guide/bootconfig.html).

Example user data for specifying boot settings:

```toml
[settings.boot]
reboot-to-reconcile = true

[settings.boot.kernel-parameters]
"console" = [
  "tty0",
  "ttyS1,115200n8",
]
"crashkernel" = [
  "2G-:256M",
]
"slub_debug" = [
  "options,slabs",
]
"usbcore.quirks" = [
  "0781:5580:bk",
  "0a5c:5834:gij",
]

[settings.boot.init-parameters]
"log_level" = ["debug"]
"splash" = []
```

If boot config data exists at `/proc/bootconfig`, it will be used to generate these API settings on first boot.
Please note that Bottlerocket only supports boot configuration for `kernel` and `init`. If any other boot config key is specified, the settings generation will fail.

#### Custom CA certificates settings

By default, Bottlerocket ships with the Mozilla CA certificate store, but you can add self-signed certificates through the API using these settings:

* `settings.pki.<bundle-name>.data`: Base64-encoded PEM-formatted certificates bundle; it can contain more than one certificate
* `settings.pki.<bundle-name>.trusted`: Whether the certificates in the bundle are trusted; defaults to `false` when not provided

Here's an example of adding a bundle of self-signed certificates as user data:

```toml
[settings.pki.my-trusted-bundle]
data="W3N..."
trusted=true

[settings.pki.dont-trust-these]
data="W3N..."
trusted=false
```

Here's the same example but using API calls:

```shell
apiclient set \
  pki.my-trusted-bundle.data="W3N..." \
  pki.my-trusted-bundle.trusted=true  \
  pki.dont-trust-these.data="N3W..."  \
  pki.dont-trust-there.trusted=false
```

You can use this method from within a [bootstrap container](#bootstrap-containers-settings), if your user data is over the size limit of the platform.

#### Host containers settings

* `settings.host-containers.admin.enabled`: Whether the admin container is enabled.
* `settings.host-containers.admin.source`: The URI of the [admin container](#admin-container).
* `settings.host-containers.admin.superpowered`: Whether the admin container has high levels of access to the Bottlerocket host.
* `settings.host-containers.control.enabled`: Whether the control container is enabled.
* `settings.host-containers.control.source`: The URI of the [control container](#control-container).
* `settings.host-containers.control.superpowered`: Whether the control container has high levels of access to the Bottlerocket host.

##### Custom host containers

[`admin`](https://github.com/bottlerocket-os/bottlerocket-admin-container) and [`control`](https://github.com/bottlerocket-os/bottlerocket-control-container) are our default host containers, but you're free to change this.
Beyond just changing the settings above to affect the `admin` and `control` containers, you can add and remove host containers entirely.
As long as you define the three fields above -- `source` with a URI, and `enabled` and `superpowered` with true/false -- you can add host containers with an API call or user data.

You can optionally define a `user-data` field with arbitrary base64-encoded data, which will be made available in the container at `/.bottlerocket/host-containers/$HOST_CONTAINER_NAME/user-data` and (since Bottlerocket v1.0.8) `/.bottlerocket/host-containers/current/user-data`.
(It was inspired by instance user data, but is entirely separate; it can be any data your host container feels like interpreting.)

Keep in mind that the default admin container (since Bottlerocket v1.0.6) relies on `user-data` to store SSH keys. You can set `user-data` to [customize the keys](https://github.com/bottlerocket-os/bottlerocket-admin-container/#authenticating-with-the-admin-container), or you can use it for your own purposes in a custom container.

Here's an example of adding a custom host container with API calls:

```shell
apiclient set \
  host-containers.custom.source=MY-CONTAINER-URI \
  host-containers.custom.enabled=true \
  host-containers.custom.superpowered=false
```

Here's the same example, but with the settings you'd add to user data:

```toml
[settings.host-containers.custom]
enabled = true
source = "MY-CONTAINER-URI"
superpowered = false
```

If the `enabled` flag is `true`, it will be started automatically.

All host containers will have the `apiclient` binary available at `/usr/local/bin/apiclient` so they're able to [interact with the API](#using-the-api-client).
You can also use `apiclient` to run programs in other host containers.
For example, to access the admin container:

```shell
apiclient exec admin bash
```

In addition, all host containers come with persistent storage that survives reboots and container start/stop cycles.
It's available at `/.bottlerocket/host-containers/$HOST_CONTAINER_NAME` and (since Bottlerocket v1.0.8) `/.bottlerocket/host-containers/current`.
The default `admin` host-container, for example, stores its SSH host keys under `/.bottlerocket/host-containers/admin/etc/ssh/`.

There are a few important caveats to understand about host containers:

* They're not orchestrated. They only start or stop according to that `enabled` flag.
* They run in a separate instance of containerd than the one used for orchestrated containers like Kubernetes pods.
* They're not updated automatically. You need to update the `source` and commit those changes.
* If you set `superpowered` to true, they'll essentially have root access to the host.

Because of these caveats, host containers are only intended for special use cases.
We use them for the control container because it needs to be available early to give you access to the OS, and for the admin container because it needs high levels of privilege and because you need it to debug when orchestration isn't working.

Be careful, and make sure you have a similar low-level use case before reaching for host containers.

#### Bootstrap containers settings

* `settings.bootstrap-containers.<name>.essential`: whether or not the container should fail the boot process, defaults to `false`
* `settings.bootstrap-containers.<name>.mode`: the mode of the container, it could be one of `off`, `once` or `always`. See below for a description of modes.
* `settings.bootstrap-containers.<name>.source`: the image for the container
* `settings.bootstrap-containers.<name>.user-data`: field with arbitrary base64-encoded data

Bootstrap containers are host containers that can be used to "bootstrap" the host before services like ECS Agent, Kubernetes, and Docker start.

Bootstrap containers are very similar to normal host containers; they come with persistent storage and with optional user data.
Unlike normal host containers, bootstrap containers can't be treated as `superpowered` containers.
However, bootstrap containers do have additional permissions that normal host containers do not have.
Bootstrap containers have access to the underlying root filesystem on `/.bottlerocket/rootfs` as well as to all the devices in the host, and they are set up with the `CAP_SYS_ADMIN` capability.
This allows bootstrap containers to create files, directories, and mounts that are visible to the host.

Bootstrap containers are set up to run after the systemd `configured.target` unit is active.
The containers' systemd unit depends on this target (and not on any of the bootstrap containers' peers) which means that bootstrap containers will not execute in a deterministic order.
The boot process will "wait" for as long as the bootstrap containers run.
Bootstrap containers configured with `essential=true` will stop the boot process if they exit code is a non-zero value.

Bootstrap containers have three different modes:

* `always`: with this setting, the container is executed on every boot.
* `off`: the container won't run
* `once`: with this setting, the container only runs on the first boot where the container is defined. Upon completion, the mode is changed to `off`.

Here's an example of adding a bootstrap container with API calls:

```shell
apiclient set \
  bootstrap-containers.bootstrap.source=MY-CONTAINER-URI \
  bootstrap-containers.bootstrap.mode=once \
  bootstrap-containers.bootstrap.essential=true
```

Here's the same example, but with the settings you'd add to user data:

```toml
[settings.bootstrap-containers.bootstrap]
source = "MY-CONTAINER-URI"
mode = "once"
essential = true
```

##### Mount propagations in bootstrap and superpowered containers

Both bootstrap and superpowered host containers are configured with the `/.bottlerocket/rootfs/mnt` bind mount that points to `/mnt` in the host, which itself is a bind mount of `/local/mnt`.
This bind mount is set up with shared propagations, so any new mount point created underneath `/.bottlerocket/rootfs/mnt` in any bootstrap or superpowered host container will propagate across mount namespaces.
You can use this feature to configure ephemeral disks attached to your hosts that you may want to use on your workloads.

#### Platform-specific settings

Platform-specific settings are automatically set at boot time by [early-boot-config](sources/api/early-boot-config) based on metadata available on the running platform.
They can be overridden for testing purposes in [the same way as other settings](#interacting-with-settings).

##### AWS-specific settings

* `settings.aws.config`: The base64 encoded content to use for AWS configuration (e.g. `base64 -w0 ~/.aws/config`).
* `settings.aws.credentials`: The base64 encoded content to use for AWS credentials (e.g. `base64 -w0 ~/.aws/credentials`).
* `settings.aws.profile`: The profile name to use from the provided `config` and `credentials` settings.

  For example:

  ```toml
  [settings.aws]
  profile = "myprofile"
  ```

  **Note**: If `settings.aws.profile` is not set, the setting will fallback to the "default" profile.
  In general it is recommended not to include a `[profile default]` section in the `aws.config` contents though.
  This may have unintended side effects for other AWS services running on the node (e.g. `aws-iam-authenticator`).

  **Note:** The `config`, `credentials`, and `profile` are optional and do not need to be set when using an Instance Profile when running on an AWS instance.

* `settings.aws.region`: This is set to the AWS region in which the instance is running, for example `us-west-2`.

  The `region` setting is automatically inferred based on calls to the Instance MetaData Service (IMDS) when running within AWS.
  It does not need to be explicitly set unless you have a reason to override this default value.

### Logs

You can use `logdog` through the [admin container](#admin-container) to obtain an archive of log files from your Bottlerocket host.
SSH to the Bottlerocket host or `apiclient exec admin bash` to access the admin container, then run:

```shell
sudo sheltie
logdog
```

This will write an archive of the logs to `/var/log/support/bottlerocket-logs.tar.gz`.
This archive is accessible from host containers at `/.bottlerocket/support`.
You can use SSH to retrieve the file.
Once you have exited from the Bottlerocket host, run a command like:

```shell
ssh -i YOUR_KEY_FILE \
  ec2-user@YOUR_HOST \
  "cat /.bottlerocket/support/bottlerocket-logs.tar.gz" > bottlerocket-logs.tar.gz
```

(If your instance isn't accessible through SSH, you can use [SSH over SSM](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started-enable-ssh-connections.html).)

For a list of what is collected, see the logdog [command list](sources/logdog/src/log_request.rs).

### Kdump Support

Bottlerocket provides support to collect kernel crash dumps whenever the system kernel panics.
Once this happens, both the dmesg log and vmcore dump are stored at `/var/log/kdump`, and the system reboots.

There are a few important caveats about the provided kdump support:

* Currently, only vmware variants have kdump support enabled
* The system kernel will reserve 256MB for the crash kernel, only when the host has at least 2GB of memory; the reserved space won't be available for processes running in the host
* The crash kernel will only be loaded when the `crashkernel` parameter is present in the kernel's cmdline and if there is memory reserved for it

### NVIDIA GPUs Support

Bottlerocket's `nvidia` variants include the required packages and configurations to leverage NVIDIA GPUs.
Currently, the following NVIDIA driver versions are supported in Bottlerocket:

* 470.X
* 515.X

The official AMIs for these variants can be used with EC2 GPU-equipped instance types such as: `p2`, `p3`, `p4`, `g3`, `g4dn`, `g5` and `g5g`.
Note that older instance types, such as `p2`, are not supported by NVIDIA driver `515.X` and above.
You need to make sure you select the appropriate AMI depending on the instance type you are planning to use.
Please see [QUICKSTART-EKS](QUICKSTART-EKS.md#aws-k8s--nvidia-variants) for further details about Kubernetes variants, and [QUICKSTART-ECS](QUICKSTART-ECS.md#aws-ecs--nvidia-variants) for ECS variants.

## Details

### Security

:shield: :crab:

To learn more about security features in Bottlerocket, please see [SECURITY FEATURES](SECURITY_FEATURES.md).
It describes how we use features like [dm-verity](https://gitlab.com/cryptsetup/cryptsetup/wikis/DMVerity) and [SELinux](https://selinuxproject.org/) to protect the system from security threats.

To learn more about security recommendations for Bottlerocket, please see [SECURITY GUIDANCE](SECURITY_GUIDANCE.md).
It documents additional steps you can take to secure the OS, and includes resources such as a [Pod Security Policy](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) for your reference.

In addition, almost all first-party components are written in [Rust](https://www.rust-lang.org/).
Rust eliminates some classes of memory safety issues, and encourages design patterns that help security.

### Packaging

Bottlerocket is built from source using a container toolchain.
We use RPM package definitions to build and install individual packages into an image.
RPM itself is not in the image - it's just a common and convenient package definition format.

We currently package the following major third-party components:

* Linux kernel ([background](https://en.wikipedia.org/wiki/Linux), [5.10 packaging](packages/kernel-5.10/), [5.15 packaging](packages/kernel-5.15/))
* glibc ([background](https://www.gnu.org/software/libc/), [packaging](packages/glibc/))
* Buildroot as build toolchain ([background](https://buildroot.org/), via the [SDK](https://github.com/bottlerocket-os/bottlerocket-sdk))
* GRUB, with patches for partition flip updates ([background](https://www.gnu.org/software/grub/), [packaging](packages/grub/))
* systemd as init ([background](https://en.wikipedia.org/wiki/Systemd), [packaging](packages/systemd/))
* wicked for networking ([background](https://github.com/openSUSE/wicked), [packaging](packages/wicked/))
* containerd ([background](https://containerd.io/), [packaging](packages/containerd/))
* Kubernetes ([background](https://kubernetes.io/), [packaging](packages/kubernetes-1.24/))
* aws-iam-authenticator ([background](https://github.com/kubernetes-sigs/aws-iam-authenticator), [packaging](packages/aws-iam-authenticator/))
* Amazon ECS agent ([background](https://github.com/aws/amazon-ecs-agent), [packaging](packages/ecs-agent/))

For further documentation or to see the rest of the packages, see the [packaging directory](packages/).

### Updates

The Bottlerocket image has two identical sets of partitions, A and B.
When updating Bottlerocket, the partition table is updated to point from set A to set B, or vice versa.

We also track successful boots, and if there are failures it will automatically revert back to the prior working partition set.

The update process uses images secured by [TUF](https://theupdateframework.github.io/).
For more details, see the [update system documentation](sources/updater/).

### API

There are two main ways you'd interact with a production Bottlerocket instance.
(There are a couple more [exploration](#exploration) methods above for test instances.)

The first method is through a container orchestrator, for when you want to run or manage containers.
This uses the standard channel for your orchestrator, for example a tool like `kubectl` for Kubernetes.

The second method is through the Bottlerocket API, for example when you want to configure the system.

There's an HTTP API server that listens on a local Unix-domain socket.
Remote access to the API requires an authenticated transport such as SSM's RunCommand or Session Manager, as described above.
For more details, see the [apiserver documentation](sources/api/apiserver/).

The [apiclient](sources/api/apiclient/) can be used to make requests.
They're just HTTP requests, but the API client simplifies making requests with the Unix-domain socket.

To make configuration easier, we have [early-boot-config](sources/api/early-boot-config/), which can send an API request for you based on instance user data.
If you start a virtual machine, like an EC2 instance, it will read TOML-formatted Bottlerocket configuration from user data and send it to the API server.
This way, you can configure your Bottlerocket instance without having to make API calls after launch.

See [Settings](#settings) above for examples and to understand what you can configure.

You can also access host containers through the API using [apiclient exec](sources/api/apiclient/README.md#exec-mode).

The server and client are the user-facing components of the API system, but there are a number of other components that work together to make sure your settings are applied, and that they survive upgrades of Bottlerocket.

For more details, see the [API system documentation](sources/api/).

### Default Volumes

Bottlerocket operates with two default storage volumes.

* The root device, holds the active and passive [partition sets](#updates-1).
  It also contains the bootloader, the dm-verity hash tree for verifying the [immutable root filesystem](SECURITY_FEATURES.md#immutable-rootfs-backed-by-dm-verity), and the data store for the Bottlerocket API.
* The data device is used as persistent storage for container images, container orchestration, [host-containers](#Custom-host-containers), and [bootstrap containers](#Bootstrap-containers-settings).

On boot Bottlerocket will increase the data partition size to use all of the data device.
If you increase the size of the device, you can reboot Bottlerocket to extend the data partition.
If you need to extend the data partition without rebooting, have a look at this [discussion](https://github.com/bottlerocket-os/bottlerocket/discussions/2011).
