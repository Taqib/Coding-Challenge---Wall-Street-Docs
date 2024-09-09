
# Coding Challenge - Wall Street Docs

Project Description:

I have been given tasked by WSD to work on a set of technical problems and provide detailed feedback on each. The project involves addressing five distinct challenges, which cover various areas of system configuration, containerization, and database management. The outlined problems are as follows:

01.Configuration Management: Implementing solutions using Ansible 2.9.16 and Puppet 4.5 or above to manage system configurations and automate tasks.

02.Docker/Kubernetes: Working in an environment based on Ubuntu 20 LTS with Docker 19 or higher, addressing containerization and orchestration scenarios.

03.Helm Deployment: Utilizing a Helm template to create a Kubernetes deployment of Elasticsearch. After setting up the environment, I will provide both a screenshot of the deployment and the resulting YAML configuration.

04.Metrics: Using Prometheus and Granfana

05.Database Management: Solving scenario-based issues related to Cassandra 4.0 or above and MongoDB 4.4.0 or above, identifying possible reasons for the problems and offering solutions.

This project requires a thorough understanding of configuration management, containerization, orchestration tools, and database troubleshooting to deliver feedback and solutions to WSD.




## Authors

- [shamsunnabitaqib@gmail.com](https://www.github.com/octokatherine)


## Badges
[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)
[![GPLv3 License](https://img.shields.io/badge/License-GPL%20v3-yellow.svg)](https://opensource.org/licenses/)
[![AGPL License](https://img.shields.io/badge/license-AGPL-blue.svg)](http://www.gnu.org/licenses/agpl-3.0)


## Documentation

[Ansible Problem & It's Solution](https://github.com/Taqib/Coding-Challenge---Wall-Street-Docs?tab=readme-ov-file)
1) Display all Ansible configurations for a host:

ans: 
```ansible -m setup <host>```,

Example:
```ansible -m setup app-vm1.fra1.internal```

2)Configure a cron job for logrotate every 10 minutes between 2h - 4h:

ans: To set up a cron job that runs logrotate every 10 minutes between 2 AM and 4 AM on all machines, you can add the following line to your crontab (edit using crontab -e):

```*/10 2-4 * * * /usr/sbin/logrotate /etc/logrotate.conf```

To deploy this across all machines using Ansible, you can create a simple playbook that ensures the cron job is set.

Playbook: ```cron_logrotate.yml```

```
---
- hosts: all
  become: yes
  tasks:
    - name: Ensure cron job for logrotate is set
      cron:
        name: "logrotate every 10 minutes between 2h - 4h"
        minute: "*/10"
        hour: "2-4"
        job: "/usr/sbin/logrotate /etc/logrotate.conf"
```

You can run this playbook using:

```ansible-playbook cron_logrotate.yml```

3) Deploy the ntpd package to the 3 servers with a custom configuration file:

You can use the following Ansible playbook to deploy the ntpd package and apply a custom configuration file /etc/ntpd.conf to the three specified servers.

Playbook: ```ntpd_deploy.yml```

```
---
- hosts: ntp_servers
  become: yes
  tasks:
    - name: Ensure ntpd package is installed
      apt:
        name: ntpd
        state: present

    - name: Deploy custom ntpd.conf
      copy:
        src: /path/to/custom/ntpd.conf
        dest: /etc/ntpd.conf
        owner: root
        group: root
        mode: 0644

    - name: Restart ntpd service
      service:
        name: ntp
        state: restarted
```

Inventory file: ```hosts```

```
[ntp_servers]
app-vm1.fra1.internal ansible_host=192.168.0.2
db-vm1.fra1.db ansible_host=192.168.0.3
web-vm1.fra1.web ansible_host=192.168.0.4
```

To run this playbook:

```
ansible-playbook -i hosts ntpd_deploy.yml
```

4) Deploy Nagios monitoring template on the Nagios server:
To deploy Nagios templates for monitoring these servers on monitoring.fra1.internal, you would first create the necessary configuration files for Nagios to monitor the servers.

Here's an example playbook to deploy the Nagios templates:

Playbook: ```nagios_monitoring.yml```

```
---
- hosts: monitoring.fra1.internal
  become: yes
  tasks:
    - name: Create Nagios configuration for app-vm1.fra1.internal
      template:
        src: nagios_templates/app-vm1.fra1.internal.cfg.j2
        dest: /etc/nagios/conf.d/app-vm1.fra1.internal.cfg
        owner: nagios
        group: nagios
        mode: 0644

    - name: Create Nagios configuration for db-vm1.fra1.db
      template:
        src: nagios_templates/db-vm1.fra1.db.cfg.j2
        dest: /etc/nagios/conf.d/db-vm1.fra1.db.cfg
        owner: nagios
        group: nagios
        mode: 0644

    - name: Create Nagios configuration for web-vm1.fra1.web
      template:
        src: nagios_templates/web-vm1.fra1.web.cfg.j2
        dest: /etc/nagios/conf.d/web-vm1.fra1.web.cfg
        owner: nagios
        group: nagios
        mode: 0644

    - name: Restart Nagios service
      service:
        name: nagios
        state: restarted
```

Example of Nagios Template: ```app-vm1.fra1.internal.cfg.j2```

```
define host {
    use                     linux-server
    host_name               app-vm1.fra1.internal
    alias                   Application Server
    address                 192.168.0.2
}

define service {
    use                     generic-service
    host_name               app-vm1.fra1.internal
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}

define service {
    use                     generic-service
    host_name               app-vm1.fra1.internal
    service_description     NTP
    check_command           check_ntp_time
}
```
You would need to create similar templates for the other two servers (```db-vm1.fra1.db``` and ```web-vm1.fra1.web```).

To apply this, run the playbook:
```
ansible-playbook -i hosts nagios_monitoring.yml
```
These steps will handle the tasks across all systems as requested.

## Architecture Diagram of the system:

```
                            +-------------------------------+
                            |       Monitoring Server        |
                            |         (Nagios)               |
                            | monitoring.fra1.internal       |
                            |        (192.168.0.5)           |
                            +-------------------------------+
                                      ^       ^       ^
                                      |       |       |
                      +---------------+       |       +---------------+
                      |                       |                       |
     +--------------------------+   +-------------------------+   +--------------------------+
     |      Application VM       |   |        Database VM       |   |         Web VM           |
     |  app-vm1.fra1.internal    |   |    db-vm1.fra1.db        |   |    web-vm1.fra1.web       |
     |       (192.168.0.2)       |   |      (192.168.0.3)       |   |      (192.168.0.4)        |
     +--------------------------+   +-------------------------+   +--------------------------+
               ^                             ^                            ^
               |                             |                            |
               +----------------------------------------------------------+
                                      |
                                Ansible Control Node
                                      |
                           +--------------------------+
                           |     Ansible (Deployer)    |
                           |    Configuration & Mgmt   |
                           |   ansible-controller-node |
                           |                           |
                           +--------------------------+
```

This design leverages automation (Ansible) to maintain a consistent and highly reliable infrastructure, combined with robust monitoring (Nagios) to ensure system health.





[Docker & Kubernets](https://github.com/Taqib/Coding-Challenge---Wall-Street-Docs?tab=readme-ov-file)

1. Docker Compose for an Nginx Server:

A)	Prepare a docker-compose for a nginx server.
Requirements:
•	nginx logs need to survive between nginx container restarts
•	docker should use network bridge subnet 172.20.8.0/24

Ans:
Here's a sample ```docker-compose.yml``` file that meets your requirements:

```
version: '3.8'

services:
  nginx:
    image: nginx:latest
    container_name: nginx-server
    ports:
      - "80:80"
    volumes:
      - ./nginx/logs:/var/log/nginx  # Persistent log storage
    networks:
      custom_network:
        ipv4_address: 172.20.8.2
    restart: always

networks:
  custom_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.8.0/24
```

Explanation:

i)Logs are stored on the host machine in the ./nginx/logs folder to persist between container restarts.

ii)The custom bridge network is created with the specified subnet 172.20.8.0/24 and the Nginx container is assigned a static IP of 172.20.8.2.

iii)The restart: always ensures that the container will restart automatically in case of a failure.


2. Kubernetes Command to Identify Pod Restart Reason:

To check the reason for the pod restart in the "internal" project under the "production" namespace, you can use the following Kubernetes command:

```kubectl describe pod <pod-name> -n production```

To filter out the pod restart reason, look for the ```Last State``` and ```Events``` sections in the output, as they will show details about the last container exit and reasons for restarts.

3. Possible Reasons for Java-App Pod Restarts:

Considering the resource quota provided for the Java app, the restarts might occur for the following reasons:

i) Memory Limit Exceeded: The pod's memory limit is set to 1500 MiB, and the application has an Xmx (maximum heap size) of 1000 MiB. This doesn't leave enough room for non-heap memory (e.g., native memory or metaspace), which could cause the application to exceed its memory limits and lead to OOM (Out of Memory) errors. This will trigger pod restarts.

ii) CPU Resource Throttling: The pod has a CPU limit of 2000 millicores (2 cores), but if the app demands more CPU, Kubernetes will throttle it. This could lead to performance degradation and potential restarts due to health checks failing or timeouts.

You can check for memory or CPU-related issues using:

```kubectl describe pod java-app-7d9d44ccbf-lmvbc -n production```

and look for ```OOMKilled``` status or resource throttling information under the ```Events``` section.

*** To resolve the issue of memory limit exceeded for the Java application, there are a few adjustments you can make in both the Java application's configuration and the Kubernetes resource limits.

Solutions:

1. Increase the Pod’s Memory Limit:

i) If possible, increase the memory limit of the Kubernetes pod to give the application enough headroom for heap and non-heap memory. Since the Java app's heap is set to Xmx 1000M, you need to account for non-heap memory like Metaspace, threads, and native memory.

ii) You can adjust the memory limit to, for example, 2000Mi or 2500Mi.

Update the pod's resource specification:

```
resources:
  requests:
    memory: "1000Mi"
  limits:
    memory: "2500Mi"
```
2. Adjust Java Memory Settings (Xmx and Xms):

i) Tuning the -Xmx value can help balance heap memory usage and non-heap memory.

ii) If increasing the pod’s memory limit isn't an option, reduce the -Xmx to ensure the pod doesn’t hit the limit. For example, lowering it to -Xmx 800M could allow for enough space for other memory needs.

3. Enable Java Native Memory Tracking:

i) To better understand how memory is being used, you can enable Java Native Memory Tracking (NMT). This will help diagnose whether excessive native memory usage is contributing to the problem.

Enable NMT using:

```-XX:NativeMemoryTracking=summary```

4. Use Java 11 or Later:
i) Java 11 has better memory management compared to older versions, especially in terms of non-heap memory like Metaspace. If feasible, upgrade to Java 11 or later.

5. Fine-Tune Garbage Collection (GC) Settings:
i) You can experiment with different GC algorithms and settings, such as G1GC, to reduce memory usage and overhead.

ii) Example:
```
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```
6. Monitor and Adjust Kubernetes Health Checks:

i) Ensure that the Kubernetes liveness and readiness probes have appropriate timeouts to prevent restarts due to temporary memory spikes.

ii) You can extend the initial delay or timeouts for the probes if memory-intensive initialization causes issues.

Example YAML for a Revised Resource Limit:

```
resources:
  requests:
    memory: "1000Mi"
    cpu: "1000m"
  limits:
    memory: "2500Mi"
    cpu: "2000m"
```
Conclusion:
The best approach depends on whether increasing the memory limit is acceptable or if the Java application needs more efficient memory usage. Start by monitoring the app’s memory consumption, either by increasing the limit or fine-tuning the JVM settings.


[HELM](https://github.com/Taqib/Coding-Challenge---Wall-Street-Docs?tab=readme-ov-file)

*** After try to create the elastisearch environment by running  helm commnd i got following error:

```
helm install elasticsearch ./elasticsearch/
Error: INSTALLATION FAILED: template: elasticsearch/templates/statefulset.yaml:1:23: executing "elasticsearch/templates/statefulset.yaml" at <.Values.global.name>: nil pointer evaluating interface {}.name
```

*** I also try to run lint command to figure out where is the error in the entire code base, & after running the following commnd i still couldn't figure out what's the actual error or problem:

```
helm lint elasticsearch/
==> Linting elasticsearch/
[INFO] Chart.yaml: icon is recommended
[INFO] values.yaml: file does not exist
[ERROR] templates/: template: elasticsearch/templates/statefulset.yaml:1:23: executing "elasticsearch/templates/statefulset.yaml" at <.Values.global.name>: nil pointer evaluating interface {}.name

Error: 1 chart(s) linted, 1 chart(s) failed
```
Note: I am not used to helm & also have very little knowledge on this tech. I tried my best but couldn't figure out as i am super busy with client project so i got very little time after office hour.So please accept my appologies in this regards.


[Metrics](https://github.com/Taqib/Coding-Challenge---Wall-Street-Docs?tab=readme-ov-file)

1) How Prometheus Works:
Ans: Prometheus is an open-source monitoring and alerting toolkit designed for reliability and scalability. It collects metrics from targets by scraping HTTP endpoints at specified intervals. Prometheus stores this data in a time-series database, where metrics are identified by a name and key-value pairs (labels).
Key Concepts:

i) Scraping: Prometheus periodically pulls metrics data from HTTP endpoints using a “pull” model.

ii) Exporters: Applications expose metrics in a specific format, and exporters are used for metrics that don’t natively support this (e.g., Node Exporter for system metrics).

iii) Time Series Database (TSDB): Prometheus stores metrics data with a timestamp, allowing it to track historical data and trends.

iv) PromQL (Prometheus Query Language): This is used to query and aggregate the metrics.

v) Alerting: Prometheus evaluates rules and triggers alerts when specified conditions are met.

2) Creating Custom Prometheus Alerts for Kubernetes Monitoring:
To create a custom Prometheus alert for Kubernetes, you define alerting rules in a YAML file. These rules specify conditions for alerts (using PromQL expressions) and can be configured to integrate with an alert manager to handle notifications.

Steps:
i) Create or modify an alerting rule YAML file.
ii) Use PromQL to define conditions for the alerts.
iii) Reload Prometheus configuration or apply the alerting rule file.

Example Alert Rule:

```
groups:
  - name: Kubernetes Alerts
    rules:
      - alert: HighPodRestartCount
        expr: sum(rate(kube_pod_container_status_restarts_total[5m])) by (namespace) > 5
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Pod has a high restart count"
          description: "Pod in namespace {{ $labels.namespace }} is experiencing more than 5 restarts within 10 minutes."
```
In this example:
*) expr: PromQL query ```sum(rate(kube_pod_container_status_restarts_total[5m])) by (namespace) > 5``` looks for pods that are restarting more than 5 times in 5 minutes.

**) for: The alert will trigger if the condition persists for 10 minutes.

***) labels and annotations: Provide metadata and context for the alert.

3) Prometheus Query for Grafana to Show Usage Trend of a Counter:

When working with counters in Prometheus (which always increase), you want to visualize the rate of increase over time rather than the raw counter values, which will continuously grow.

To display the rate of usage in Grafana for a counter metric (e.g., ```http_requests_total```), you can use the following query:

```promql```

```rate(http_requests_total[5m])```

****) rate(http_requests_total[5m]): Calculates the per-second rate of increase in the ```http_requests_total``` counter over the past 5 minutes. This is useful for detecting trends in traffic or resource usage.

For longer trends, you can adjust the time window to a larger period like [1h] or [24h].


## Architecture Diagram of the overall alerting system:

The architecture can be broken into key components:
```
+-----------------------------------------------+
|          Kubernetes Cluster                   |
|-----------------------------------------------|
| +-------------------+   +-------------------+ |
| | App Pods          |   | Kube-State-Metrics | |
| | (Nginx, Custom)   |   +-------------------+ |
| +-------------------+                         |
| +-------------------+   +-------------------+ |
| | Node Exporter     |   | Other Exporters    | |
| +-------------------+   +-------------------+ |
+-----------------------------------------------+

             | Scrapes metrics |
             v                 v
+-------------------------------------------------+
|                Prometheus Server                |
| +---------------------------------------------+ |
| | Time-Series DB   | Alerting Rules Engine    | |
| +---------------------------------------------+ |
| | Alert Manager                               | |
| +---------------------------------------------+ |
+-------------------------------------------------+

       | Alerts                                  | Metrics
       v                                         v
+--------------------+            +---------------------------------+
| Notification System|            |              Grafana            |
| (Slack, Email)     |            | +-----------------------------+ |
+--------------------+            | | Customizable Dashboards      | |
                                   | +-----------------------------+ |
                                   | +-----------------------------+ |
                                   | | Application Performance      | |
                                   | +-----------------------------+ |
                                   +---------------------------------+
```



[Databases](https://github.com/Taqib/Coding-Challenge---Wall-Street-Docs?tab=readme-ov-file)

Suggested environment: Cassandra 4.0 or above, mongo 4.4.0 or above

## 1)	Cassandra:

The issue you're facing in Cassandra where queries return different results and include records that were deleted days ago is likely caused by eventual consistency and stale replicas. Here’s a breakdown of the likely reasons:

A. Eventual Consistency Model
* Cassandra uses an eventual consistency model, meaning that data changes (like deletes or updates) propagate across replicas over time. If a query is run before all replicas have received the latest updates, it may return different results, depending on which replica is queried.

B. Tombstones and Delayed Deletion
* In Cassandra, a delete operation doesn't immediately remove data. Instead, it creates a tombstone, which is a marker indicating that data was deleted. The tombstone is later processed during the compaction process, but if compaction hasn’t run or is delayed, old data may still be returned by some nodes.

C. Replica Synchronization Issues
* Some nodes in your cluster may have outdated information due to failed replication processes, network partitioning, or node failures. These nodes may continue to return stale data if they haven’t received the updated delete operations.

## How to Avoid the Issue:
A. Tune Consistency Levels:
* Use a stronger consistency level for both reads and writes. For example, QUORUM or ALL consistency levels ensure that more replicas are checked before returning a result or confirming a write operation. This reduces the likelihood of querying a stale replica.

B. Run Repair Operations:
* Periodically run ```nodetool repair``` to ensure data consistency across all replicas. This helps synchronize nodes that may have missed delete or update operations.

C. Ensure Regular Compaction:
* Make sure compaction is running frequently. You can manually trigger compaction via ```nodetool compact``` to remove old tombstones and ensure that deleted records are fully removed.

D. Monitor Tombstones:
* Keep an eye on the number of tombstones. Too many tombstones can slow down queries and cause inconsistent results. You can monitor this using Cassandra logs and tools.

By addressing these areas, you can reduce the chances of stale data appearing in query results.


## 2) Mongo:

To shard the ```sanfrancisco.company_name``` collection based on ```_id```, follow these steps:

1. Upgrade MongoDB:
Before proceeding with sharding, you need to upgrade your MongoDB version to at least 4.4.0 since your current MongoDB version (3.6.18) does not support sharding on replica sets in the way required for efficient sharding. Follow the MongoDB upgrade documentation to upgrade both the config server and the replica sets.

2. Enable Sharding on the Database:
After upgrading MongoDB, connect to your MongoDB instance and enable sharding on the ```sanfrancisco``` database.

```bash```
```
mongos> use admin
mongos> sh.enableSharding("sanfrancisco")
```
3. Shard the ```company_name``` Collection:
To shard the collection based on the ```_id``` field, use the ```sh.shardCollection``` command.

```bash```

```
mongos> sh.shardCollection("sanfrancisco.company_name", { "_id": 1 })
```
This command initiates sharding on the ```sanfrancisco.company_name``` collection using the ```_id``` field as the shard key.

4. Add Replicaset_2 as a Shard:
Assuming replicaset_2 is already initialized, add it as a shard.

```bash```

```
mongos> sh.addShard("replicaset_2/mongo-replicaset2-host:port")
```

5. Verify Sharding:
Check the sharding status to verify if the collection is being sharded and balanced across ```replicaset_1``` and ```replicaset_2```.

```bash```

``` mongos> sh.status() ```
This will show the shard distribution and status for ```sanfrancisco.company_name```.

6. Monitor the Balancing Process:

Once the sharding is set up, MongoDB’s balancer should start distributing data across the shards. To ensure this, you can start the balancer if needed:

sh.startBalancer()

You can check the balancer status:
sh.getBalancerState()   
MongoDB will start balancing the data across the shards. You can monitor the balancer's progress using:

```bash```
```
mongos> use config
mongos> db.locks.find({ _id: "balancer" })
```
Additionally, check the ```sharding``` logs to track how data is distributed.

7. Performance Testing:
Once sharding is enabled and data has been distributed, monitor the performance to ensure it has improved. You can use ```mongostat```, ```mongotop```, and MongoDB logs to assess improvements.

This process ensures the ```sanfrancisco.company_name``` collection will be evenly distributed between ```replicaset_1``` and ```replicaset_2```, resolving performance issues related to insufficient hardware in ```replicaset_1```.







## 🚀 About Me
I am a skilled DevOps engineer and AWS-certified professional with 4 years of experience in building, optimizing, and managing infrastructure and platforms. My expertise spans public cloud environments, particularly AWS, and Linux-based systems, with a strong focus on automation, programming, and system administration. I am dedicated to designing and maintaining robust CI/CD pipelines that streamline software delivery. By collaborating across teams, I consistently work to improve deployment processes, enhance system reliability, and drive the efficient delivery of high-quality solutions to end users.

