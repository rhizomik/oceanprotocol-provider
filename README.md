# Ocean Protocol Compute to Data with Helm

## Documentation

Official [Helm documentation](https://helm.sh/docs/)

Official Ocean Protocol documentation of [Compute to data infrastructure](https://docs.oceanprotocol.com/infrastructure/compute-to-data-minikube) list of available container images:

 - Provider:
    - repository: oceanprotocol/provider-py
    - tags: latest / v2.1.6
 - API:
    - repository: oceanprotocol/operator-service
    - tag: v4main / latest
 - Operator Engine:
    - repository: oceanprotocol/operator-engine
    - tags: latest / v1.1.2 / v4main / nvidia / nvidia-test
 - POD Configuration:
    - repository: oceanprotocol/pod-configuration
    - tags: sanitize-url / v4main / v1.1.1 / latest
  - Operator Publishing:
    - repository: oceanprotocol/pod-publishing
    - tag: v4main

[OceanProtocol Provider on Kubernetes](https://github.com/rogargon/oceanprotocol-provider-kubectl/) by Roberto Garc&iacute;a list container image used:

 - Provider:
   - repository: oceanprotocol/provider-py
   - tags: v2.1.3
 - API:
   - repository: oceanprotocol/operator-service
   - tag: v4main
 - Operator Engine:
   - repository: rodargon/operator-engine
   - tag: gke-gpu
 - POD Configuration:
   - repository: rodargon/pod-configuration
   - tag: timeout
 - Operator Publishing:
   - repository: oceanprotocol/pod-publishing
   - tag: v4main

## Directory structure

```plain
oceanprotocol-provider
├── charts
│   ├── ipfs
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── deployment.yaml
│   │   │   ├── ingress.yaml
│   │   │   ├── pvc.yaml
│   │   │   └── service.yaml
│   │   └── values.yaml
│   ├── operator-api
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── deployment.yaml
│   │   │   └── service.yaml
│   │   └── values.yaml
│   ├── operator-engine
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── deployment.yaml
│   │   │   ├── role-binding.yaml
│   │   │   ├── role.yaml
│   │   │   └── service-account.yaml
│   │   └── values.yaml
│   ├── postgres
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── deployment.yaml
│   │   │   ├── pvc.yaml
│   │   │   └── service.yaml
│   │   └── values.yaml
│   └── provider
│       ├── Chart.yaml
│       ├── templates
│       │   ├── deployment.yaml
│       │   ├── ingress.yaml
│       │   └── service.yaml
│       └── values.yaml
├── Chart.yaml
├── LICENSE
├── README.md
├── templates
│   ├── NOTES.txt
│   └── secrets.yaml
└── values.yaml

12 directories, 32 files
```

## Install

### Create a repository

Add a new repository with this chart:

```console
helm repo add oceanprotocol-provider https://arsys-internet.github.io/oceanprotocol-provider/
```

If everything works fine you should get something like this:

```console
"oceanprotocol-provider" has been added to your repositories
```

### Personalize the deployment

Create a values file adapted to your kubernetes cluster configuration:

```yaml
# Default values for the deployment of oceanprotocol-provider in Arsys DCD Managed Kubernetes.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

networks:
  networksURL: |
    { "100": "https://rpc.genx.minimal-gaia-x.eu",
      "32456": "https://rpc.dev.pontus-x.eu",
      "32457": "https://rpc.test.pontus-x.eu" }
  privateProviders: |
    { "100": "0x0...",
      "32456": "0x0...",
      "32457": "0x0..." }
  publicProviders: |
    [ "0x0...",
    [ "0x0...",
    [ "0x0..." ]
  privateOperator: "0x0..."
  publicOperator: "0x0..."

ipfs:
  storage:
    classname: "ionos-enterprise-hdd"
  ingress:
    annotations:
      cert-manager.io/cluster-issuer: letsencrypt-prod
    host: "ipfs.ocean.arlabdevelopments.com"
    tls: true
    secretName: ipfs-tls

postgres:
  storage:
    classname: "ionos-enterprise-ssd"

provider:
  image:
    tag: "v2.1.3"
  ipfsGateway: "https://ipfs.ocean.arlabdevelopments.com"
  providerFeeToken: |
     { "100": "0x0995527d3473b3a98c471f1ed8787acd77fbf009", 
      "32456": "0x8a4826071983655805bf4f29828577cd6b1ac0cb",
      "32457": "0xdd0a0278f6BAF167999ccd8Aa6C11A9e2fA37F0a" }
  aquariusURL: "https://aquarius.pontus-x.eu/"
  ingress:
    annotations:
      cert-manager.io/cluster-issuer: letsencrypt-prod
    host: "provider.ocean.arlabdevelopments.com"
    tls: true
    secretName: provider-tls

operator-engine:
  image:
    repository: "rogargon/operator-engine"
    tag: "gke-gpu"
  description: "ArsysLab Data Room"
  jobStorageClassname: "ionos-enterprise-hdd"
  priceMinute: "0"
  ipfsOutputPrefix: "https://ipfs.ocean.arlabdevelopments.com/ipfs/"
  ipfsAdminLogsPrefix: "https://ipfs.ocean.arlabdevelopments.com/ipfs/"
  podConfContainer: "rogargon/pod-configuration:timeout"
```

### Deploy the provider

Once the values are adjusted to our needs, install the provider using this command:

```console
helm upgrade --install --namespace dataspace --create-namespace --values ./oceanprotocol-arsys.yaml arsys-c2d oceanprotocol-provider/oceanprotocol-provider
```

If everything works fine you should get something like this:

```plain
Release "arsys-c2d" does not exist. Installing it now.
NAME: arsys-c2d
LAST DEPLOYED: Fri Jul  5 13:07:23 2024
NAMESPACE: dataspace
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing oceanprotocol-provider.

The published services for arsys-c2d are:

* IPFS: https://ipfs.domain/
* Provider: https://provider.domain/

If it is a new install, and not an upgrade, wait until everything is `Running`
to initialise the PostgreSQL database with the following command:

$ kubectl run --namespace dataspace --attach --rm --restart=Never \
  --image curlimages/curl pgsqlinit -- \
  curl -X POST -H "accept: application/json" \
    -H "Admin: postgresadmin" \
    "http://arsys-c2d-operator-api.dataspace:8050/api/v1/operator/pgsqlinit"

Once DB initialization it's done, then you can list the computational 
environments available using this URL:
  https://provider.domain/api/services/computeEnvironments
```

After the first deployment ensure to launch de database initialization command shown on the notes of helm.

## Services

For a normal working provider we have the following resources created on kubernetes:

```
$ kubectl get --namespace dataspace --output wide all,jobs,persistentvolumeclaims,persistentvolumes,ingresses,secrets,configmaps,certificates
NAME                                            READY   STATUS    RESTARTS   AGE     IP              NODE                       NOMINATED NODE   READINESS GATES
pod/arsys-c2d-ipfs-84d4956dd9-gxwwf             1/1     Running   0          7m      10.211.46.218   oceanprotocol-dbpc6c4vhq   <none>           <none>
pod/arsys-c2d-operator-api-64cf4fdd9f-97kv9     1/1     Running   0          7m      10.211.46.217   oceanprotocol-dbpc6c4vhq   <none>           <none>
pod/arsys-c2d-operator-engine-cb45cf98c-6m7bg   1/1     Running   0          7m      10.211.46.230   oceanprotocol-dbpc6c4vhq   <none>           <none>
pod/arsys-c2d-postgres-7f6f9ddc4b-cqvzm         1/1     Running   0          7m      10.211.46.219   oceanprotocol-dbpc6c4vhq   <none>           <none>
pod/arsys-c2d-provider-7b4db6567d-w64wv         1/1     Running   0          7m      10.222.91.215   oceanprotocol-sj5zd352ss   <none>           <none>

NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE   SELECTOR
service/arsys-c2d-ipfs           ClusterIP   10.233.47.5     <none>        5001/TCP,8080/TCP   7m    app=arsys-c2d-ipfs
service/arsys-c2d-operator-api   ClusterIP   10.233.10.28    <none>        8050/TCP            7m    app=arsys-c2d-operator-api
service/arsys-c2d-postgres       ClusterIP   10.233.30.179   <none>        5432/TCP            7m    app=arsys-c2d-postgres
service/arsys-c2d-provider       ClusterIP   10.233.42.212   <none>        8030/TCP            7m    app=arsys-c2d-provider

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                  IMAGES                                  SELECTOR
deployment.apps/arsys-c2d-ipfs              1/1     1            1           7m    arsys-c2d-ipfs              ipfs/go-ipfs:latest                     app=arsys-c2d-ipfs
deployment.apps/arsys-c2d-operator-api      1/1     1            1           7m    arsys-c2d-operator-api      oceanprotocol/operator-service:v4main   app=arsys-c2d-operator-api
deployment.apps/arsys-c2d-operator-engine   1/1     1            1           7m    arsys-c2d-operator-engine   rogargon/operator-engine:gke-gpu        app=arsys-c2d-operator-engine
deployment.apps/arsys-c2d-postgres          1/1     1            1           7m    arsys-c2d-postgres          postgres:10.4                           app=arsys-c2d-postgres
deployment.apps/arsys-c2d-provider          1/1     1            1           7m    provider                    oceanprotocol/provider-py:v2.1.3        app=arsys-c2d-provider

NAME                                                  DESIRED   CURRENT   READY   AGE     CONTAINERS                  IMAGES                                  SELECTOR
replicaset.apps/arsys-c2d-ipfs-84d4956dd9             1         1         1       7m      arsys-c2d-ipfs              ipfs/go-ipfs:latest                     app=arsys-c2d-ipfs,pod-template-hash=84d4956dd9
replicaset.apps/arsys-c2d-operator-api-64cf4fdd9f     1         1         1       7m      arsys-c2d-operator-api      oceanprotocol/operator-service:v4main   app=arsys-c2d-operator-api,pod-template-hash=64cf4fdd9f
replicaset.apps/arsys-c2d-operator-engine-cb45cf98c   1         1         1       7m      arsys-c2d-operator-engine   rogargon/operator-engine:gke-gpu        app=arsys-c2d-operator-engine,pod-template-hash=cb45cf98c
replicaset.apps/arsys-c2d-postgres-7f6f9ddc4b         1         1         1       7m      arsys-c2d-postgres          postgres:10.4                           app=arsys-c2d-postgres,pod-template-hash=7f6f9ddc4b
replicaset.apps/arsys-c2d-provider-7b4db6567d         1         1         1       7m      provider                    oceanprotocol/provider-py:v2.1.3        app=arsys-c2d-provider,pod-template-hash=7b4db6567d

NAME                                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS           VOLUMEATTRIBUTESCLASS   AGE   VOLUMEMODE
persistentvolumeclaim/arsys-c2d-ipfs       Bound    pvc-ef15dc6c-3598-424d-a248-541be3b2056e   1Gi        RWO            ionos-enterprise-hdd   <unset>                 7m    Filesystem
persistentvolumeclaim/arsys-c2d-postgres   Bound    pvc-e16f6af6-71fc-4cb9-93c6-d9b2ea8dd1a4   1Gi        RWO            ionos-enterprise-ssd   <unset>                 7m    Filesystem

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                          STORAGECLASS           VOLUMEATTRIBUTESCLASS   REASON   AGE   VOLUMEMODE
persistentvolume/pvc-e16f6af6-71fc-4cb9-93c6-d9b2ea8dd1a4   1Gi        RWO            Delete           Bound    dataspace/arsys-c2d-postgres   ionos-enterprise-ssd   <unset>                          7m    Filesystem
persistentvolume/pvc-ef15dc6c-3598-424d-a248-541be3b2056e   1Gi        RWO            Delete           Bound    dataspace/arsys-c2d-ipfs       ionos-enterprise-hdd   <unset>                          7m    Filesystem

NAME                                           CLASS   HOSTS                                  ADDRESS        PORTS     AGE
ingress.networking.k8s.io/arsys-c2d-ipfs       nginx   ipfs.domain       5.250.184.10   80, 443   7m
ingress.networking.k8s.io/arsys-c2d-provider   nginx   provider.domain   5.250.184.10   80, 443   7m

NAME                                     TYPE                 DATA   AGE
secret/arsys-c2d-networks                Opaque               4      7m
secret/arsys-c2d-postgres                Opaque               6      7m
secret/ipfs-tls                          kubernetes.io/tls    2      7m
secret/provider-tls                      kubernetes.io/tls    2      7m
secret/sh.helm.release.v1.arsys-c2d.v1   helm.sh/release.v1   1      7m

NAME                         DATA   AGE
configmap/kube-root-ca.crt   1      21d

NAME                                       READY   SECRET         ISSUER             STATUS                                          AGE
certificate.cert-manager.io/ipfs-tls       True    ipfs-tls       letsencrypt-prod   Certificate is up to date and has not expired   7m
certificate.cert-manager.io/provider-tls   True    provider-tls   letsencrypt-prod   Certificate is up to date and has not expired   7m
```

## Example with MiniKube cluster

Instructions to deploy [OceanProtocol Provider](https://docs.oceanprotocol.com/developers/provider) on a
[MiniKube](https://minikube.sigs.k8s.io/docs/start/) Kubernetes cluster, installed following the instructions
on the previous link.

After starting the cluster, enable nginx ingress by running:

```shell
minikube addons enable ingress
```

To run locally, configure in `/etc/hosts` the following entry pointing to the IP of the minikube cluster,
which can be obtained by running `minikube ip`. For instance, if the cluster IP is `192.168.64.5`:

```
192.168.64.5    ipfs.local provider.local
```

Once the Kubernetes cluster is ready, it is time to install the Helm chart for the Compute to Data Provider.
First, add the repository with this chart:

```console
helm repo add oceanprotocol-provider https://rhizomik.github.io/oceanprotocol-provider/
```

Then, install it using the provided sample values for a MiniKube deployment in `values-minikube.yaml`:

```shell
helm upgrade --install --namespace dataspace --create-namespace --values ./values-minikube.yaml minikube-ctd oceanprotocol-provider/oceanprotocol-provider
```

Then, follow the instructions on the output of the Helm command. First, to wait until all pods are running, which can
be checked using the indicated command, for instance:

```shell
kubectl get --namespace dataspace pods
```

Second, to initialize the PostgreSQL database using the command also indicated in Helm's output, for instance:

```shell
kubectl run --namespace dataspace --attach --rm --restart=Never \
  --image curlimages/curl pgsqlinit -- \
  curl -X POST -H "accept: application/json" \
    -H "Admin: myAdminSecret" \
    "http://minikube-ctd-operator-api.dataspace:8050/api/v1/operator/pgsqlinit"
```

Finally, to check the provider is running, you can list the computational environments available using the
URL indicated in the output, for instance: http://provider.local/api/services/computeEnvironments
