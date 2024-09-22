# tig_linux_server
Telegraf, Influxdb and Grafana docker compose with dashboard for a typical host Linux server

## To run use
_docker compose up_ and point a browser at localhost:3001

## To run in the backround
  * docker compose up -d  
  * To clean up and remove the containers/data  
     * docker compose down  

## Networking stuff. 
The Grafana and Influxdb containers are both part of the same bridge tig_linux_server_default.    
   * docker network inspect tig_linux_server_default

```
ekenny@fedora:~/tig_linux_server$ docker network inspect tig_linux_server_default
[
    {
        "Name": "tig_linux_server_default",
        "Id": "d1435bacb06c24034d76556130ff0fa6c76214b307ba86e070634afe03db0721",
        "Created": "2024-09-22T21:07:19.885342834+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "1bb924f770b9d69fad278b9efed242b01364213a3c6466c83c5fd0e6dcfdee00": {
                "Name": "myinfluxdb",
                "EndpointID": "d35945e003780abfbf3d02c836fc930e91fbdebd750225526fee0dbb5d8e80f4",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "b97343097c0ce32efc8b042992c631f9006225f2bf6ddf4cd0091509f21fd905": {
                "Name": "mygrafana",
                "EndpointID": "a80345cbc7ada2edc629f77277ff210c54714c2e59f307a1ecc8ded468987e6a",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "default",
            "com.docker.compose.project": "tig_linux_server",
            "com.docker.compose.version": "2.29.6"
        }
    }
]
```
```
ekenny@fedora:~/tig_linux_server$ docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED         STATUS         PORTS                                         NAMES
b97343097c0c   grafana/grafana:10.3.1       "/run.sh"                8 seconds ago   Up 6 seconds   0.0.0.0:3001->3000/tcp, [::]:3001->3000/tcp   mygrafana
e9ee78b0fc42   telegraf:latest              "/entrypoint.sh --co…"   8 seconds ago   Up 6 seconds                                                 mytelegraf
1bb924f770b9   influxdb:1.8                 "/entrypoint.sh infl…"   8 seconds ago   Up 7 seconds   0.0.0.0:8087->8086/tcp, [::]:8087->8086/tcp   myinfluxdb
```

   Note docker uses hostnames (DNS resolution) within a bridge for connectivity between the containers. Hence the datasource is the container name of Influxdb in the Grafana provisioning folder.  
   Note the telegraf container is not in the same bridge as the other two containers. 
   Telegraf is connected to the host for networking. This is to make it easier for telegraf to connect with other external hosts. 
   This is achieved by specifying:
   * network_mode: "host"  in the docker compose file under telegraf config.   
    Hence telegraf has to use the host localhost:8087 mapping to connect to the Influxdb container from the host to tig_linux_server_default bridge. 

## Influxdb
A couple of ways to connect via docker
* docker exec -it myinfluxdb influx
 * Useful commands
   * show databases
   * use telegraf_metics
   * show measurements
   * select * from cpu limit 10;
   * SELECT mean(usage_user) as "user", mean(usage_system) as "system", mean(usage_softirq) as "softirq", mean(usage_steal) as "steal", mean(usage_nice) as "nice", mean(usage_irq) as "irq", mean(usage_iowait) as "iowait", mean(usage_guest) as "guest", mean(usage_guest_nice) as "guest_nice"  FROM "cpu" WHERE "host" =~ /$server$/ and cpu = 'cpu-total' AND $timeFilter GROUP BY time($interval), *
* from the host to localhost:8087
  can use a local influx client on the host to connect.

## Grafana
   username:password admin:admin   
   http://localhost:3001   
   You tell grafana how to connect to Influxdb in the provisiong folder: grafana/provisioning/datasources/myinfluxdb.yaml  
   url: http://myinfluxdb:8086   
   Note that you use the hostname of the influxdb docker container(myinfluxdb) as this is inside the same bridge as the Grafana server
  
