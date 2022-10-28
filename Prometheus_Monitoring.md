# **Prometheus monitoring for Openshift**
Openshift provides  kube-state-metric api through which we can monitor state and health of the Cluster.

**How to get API endpoint:**

1. **Go to Copy login command for the openshift cluster.**
<img width="329" alt="Screen Shot 2022-10-28 at 10 17 26 AM" src="https://user-images.githubusercontent.com/46498945/198671799-c7e3d1df-f2e1-4202-8367-3f2422b3f66f.png">


**2. Get Authorization token and and end point, it should look something like:** 

`curl -H "Authorization: Bearer sha256~SFZ9T9s8DHoQP5esDjXjhisuzWJLOlBzA0LlLgA3AOc" "https://api.ocp46.cpd.walmart.com:6443/apis/user.openshift.io/v1/users/~"`

To create a service account, with a session token which does not expire, for use with scripted access, use the `oc create sa` command, and pass the name to give the service account.

To view details of the service account created, run `oc describe` on the service account resource.

The Bearer token is auto-generated and will expire. But if you need a persistent token, it is recommended to create a service account and persistent token with it:

To create a service account, with a session token which does not expire, for use with scripted access, use the oc create sa command, and pass the name to give the service account.
```
$ oc create sa robot
serviceaccount "robot" created
```

To view details of the service account created, run oc describe on the service account resource.


```
$ oc describe sa robot
Name:        robot
Namespace:   cookbook
Labels:      <none>
Annotations: <none>

Image pull secrets: robot-dockercfg-vl9qn

Mountable secrets:  robot-token-mhf9x
                    robot-dockercfg-vl9qn

Tokens:             robot-token-4nkdw
                    robot-token-mhf9x

```
The service account created can have [Role Based Access](https://docs.openshift.com/container-platform/4.8/authentication/using-rbac.html) assigned to it.

![Screen Shot 2022-10-28 at 10 34 14 AM](https://user-images.githubusercontent.com/46498945/198672298-acf5d5f4-a243-4ebe-9f8f-8019b4e3c9e6.png)


**3. Once you have the token add this to to Prometheus config file as [scrape_config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config):**
 
```
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.

  - job_name: 'scrape_walmart_ocp'
    scheme: https
    authorization:
      credentials: sha256~SFZ9T9s8DHoQP5esDjXjhisuzWJLOlBzA0LlLgA3AOc
    tls_config:
      insecure_skip_verify: true
    metrics_path: '/apis/metrics.k8s.io/v1beta1/nodes'
    scrape_interval: 15s
    static_configs:
      - targets: ['api.ocp46.cpd.walmart.com:6443']
```


There multiple metric endpoint available and you can create seperate job for each endpoint. Some of the paths to be monitored are:
```
apis/metrics.k8s.io/v1beta1/nodes
apis/metrics.k8s.io/v1beta1/pods
apis/project.openshift.io/v1/projects
api/v1/namespaces
```
Some of the metrics you will get:
```
      "metadata": {
        "name": "ocp46-ijjzc-worker-1",
        "creationTimestamp": "2022-10-28T14:51:54Z",
        "labels": {
          "beta.kubernetes.io/arch": "amd64",
          "beta.kubernetes.io/instance-type": "Standard_D16s_v3",
          "beta.kubernetes.io/os": "linux",
          "failure-domain.beta.kubernetes.io/region": "westus2",
          "failure-domain.beta.kubernetes.io/zone": "westus2-2",
          "kubernetes.io/arch": "amd64",
          "kubernetes.io/hostname": "ocp46-ijjzc-worker-1",
          "kubernetes.io/os": "linux",
          "node-role.kubernetes.io/worker": "",
          "node.kubernetes.io/instance-type": "Standard_D16s_v3",
          "node.openshift.io/os_id": "rhcos",
          "topology.kubernetes.io/region": "westus2",
          "topology.kubernetes.io/zone": "westus2-2"
        }
      },
      "timestamp": "2022-10-28T14:51:54Z",
      "window": "1m0s",
      "usage": {
        "cpu": "725m",
        "memory": "13726220Ki"
```

```
      "metadata": {
        "name": "mysql-586d947454-mfjw9",
        "namespace": "pvc-test",
        "creationTimestamp": "2022-10-28T14:54:02Z",
        "labels": {
          "deployment": "mysql",
          "pod-template-hash": "586d947454"
        }
      },
      "timestamp": "2022-10-28T14:54:02Z",
      "window": "5m0s",
      "containers": [
        {
          "name": "POD",
          "usage": {
            "cpu": "0",
            "memory": "184Ki"
          }
        },
        {
          "name": "mysql",
          "usage": {
            "cpu": "1m",
            "memory": "328136Ki"
          }
```         



You can get list of apis by:
```
curl -k -H "Authorization: Bearer sha256~SFZ9T9s8DHoQP5esDjXjhisuzWJLOlBzA0LlLgA3AOc" "https://api.ocp46.cpd.walmart.com:6443/"
```

**4. Once added and loaded in prometheus, tagets should come up  and you can query them and visualize them as needed:**
![Unknown](https://user-images.githubusercontent.com/46498945/198672448-516fe179-3b12-43d9-bd23-7d1c6ab3d4b2.png)
