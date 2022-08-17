# Misc

## KMS

First of all, most keys you work with are Symmetric (S3, RDS, etcetera). There's a reason for this: Symmetric keys are just more useful if you can make sure the keys don't have to leave AWS (which is usually the case). However, you can also make other types of keys:

- Symmetric - The default.
- Asymmetric - Use if you want a public key that can leave AWS. You can use RSA or elliptic curves.
- HMAC - Symmetric, for generating HMAC codes. HMAC codes are used to 'sign'/authenticate a message - If you receive a message + correct HMAC code, you can be sure the message was created by somebody with access to the key.
- Data key - This is an ad-hoc symmetric key you can use in your own application. AWS returns a plaintext and encrypted key. The encrypted key can be decrypted using the main AWS-managed key. The plain key is used to encrypt/decrypt data. The key is not stored in AWS, the idea is you store the encrypted key along side the data.

Secondly, there is a difference between customer-managed keys and AWS managed keys. The main difference is the access policy - AWS managed keys are by definition accesible by the entire account (of course provided you have the kms:decrypt or such role). Also, customer managed keys allow you to control the renewal yourself.