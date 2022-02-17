## Installation steps

1. Generate the CA cert and key for autogeneration of certificates.

```bash
kubectl create secret tls ca-pair-sslcerts \
  --cert=$TUTORIAL_HOME/../../assets/certs/generated/ca.pem \
  --key=$TUTORIAL_HOME/../../assets/certs/generated/ca-key.pem \
  --namespace confluent
```

2. Create authentication credentials secret.

```bash
kubectl create secret generic credential \
  --from-file=plain-users.json=$TUTORIAL_HOME/creds-kafka-sasl-users.json \
  --from-file=digest-users.json=$TUTORIAL_HOME/creds-zookeeper-sasl-digest-users.json \
  --from-file=digest.txt=$TUTORIAL_HOME/creds-kafka-zookeeper-credentials.txt \
  --from-file=plain.txt=$TUTORIAL_HOME/creds-client-kafka-sasl-user.txt \
  --from-file=basic.txt=$TUTORIAL_HOME/creds-control-center-users.txt \
  --from-file=ldap.txt=$TUTORIAL_HOME/ldap.txt \
  --namespace confluent
```

3. Create MDS related artifacts.

```bash
kubectl create secret generic mds-token \
  --from-file=mdsPublicKey.pem=$TUTORIAL_HOME/../../assets/certs/mds-publickey.txt \
  --from-file=mdsTokenKeyPair.pem=$TUTORIAL_HOME/../../assets/certs/mds-tokenkeypair.txt \
  --namespace confluent
# Kafka RBAC credential
kubectl create secret generic mds-client \
  --from-file=bearer.txt=$TUTORIAL_HOME/bearer.txt \
  --namespace confluent
# Control Center RBAC credential
kubectl create secret generic c3-mds-client \
  --from-file=bearer.txt=$TUTORIAL_HOME/c3-mds-client.txt \
  --namespace confluent
# Connect RBAC credential
kubectl create secret generic connect-mds-client \
  --from-file=bearer.txt=$TUTORIAL_HOME/connect-mds-client.txt \
  --namespace confluent
# Schema Registry RBAC credential
kubectl create secret generic sr-mds-client \
  --from-file=bearer.txt=$TUTORIAL_HOME/sr-mds-client.txt \
  --namespace confluent
# ksqlDB RBAC credential
kubectl create secret generic ksqldb-mds-client \
  --from-file=bearer.txt=$TUTORIAL_HOME/ksqldb-mds-client.txt \
  --namespace confluent
# Kafka REST credential
kubectl create secret generic rest-credential \
  --from-file=bearer.txt=$TUTORIAL_HOME/bearer.txt \
  --from-file=basic.txt=$TUTORIAL_HOME/bearer.txt \
  --namespace confluent
```

4. Deploy confluent platform.

```bash
kubectl apply -f platform.yml -n confluent
```

5. Deploy role bindings.

```bash
kubectl apply -f crb.yml -n confluent
```

## Troubleshooting/Gotchas

Cluster Rolebindings configured for a cluster need to be deleted explicitly once the cluster is deleted. This can be done by the following command:

```bash
for rb in $(kubectl -n confluent get cfrb --no-headers -ojsonpath='{.items[*].metadata.name}'); do kubectl -n confluent  patch cfrb $rb -p '{"metadata":{"finalizers":[]}}' --type=merge; done
```

Please change the namespace from confluent to your appropriate namespace in the above command.

## TODO

1. Add connector
2. Add and expose schema registry.
3. Add services and ingresses
4. Instructions for producer, consumer, connector and ksqldb.
5. Adding/removing/updating new users.
6. Production cluster sizing recommendations
7. Reconfigure flux with Azure china registries.
8. Reconfigure nginx with Azure china registries.