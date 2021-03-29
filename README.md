# kubectl-cost

`kubectl-cost` is a [kubectl plugin](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/) that provides easy CLI access to Kubernetes cost allocation metrics via the [kubecost APIs](https://github.com/kubecost/docs/blob/master/apis.md). It allows developers, devops, and others to quickly determine the cost & efficiency for any Kubernetes workload.

<img src="assets/regular.gif" alt="Standard CLI Usage" width="600">

<img src="assets/tui.gif" alt="TUI Usage" width="600">

## Installation

1. [Install Kubecost](https://docs.kubecost.com/install)

> This software requires that you have a running deployment of [Kubecost](https://kubecost.com/) in your cluster. 


#### Linux/MacOS

``` sh
os=$(uname | tr '[:upper:]' '[:lower:]') && \
arch=$(uname -m | tr '[:upper:]' '[:lower:]' | sed -e s/x86_64/amd64/) && \
curl -s -L https://github.com/kubecost/kubectl-cost/releases/latest/download/kubectl-cost-$os-$arch.tar.gz | tar xz -C /tmp && \
sudo mv /tmp/kubectl-cost /usr/local/bin/kubectl-cost
```


#### Latest Release

If you prefer to download from GitHub, or are on Windows, go the the [releases](https://github.com/kubecost/kubectl-cost/releases) and download the appropriate binary for your system. Rename it to `kubectl-cost` and put it in your `PATH`.

As long as the binary is still named `kubectl-cost` and is somewhere in your `PATH`, it will be usable. This is because `kubectl` automatically finds plugins by looking for executables prefixed with `kubecost-` in your `PATH`.

## Usage

There are five supported subcommands: `namespace`, `deployment`, `controller`, `label`, and `tui`, which display cost information aggregated by the name of the subcommand (see Examples). Each subcommand has two primary modes, rate and non-rate. Rate (the default) displays the projected monthly cost based on the activity during the window. Non-rate (`--historical`) displays the total cost for the duration of the window.

The exception to these descriptions is `kubectl cost tui`, which displays a TUI and is currently limited to only monthly rate projections. It currently supports all of the previously mentioned aggregations except label. These limitations are because the TUI is an experimental feature - if you like it, let us know! We'd be happy to dedicate time to expanding its functionality.


#### Examples
Show the projected monthly rate for each namespace
based on the last 5 days of activity.
``` sh
kubectl cost namespace --window 5d
```

Show how much each namespace cost over the past 5 days
with additional CPU and memory cost and without efficiency.
``` sh
kubectl cost namespace \
  --historical \
  --window 5d \
  --show-cpu \
  --show-memory \
  --show-efficiency=false
```

Show the projected monthly rate for each controller
based on the last 5 days of activity with PV (persistent
volume) cost breakdown.
``` sh
kubectl cost controller --window 5d --show-pv
```

Show costs over the past 5 days broken down by the value
of the `app` label:
``` sh
kubectl cost label --historical -l app
```

Show the projected monthly rate for each deployment
based on the last month of activity with CPU, memory,
GPU, PV, and network cost breakdown.
``` sh
kubectl cost deployment --window month -A
```

Show the projected monthly rate for each deployment
in the `kubecost` namespace based on the last 3 days
of activity with CPU cost breakdown.
``` sh
kubectl cost deployment \
  --window 3d \
  --show-cpu \
  -n kubecost
```

The same, but with a non-standard Kubecost deployment
in the namespace `kubecost-staging` with the cost
analyzer service called `kubecost-staging-cost-analyzer`.
``` sh
kubectl cost deployment \
  --window 3d \
  --show-cpu \
  -n kubecost \
  -N kubecost-staging \
  --service-name kubecost-staging-cost-analyzer
```



#### Flags
See `kubectl cost [subcommand] --help` for the full set of flags.

The following flags modify the behavior of the subcommands:
```
    --historical                  show the total cost during the window instead of the projected monthly rate based on the data in the window"
    --show-cpu                    show data for CPU cost
    --show-efficiency             show efficiency of cost alongside CPU and memory cost (default true)
    --show-gpu                    show data for GPU cost
    --show-memory                 show data for memory cost
    --show-network                show data for network cost
    --show-pv                     show data for PV (physical volume) cost
    --show-shared                 show shared cost data
-A, --show-all-resources          Equivalent to --show-cpu --show-memory --show-gpu --show-pv --show-network.
    --window string               the window of data to query (default "yesterday")
    --service-name string         The name of the kubecost cost analyzer service. Change if you're running a non-standard deployment, like the staging helm chart. (default "kubecost-cost-analyzer")
-n, --namespace string            Limit results to only one namespace. Defaults to all namespaces.
-N, --kubecost-namespace string   The namespace that kubecost is deployed in. Requests to the API will be directed to this namespace. (default "kubecost")
```


`kubectl cost` has to interact with the Kubernetes API server. It tries to use your kubeconfig. These flags are common to `kubectl` and allow you to customize this behavior.
``` sh
      --as string                      Username to impersonate for the operation
      --as-group stringArray           Group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --cache-dir string               Default cache directory (default "/home/delta/.kube/cache")
      --certificate-authority string   Path to a cert file for the certificate authority
      --client-certificate string      Path to a client certificate file for TLS
      --client-key string              Path to a client key file for TLS
      --cluster string                 The name of the kubeconfig cluster to use
      --context string                 The name of the kubeconfig context to use
  -h, --help                           help for cost
      --insecure-skip-tls-verify       If true, the server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kubeconfig string              Path to the kubeconfig file to use for CLI requests.
      --request-timeout string         The length of time to wait before giving up on a single server request. Non-zero values should contain a corresponding time unit (e.g. 1s, 2m, 3h). A value of zero means don't timeout requests. (default "0")
  -s, --server string                  The address and port of the Kubernetes API server
      --tls-server-name string         Server name to use for server certificate validation. If it is not provided, the hostname used to contact the server is used
      --token string                   Bearer token for authentication to the API server
      --user string                    The name of the kubeconfig user to use
```

