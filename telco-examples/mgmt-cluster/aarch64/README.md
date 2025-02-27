## Tech-Preview AARCH64 Integration (full e2e, mgmt-cluster and downstream clusters using aarch64 architecture)

This is an example to demonstrate how to deploy a management cluster for Telco using SUSE Edge for Telco (formerly known as ATIP) and the fully automated directed network provisioning using aarch64 architecture.

# Management Cluster in a single-node setup (aarch64)

This is an example of using Edge Image Builder (EIB) to generate a management cluster iso image for SUSE Edge for Telco. The management cluster will contain the following components:
- SUSE Linux Micro 6.0 Kernel (SL Micro 6.0)
- RKE2
- CNI plugins (e.g. Multus, Cilium)
- Rancher Prime
- Neuvector
- Longhorn
- Static IPs or DHCP network configuration
- Metal3 and the CAPI provider
- Aarch64 architecture

You need to modify the following values in the `mgmt-cluster-arm-singlenode.yaml` file:

- `${ROOT_PASSWORD}` - The root password for the management cluster. This could be generated using `openssl passwd -6 PASSWORD` and replacing PASSWORD with the desired password, and then replacing the value in the `mgmt-cluster-singlenode.yaml` file. The final rancher password will be configured based on the file `custom/files/basic-setup.sh`.
- `${SCC_REGISTRATION_CODE}` - The registration code for the SUSE Customer Center for the SLE Micro product. This could be obtained from the SUSE Customer Center and replacing the value in the `mgmt-cluster-arm-singlenode.yaml` file.

You need to modify the following values in the `network/mgmt-cluster-network.yaml` file :

- `${MGMT_GATEWAY}` - This is the gateway IP of your management cluster network.
- `${MGMT_DNS}` - This is the DNS IP of your management cluster network.
- `${MGMT_CLUSTER_IP}` - This is the static IP of your management cluster single node.
- `${MGMT_MAC}` - This is the MAC address of your management cluster node.

You need to modify the `${MGMT_CLUSTER_IP}` with the Node IP in the following files:

- `kubernetes/helm/values/metal3.yaml`

- `kubernetes/helm/values/rancher.yaml`

You need to modify the following values in the `custom/scripts/99-register.sh` file:

- `${SCC_REGISTRATION_CODE}` - The registration code for the SUSE Customer Center for the SL Micro product. This could be obtained from the SUSE Customer Center and replacing the value in the `99-register.sh` file.

- `${SCC_ACCOUNT_EMAIL}` - The email address for the SUSE Customer Center account. This could be obtained from the SUSE Customer Center and replacing the value in the `99-register.sh` file.

You need to modify the following folder:

- `base-images` - To include inside the `SL-Micro.aarch64-6.0-Default-SelfInstall-GM2.install.iso` image downloaded from the SUSE Customer Center.

## Optional modifications

### Add certificates to use HTTPS server to provide images using TLS

This is an optional step to add certificates to the management cluster to provide images using HTTPS Server (Helm Chart metal3 Version >= 0.7.1)

1. Modify the `kubernetes/helm/values/metal3.yaml` file to set to true the following value in the global section:

```yaml
global:
  additionalTrustedCAs: true
```

2. If you are deploying a mgmt-cluster from scratch using EIB, then add the secret to the manifests folder `kubernetes/manifests/metal3-cacert-secret.yaml` to automate the creation of the secret in the management cluster:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: metal3-system
---
apiVersion: v1
kind: Secret
metadata:
  name: tls-ca-additional
  namespace: metal3-system
type: Opaque
data:
  ca-additional.crt: {{ additional_ca_cert | b64encode }}
```

3. Alternatively, you can use the following command to create the secret manually:

```bash
kubectl -n meta3-system create secret generic tls-ca-additional --from-file=ca-additional.crt=./ca-additional.crt
```

where `ca-additional.crt` is the certificate file that you want to use to provide images using HTTPS.

## Building the Management Cluster Image using EIB

1. Clone this repo and navigate to the `telco-examples/mgmt-cluster/aarch64/eib` directory:

```bash
$ git clone https://github.com/suse-edge/atip.git
$ cd telco-examples/mgmt-cluster/aarch64/eib
```

2. Modify the files described above.

3. Run the image building process.

```bash
$ sudo podman run --rm --privileged -it -v $PWD:/eib \
registry.suse.com/edge/3.2/edge-image-builder:1.1.0 \
build --definition-file mgmt-cluster-arm-singlenode.yaml
```

## Deploy the Management Cluster

Once the build process is finished, you will find the modified ISO image in the `eib` directory. You can then proceed to provision a VM or a baremetal server with it.
