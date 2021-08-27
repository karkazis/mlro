# Resource Monitoring and Analysis tools

## Monitoring tools deployment

### System prerequisites

* **Operating System:**
    Ubuntu 18 LTS or higher 64bit
    
* **Docker:**
    Docker engine 19.03.12

* **Internet connection**

**1. Prometheus server**
Prometheus is an open-source monitoring and alerting toolkit, which is bundled in a standalone server, independent of any strict requirements, like network storage or dependency on remote services.
```bash
 sudo docker run -d  -p 9090:9090  -v ./monitoring-server/config:/etc/prometheus --name prometheus  prom/prometheus
```

**2. Pushgateway server**
Despite the fact that the default approach proposed by Prometheus is to retrieve the metrics data by performing http get requests to different types (containers, VMs, physical machines etc) of monitoring targets, the utilization of Pushgateway might provide several advantages, like no need for implementation of specific application exporters or configuration of the Prometheus for short live applications. 
```bash
sudo docker run -d -p 9091:9091 --name pushgateway prom/pushgateway
```

**3. Grafana**
```bash
sudo docker run -d -p 3000:3000 --name grafana grafana/grafana
```

**4. Netdata monitoring server**
Netdata.io is a powerful tool designed to collect a huge list of metrics from every system and application in real-time. For the virtual machines monitoring a Netdata plugin is used, specially designed to collect metrics from vSphere ESXi. So, Prometheus collects the target metrics from the Netdata for both physical and virtual resources.

```bash
 sudo docker run -d \
 -p 19999:19999   \
 -v monitoring-targets/config:/etc/netdata/go.d -v /proc:/host/proc:ro   \
 -v /sys:/host/sys:ro   \
 -v /var/run/docker.sock:/var/run/docker.sock:ro   \
 --cap-add SYS_PTRACE   \
 --name=netdata   \
 --security-opt apparmor=unconfined  
 netdata/netdata
```

**4. cAdvisor**
cAdvisor (Container Advisor) provides container users an understanding of the resource usage and performance characteristics of their running containers. It is a running daemon that collects, aggregates, processes, and exports information about running containers. Specifically, for each container it keeps resource isolation parameters, historical resource usage, histograms of complete historical resource usage and network statistics. This data is exported by container and machine-wide.
```bash
sudo docker run   \
--volume=/:/rootfs:ro   \
--volume=/var/run:/var/run:ro   \
--volume=/sys:/sys:ro   \
--volume=/var/lib/docker/:/var/lib/docker:ro   \
--volume=/dev/disk/:/dev/disk:ro   \
--publish=8080:8080   \
--detach=true   \
--name=cadvisor   \
--privileged   \
--device=/dev/kmsg   \
gcr.io/cadvisor/cadvisor
```

**3. Kubernetes cluster monitoring**
Prometheus monitoring system is able to collect information from kubernetes clusters, this functionality requires the deployment of a separate Prometheus instance inside the k8s cluster and the configuration of the monitoring servet to scrap the metrics.

```bash
kubectl apply -f /monitoring-targets/config/k8s-prometheus.yaml
```

## Analysis tools deployment
### System prerequisites

* **Operating System:**
    Ubuntu 18 LTS or higher (64bit)
    
* **Docker:**
    Docker engine 19.03.12

* **Internet connection**

**1. Spark cluster deployment**
Apache Spark is a unified analytics engine for large-scale data processing. It provides high-level APIs in Java, Scala, Python and R, and an optimized engine that supports general execution graphs. It also supports a rich set of higher-level tools including Spark SQL for SQL and structured data processing, MLlib for machine learning, GraphX for graph processing, and Structured Streaming for incremental computation and stream processing.

```bash
cd analysis-server
docker build -t cluster-apache-spark:3.0.2 .
docker-compose -d up
```


## Licensing 
All the tools are under Apache 2.0 license.