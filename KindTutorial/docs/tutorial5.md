# Tutorial 5 - Deploying Vault

HashiCorp Vault will be used as a solution to to manage the storage of sensitive resources including the credentials for accessing MySQL by the python application.

## Deploy Vault

First, create a new namespace called `vault` that will be used to contain the resources associated with the deployment of vault.

```shell
kubectl apply -f $TUTORIAL_ROOT/cloudnativesecuritycon-workload-identity-tutorial/KindTutorial/resources/secure/vault/namespace.yaml
```

Deploy Vault to the `vault` namespace. Ensure the `APP_DOMAIN` environment variable is still defined so that the host property of the `Ingress` can be configured appropriately. 

```shell
envsubst < $TUTORIAL_ROOT/cloudnativesecuritycon-workload-identity-tutorial/KindTutorial/resources/secure/vault/vault.yaml | kubectl apply -f - 
```

Confirm Vault is running in the `vault` namespace. 

```shell
kubectl get pods -n vault
```

Now let's port-forward the Vault. Open a new terminal and run the following:

```shell
kubectl port-forward -n vault svc/vault 8200:8200 2>&1 >/dev/null &
```

Next, obtain the location of the Vault instance and set the `VAULT_ADDR` environment variable by executing the following command:

```shell
export VAULT_ADDR=http://localhost:8200
```

Confirm that the Vault health endpoint returns a successful response

```
curl  $VAULT_ADDR/v1/sys/health | jq
```

A response similar to the following indicates Vault is running and healthy

```json
{
  "initialized": true,
  "sealed": false,
  "standby": false,
  "performance_standby": false,
  "replication_performance_mode": "disabled",
  "replication_dr_mode": "disabled",
  "server_time_utc": 1719336763,
  "version": "1.7.0",
  "cluster_name": "vault-cluster-0674246f",
  "cluster_id": "7c5d50b2-1288-02dd-9094-bda8ca7f72c1"
}
```

## Configure Vault with SPIRE

Once the deployment of Vault has been validated, the next step is to enable the use of several of the capabilities provided by Vault:

* [JWT/OIDC Authentication](https://developer.hashicorp.com/vault/docs/auth/jwt) - Enables authenticating to Vault using a JWT provided by SPIRE
* [Policies](https://developer.hashicorp.com/vault/docs/concepts/policies) - Grant access to read sensitive values stored in Vault
* Roles - Mapping between a policy and an authenticated entity

Obtain a token for accessing the Vault instance by setting the `ROOT_TOKEN` environment variable which will be used by the [Vault CLI](https://developer.hashicorp.com/vault/docs/commands).

```shell
export ROOT_TOKEN=$(kubectl -n vault logs $(kubectl -n vault get po | grep vault-| awk '{print $1}') | grep Root | cut -d' ' -f3); echo "export ROOT_TOKEN=$ROOT_TOKEN"
```

Login to Vault using the CLI

```shell
vault login -no-print "${ROOT_TOKEN}"
```

A script is available to automate the configuration of Vault to support accessing resources in Vault. Specifically, the script performs the following activities:
 
1. Enables JWT authentication using the SPIRE OIDC provider to enable Kubernetes workloads the ability to login to Vault
2. Creates a Policy allowing read access to contents stored within the `secret/data/db-config/*` Vault path
3. Creates a Role that enables workloads running in the `workload-identity-tutorial` namespace the ability to authenticate to vault and access resources defined by the aforementioned Policy

Enable these configurations by executing the script called `vault-oidc.sh`

```shell
$TUTORIAL_ROOT/cloudnativesecuritycon-workload-identity-tutorial/KindTutorial/resources/secure/vault/vault-oidc.sh
```

Confirm the policy called `dbpolicy` has been pushed to Vault:

```shell
vault policy read dbpolicy
```

Next, verify JWT authentication is included in the list of enabled providers with the Path `jwt`:

```shell
vault auth list
```

Finally, confirm a role called `dbrole` has been created and associated to the jwt authentication provider

```shell
vault read auth/jwt/role/dbrole
```

If the above set of resources were found in Vault, then the instance is ready for us

## Storing Credentials in Vault

Now that Vault has been deployed and configured, Kaya recommends transitioning from storing the database credentials within a `ConfigMap` stored in Kubernetes to Vault.

First, obtain the configuration file containing the database credentials from the ConfigMap called `db-config`:

```shell
kubectl get cm -n workload-identity-tutorial db-config -o jsonpath='{ .data.config\.ini }' > $TUTORIAL_ROOT/cloudnativesecuritycon-workload-identity-tutorial/KindTutorial/resources/secure/vault/config.ini
```

Vault accepts configurations that are stored in JSON format and since this configuration resource is in `.ini` format instead, the contents can be encoded in base64 format and stored within Vault.

Encode the contents of the `config.ini` file in a variable called `SHA64`

```shell
SHA64=$(openssl base64 -in $TUTORIAL_ROOT/cloudnativesecuritycon-workload-identity-tutorial/KindTutorial/resources/secure/vault/config.ini)
```

Utilize the [KV Secrets Engine](https://developer.hashicorp.com/vault/docs/secrets/kv) within vault to securely store the configuration:

```shell
vault kv put secret/db-config/config.ini sha="$SHA64"
```

Confirm the secret can be retrieved from Vault

```shell
vault kv get -field=sha secret/db-config/config.ini | openssl base64 -d
```

[Previous Tutorial - Deploying SPIRE](tutorial4.md)

[Next Tutorial - Accessing Resources Stored in Vault using SPIRE](tutorial6.md)

[Home](../README.md)
