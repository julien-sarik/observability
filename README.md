# observability
## build
```
podman build keycloak/ -t keycloak:local
```
## run
Create podman network
```
podman network create foo
```

```
podman play kube pods.yml --net foo
```

Browse Prometheus at `http://localhost:9090` and Grafana at `http://localhost:3000` using admin/admin credentials.  
The Grafana data source can be configured in one of the 2 following ways:  
- mount a YAML config file into `/usr/share/grafana/provisioning/datasources` as described in https://grafana.com/docs/grafana/latest/datasources/prometheus/#provision-the-data-source
- use the Grafana UI as described in https://grafana.com/docs/grafana/latest/datasources/#add-a-data-source

Shutdown
```
podman play kube pods.yml --down
```

## metrics
### keycloak
Keycloak is running on `http://localhost:8080` and metrics are available at `http://localhost:9000/metrics`.  
#### user session
`# HELP vendor_statistics_approximate_entries_unique Approximate number of entries currently in the cache for which the local node is a primary owner, including persisted and expired entries`  
`vendor_statistics_approximate_entries_unique{cache="sessions"}`  

`# HELP vendor_statistics_approximate_entries Approximate number of entries currently in the cache, including persisted and expired entries`  

`# HELP vendor_statistics_store_primary_owner_total The number of single key stores when this node is the primary owner`  
For some reason the metric below is always `-1`:  
`HELP vendor_statistics_number_of_entries_in_memory Number of entries currently in-memory excluding expired entries`
### traefik
Traefik is running on `http://localhost:8888` and metrics are available at `http://localhost:8889/metrics`.  
Example of request proxied by Traefik to Keycloak `http://localhost:8888/realms/master/.well-known/openid-configuration`.  
The granularity is per Traefik service i.e. not per endpoint within a service.  
#### duration per service
histogram:  
`traefik_service_request_duration_seconds`

## doc
### Prometheus
https://prometheus.io/docs/prometheus/latest/getting_started/

Average of the request duration over time:  
```
increase(traefik_service_request_duration_seconds_sum[$__rate_interval]) / increase(traefik_service_request_duration_seconds_count[$__rate_interval])
```