# tig_linux_server
Telegraf Influxdb Grafana docker compose with dashboard for the host linux server

To run use
docker compose up 
and point a browser at localhost:3001

To run in the backrun
docker compose up -d

To clean up and remove the containers/data 
docker compose down

Useful things to know. 
The Grafana and Influxdb containers are both part of the same bridge tig_linux_server_default
docker network inspect tig_linux_server_default

docker uses hostnames(DNS) within a bridge for connectivity between the containers. Hence the datasource is the container name of Influxdb in the Grafana provisioning folder

grafana/provisioning/datasources/myinfluxdb.yaml
url: http://myinfluxdb:8086

Note the telegraf container is not in the same bridge as the other two containers. Telegraf is connected to the host for networking. This is to make it easier for telegraf to connect with other external hosts. 
This is achieved by specifying network_mode: "host"  in the docker compose file under telegraf config. Hence telegraf has to use the host localhost:8087 mapping to connecto the Influxdb container which is inside the tig_linux_server_default bridge. 

Influxdb
A couple of ways to connect via docker
docker exec -it myinfluxdb influx
show databases
use telegraf_metics
show measurements

Also via localhost:8087

Grafana
localhost:3001
 
