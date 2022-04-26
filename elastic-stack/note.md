- 新建 namespace elk，并添加权限

```
oc adm policy add-scc-to-user privileged -z default -n elk
```

- tls 配置

```
参考网址 ：https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-basic-setup.html

# 生成 CA
./bin/elasticsearch-certutil ca
# 生成证书
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12

# 添加配置，elastic-certificates.p12 的目录更改，放在 PV 中
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.client_authentication: required
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12

```

- 创建角色

```

POST \_security/role/logstash_writer
{
"cluster": ["manage_index_templates", "monitor", "manage_ilm"],
"indices": [
{
"names": [ "logstash-*","applog" ],
"privileges": ["write","create","create_index","manage","manage_ilm"]
}
]
}

```

- 创建用户

```

POST \_security/user/logstash_internal
{
"password" : "x-pack-test-password",
"roles" : [ "logstash_writer"],
"full_name" : "Internal Logstash User"
}

```

- 添加配置

```

input {
elasticsearch {
...
user => logstash_internal
password => x-pack-test-password
}
}
filter {
elasticsearch {
...
user => logstash_internal
password => x-pack-test-password
}
}
output {
elasticsearch {
...
user => logstash_internal
password => x-pack-test-password
}
}

```

```

```

```

```
