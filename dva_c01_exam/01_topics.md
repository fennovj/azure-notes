# Topics

These are topics I need to study. Mainly how they work, terminology, maybe test them out

- Step Functions
- X-Ray
- API Gateway (Proxy integration?)
- Lambda

## Redis vs Memcached

<https://aws.amazon.com/elasticache/redis-vs-memcached/>

Generally, Memcache is simpler to use, Redis has more advanced features.

Memcached is multithreaded

Redis has snapshots, replication, more scripting like lua and geospatial functions. Also Redis is better if you want high availability, e.g. if your cache is really important to not overload the application.

So in summary: use Memcached either if the question specifically asks for multithreaded, or for something as simple to set up as possible. Otherwise, especially if the question asks for availability and performance, use Redis.

