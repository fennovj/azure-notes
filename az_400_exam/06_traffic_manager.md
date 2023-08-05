# Azure Traffic manager

<https://learn.microsoft.com/en-us/azure/traffic-manager/traffic-manager-monitoring>
<https://learn.microsoft.com/en-us/azure/traffic-manager/traffic-manager-routing-methods>
<https://learn.microsoft.com/en-us/azure/architecture/guide/technology-choices/load-balancing-overview>

Azure Traffic Manager is a DNS-based load balancer. Generally, it is not very recommended by Azure, as it cannot take into account information within the request itself, compared to Azure Front Door, which can load balance e.g. on request parameters. 
However, being DNS-based does allow it to easily work globally - e.g. route requests based on geographical location.

Also note that due to being DNS-based, the client connects *directly* to your endpoint. Compared to e.g. Azure Front Door, where every request is routed through Front Door. With Traffic Manager, the client does a single DNS query, and after that, connects directly to the server.

Also, Traffic Manager implements blue-green deployments. (this is not neccecarily the only way to achieve this. e.g. Azure WebApps can also be used to achieve blue/green deployments)

## Routing methods

- Priority routing method: give a list of endpoints. e.g. 'Primary', 'Failover 1', 'Failover 2'. Traffic is routed to the highest priority available endpoint.
- Weighted routing method: give a list of weights. e.g. 50 for eu-west-1, 50 for eu-west-2. you can do a lot more with this, e.g.:
  - A/B testing: send a small percentage to a test server
  - Gradual upgrades: gradually increase the weight of a server. This is also called a 'blue/green' deployment.

A risk with Weighted routing is that due to recursive DNS caching, the weights may not apply how you expect. E.g. let's say an entire office uses the same DNS server, likely that entire office will get the same endpoint, regardless of what the weights are.
- Performance routing method: Using the query source IP adress, look up the closest available endpoint. e.g. if source IP is from south asia, and you have endpoints in europe and east asia, the endpoint from east asia will be returned.
- Geography routing method: uses a lookup table, where each endpoint has an assigned region. e.g. Endpoint 1 is Germany, Endpoint 2 is the rest.
- 