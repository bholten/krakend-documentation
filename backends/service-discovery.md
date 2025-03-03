---
lastmod: 2022-11-09
date: 2016-09-30
aliases: ["/service-discovery/dns-srv/", "/docs/service-discovery/overview/"]
linktitle: Service Discovery
title: Service Discovery
weight: 10
menu:
  community_current:
    parent: "050 Backends Configuration"
---
The service discovery (`sd`) is an optional attribute of the `backend` section that enables KrakenD to detect and locate services automatically on your enterprise network.

{{< note title="Related read" type="tip" >}}
You might also want to read the [Load Balancer](/docs/throttling/load-balancing/) documentation
{{< /note >}}


The chosen **service discovery strategy** determines how to retrieve (statically or dynamically) the final list of IPs, hostnames, or services pointing to your backends. If your host list is dynamic, you can use an external service discovery provider and let KrakenD interact with it to get the hosts. If your host list is static (it doesn't change) or you use a service name or an external balancer, you can use `static` resolution and directly use the values provided under `host[]`.

KrakenD must be in a network that can reach any declared hosts. With more than one host, KrakenD [load balances](/docs/throttling/load-balancing/) the connections to the hosts in the list.

The **possible values** for `sd` are:

- `static`: For static resolution
- `dns`: For DNS SRV resolution

See below.

## Static resolution
The `static` resolution is the default service discovery strategy. It implies that you write directly in the configuration the protocol plus the service name, hosts or IPs you want to connect to.

The `static` resolution uses a **list** of hosts to **load balance** (in a Round Robin fashion) all servers in the list, and you should expect more or less an equivalent number of hits on each backend. However, if you use a **Kubernetes service**, then it load-balances itself so that you only need one entry.

{{< note title="Declaring hosts on Kubernetes" type="tip" >}}
When the consumed hosts are behind a balancer or a service name, write a single entry in the array with that name.
{{< /note >}}

To use static resolution, you don't need to declare anything other than the `host` list. However, you can add the `"sd": "static"` property in the backend configuration for a more explicit reading, as it is the default value when `sd` is not declared). Example:

```json
{
"backend": [
    {
        "url_pattern": "/some-url",
        "sd": "static",
        "host": [
            "http://my-service-01.api.com:9000",
            "http://my-service-02.api.com:9000"
        ]
    }
]
}
```

## DNS SRV Service Discovery (Kubernetes/Consul)
The `DNS SRV`([see RFC](https://datatracker.ietf.org/doc/html/rfc2782)) is a market standard used by systems such as **Kubernetes, Mesos, Haproxy, Nginx plus, AWS ECS, Linkerd**, and many more. An SRV entry is a custom DNS record that establishes connections between services. When KrakenD needs to know the location of a specific service, it will search for a related SRV record.

The format of the `SRV` record is as follows:

    _service._proto.name. ttl IN SRV priority weight port target

**Example**. A service running on port `8000` with maximum priority (`0`) and a weight `5` ):

    _api._tcp.example.com. 86400 IN SRV 0 5 8000 foo.example.com.

{{< note title="Caching" type="info" >}}
The DNS-based service discovery caches entries for 30 seconds.
{{< /note >}}

To integrate **Consul, Kubernetes, or any other `DNS SRV` compatible system** as the Service Discovery, you only need to set two keys:

- `"sd": "dns"`: To use dynamic host resolution using the service discovery strategy
- `"host": []`: And entry with the service name providing the resolution (e.g.: Consul address)

Add these keys in the `backend` section of your configuration. If there is another `host` key in the root level of the configuration, you don't need to declare it here if the value is the same.

For instance:

```json
{
    "backend": [
        {
            "url_pattern": "/foo",
            "sd": "dns",
            "host": [
                "api-catalog.service.consul.srv"
            ],
            "disable_host_sanitize": true
        }
    ]
}
```
With the configuration above, KrakenD will query every 30 seconds the `api-catalog.service.consul.srv` DNS and will apply to the internal balancer any weights and priorities returned by the DNS record.