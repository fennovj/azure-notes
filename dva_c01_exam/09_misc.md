# Misc

## KMS

First of all, most keys you work with are Symmetric (S3, RDS, etcetera). There's a reason for this: Symmetric keys are just more useful if you can make sure the keys don't have to leave AWS (which is usually the case). However, you can also make other types of keys:

- Symmetric - The default.
- Asymmetric - Use if you want a public key that can leave AWS. You can use RSA or elliptic curves.
- HMAC - Symmetric, for generating HMAC codes. HMAC codes are used to 'sign'/authenticate a message - If you receive a message + correct HMAC code, you can be sure the message was created by somebody with access to the key.
- Data key - This is an ad-hoc symmetric key you can use in your own application. AWS returns a plaintext and encrypted key. The encrypted key can be decrypted using an existing symmetric key. The plain key is used to encrypt/decrypt data. The key is not stored in AWS, the idea is you store the encrypted key alongside the data.

Secondly, there is a difference between customer-managed keys and AWS managed keys. The main difference is the access policy - AWS managed keys are by definition accesible by the entire account (of course provided you have the kms:decrypt or such role), while Customer keys can have a key policy. Also, customer managed keys allow you to control the renewal yourself.

## Redis vs Memcached

<https://aws.amazon.com/elasticache/redis-vs-memcached/>

Generally, Memcache is simpler to use, Redis has more advanced features.

Memcached is multithreaded

Redis has snapshots, replication, more scripting like lua and geospatial functions. Also Redis is better if you want high availability, e.g. if your cache is really important to not overload the application.

So in summary: use Memcached either if the question specifically asks for multithreaded, or for something as simple to set up as possible. Otherwise, especially if the question asks for availability and performance, use Redis.

