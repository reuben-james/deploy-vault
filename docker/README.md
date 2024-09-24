# Vault in docker with a local filesystem storage backend

# build docker
docker build -t vault-local-fs .

# run vault with filesystem backend
docker-compose up -d vault-local-fs

# POST secrets using HTTP
VAULT_TOKEN=<A working token>
curl \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -H "Content-Type: application/json" \
    -X POST \
    -d '{ "tf_var_01": "test-password" }' \
    https://vault:8200/v1/kv/tf-overrides/test

curl \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -H "Content-Type: application/json" \
    -X POST \
    -d '{ "tf_var_01": "prod-password" }' \
    https://vault:8200/v1/kv/tf-overrides/prod

# GET secrets
curl \
    -H "X-Vault-Token: $VAULT_TOKEN" \
    -H "Content-Type: application/json" \
    -X GET \
    https://vault:8200/v1/kv/tf-overrides/test


# TLS (UI/API)

### /etc/hosts

The config in this repo sets Vault up in docker compose and hosts vault on https://vault

To make this work you need the following entry in /etc/hosts
```
0.0.0.0		vault
```

### WARNING

Vault requires the following files for enable TLS on the TCP listeners (HTTP):

| Env Varible (and Default location)           | What it contains                              |
| ---------------------------------------------| --------------------------------------------- |
| VAULT_CACERT=/opt/vault/tls/vault-ca.pem     | CA Chain -> IntermediateCA + RootCA           |
| VAULT_TLSCERT=/opt/vault/tls/vault-cert.pem  | Client Chain -> Server Cert + IntermediateCA  |
| VAULT_TLSKEY=/opt/vault/tls/vault-key.pem    | The Server Cert's Private Key                 |

### Docker compose TLS config

Setting Cert ENV in docker-compose.yml
```
environment:
  - VAULT_CACERT=/opt/vault/tls/vault-ca.pem
  - VAULT_TLSCERT=/opt/vault/tls/vault-cert.pem
  - VAULT_TLSKEY=/opt/vault/tls/vault-key.pem
```

Cert file VOLUMES: 
```
volumes:
  - ./tls/ca-chain.pem:/opt/vault/tls/vault-ca.pem
  - ./tls/client-chain.pem:/opt/vault/tls/vault-cert.pem
  - ./tls/vault.key.pem:/opt/vault/tls/vault-key.pem
```

### Client certificate (Browser)

You will also need to have a client certificate, issued by the same Trusted RootCA/IntermediateCA imported into your browser. This needs to be a p12 formatted client certificate. 