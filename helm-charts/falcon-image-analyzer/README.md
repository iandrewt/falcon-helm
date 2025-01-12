# CrowdStrike Falcon Image Analyzer (IAR Image Assessment at Runtime) Helm Chart

[Falcon](https://www.crowdstrike.com/) is the [CrowdStrike](https://www.crowdstrike.com/)
platform purpose-built to stop breaches via a unified set of cloud-delivered
technologies that prevent all types of attacks — including malware and much
more.

## Kubernetes cluster compatability

The Falcon Image Analyzer Helm chart has been tested to deploy on the following Kubernetes distributions:

* Amazon Elastic Kubernetes Service (EKS) - EKS and EKS Fargate
* Azure Kubernetes Service (AKS)
* Google Kubernetes Engine (GKE)
* SUSE Rancher K3s
* Red Hat OpenShift Kubernetes

## New updates in curent release
- Removed the `crowdstrikeConfig.agentRunmode` variable from values.
- added `privateRegistries.credentials` variable in values. Details below.

## Dependencies

1. Requires a x86_64 Kubernetes cluster
1. Before deploying the Helm chart, you should have the `falcon-imageanalyzer` container image in your own container registry, or use CrowdStrike's registry before installing the Helm chart. See the [Deployment Considerations](#deployment-considerations) for more.
1. Helm 3.x is installed and supported by the Kubernetes vendor.

## Installation

### Add the CrowdStrike Falcon Helm repository

```
helm repo add crowdstrike https://crowdstrike.github.io/falcon-helm
```

### Update the local Helm repository cache

```
helm repo update
```

## Falcon configuration options

The following tables list the Falcon sensor configurable parameters and their default values.

| Parameter                                                              | Description                                                                                                                                                    | Default                                                                                |
|:-----------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------|
| `deployment.enabled`   required                                        | Set to `true` if running in Watcher Mode i.e.                                                                                                                  | false                                                                                  |
| `daemsonset.enabled`     required                                      | Set to `true` if running in Socket Mode i.e. Both CANNOT be true . This  causes the IAR to run in `socket` mode                                                | false                                                                                  |
| `privateRegistries.credentials`  optional                              | Use this param to provide the comma separated registry secrets of the form namsepace1:secretname1,namespace:secret2                                            | ""                                                                                     |
| `image.repo`       required                                            | IAR image repo name                                                                                                                                            | `[CROWDSTREIKE_IMAGE_REGISTRY]/falcon-imageanalyzer/us-1/release/falcon-imageanalyzer` |
| `image.tag`        required                                            | Image tag version                                                                                                                                              | None                                                                                   |
| `image.registryConfigJSON`        optional                             | iar private registry secret in docker config format                                                                                                            | None                                                                                   |
| `azure.enabled`         optional                                       | Set to `true` if cluster is Azure AKS or self-managed on Azure nodes.                                                                                          | false                                                                                  |
| `azure.azureConfig`          optional                                  | Azure  config file path                                                                                                                                        | `/etc/kubernetes/azure.json`                                                           |
| `gcp.enabled`                  optional                                | Set to `true` if cluster is Gogle GKE or self-managed on Google Cloud GCP nodes.                                                                               | false                                                                                  |
| `crowdstrikeConfig.clusterName`     required                           | Cluster name                                                                                                                                                   | None                                                                                   |
| `crowdstrikeConfig.enableDebug`   optional                             | Set to `true` for debug level log verbosity.                                                                                                                   | false                                                                                  |
| `crowdstrikeConfig.clientID`    required                               | CrowdStrike Falcon OAuth API Client ID                                                                                                                         | None                                                                                   |
| `crowdstrikeConfig.clientSecret`     required                          | CrowdStrike Falcon OAuth API Client secret                                                                                                                     | None                                                                                   |
| `crowdstrikeConfig.cid`         required                               | Customer ID (CID)                                                                                                                                              | None                                                                                   |
| `crowdstrikeConfig.dockerAPIToken`  optional                           | Crowdstrike Artifactory Image Pull Token for pulling IAR image directly from  `[CROWDSTREIKE_IMAGE_REGISTRY] described below`                                  | None                                                                                   |
| `crowdstrikeConfig.existingSecret`      optional                       | Existing secret ref name of the customer Kubernetes cluster                                                                                                    | None                                                                                   |
| `crowdstrikeConfig.agentRegion`      required                          | Region of the CrowdStrike API to connect to us-1/us-2/eu-1                                                                                                     | None                                                                                   |
| `crowdstrikeConfig.agentRuntime`             required ( if daemonset ) | The underlying runtime of the OS. docker/containerd/podman/crio. ONLY TO BE USED with `daemonset.enabled` = `true`                                             | None                                                                                   |
| `crowdstrikeConfig.agentRuntimeSocket`              optional           | The unix socket path for the runtime socket. For example: `unix///var/run/docker.sock`. ONLY TO BE USED with ONLY TO BE USED with `daemonset.enabled` = `true` | None                                                                                   |



The `[CROWDSTREIKE_IMAGE_REGISTRY]` can be replaced with below registries based on the environment ( `agentRegion` )

- `us-1 or us-2 or eu-1` = `registry.crowdstrike.com`
- `gov-1` = `registry.laggar.gcw.crowdstrike.com`
- `gov-2` = `registry.us-gov-2.crowdstrike.mil`

Note:
-
- Please set either `daemonset.enabled` OR `deployment.enabled`

- For deployment the replica count is set to **1** always. this is because IAR is not a load balanced service i.e. increasing replicas will not divide the work but rather duplicate creating unncessary resource consumption.

- For ease of installation and avoid complication the recommended way to install IAR creating a `config.yaml` file at some path like below

For deployment
```
deployment:
  enabled: true


#optional. Use If in EKS / or EC2 required Roles. See Section IAM Roles fopr more details
serviceAccount:
  # Annotations to add to the service account
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::532730071073:role/svc-devtest-cwpp-oidc-eks


#optional. Use if target registries are private with secret. See section Authentication for Private Registries for more details
privateRegistries
  credentials

image:
  repository: "[CROWDSTREIKE_IMAGE_REGISTRY]/falcon-imageanalyzer/us-1/release/falcon-imageanalyzer"
  tag: 1.0.3

  # OPTIONAL
  # Value must be base64. This setting conflicts with image.pullSecret
  # The base64 encoded string of the docker config json for the pull secret can be
  # gotten through:
  # $ cat ~/.docker/config.json | base64 -
  registryConfigJSON:
crowdstrikeConfig:
  clientID: "xxxxxxxxxxx"
  clientSecret: "yyyyyyyyyyyy"
  clusterName: my-test-cluster
  agentRegion: us-1 or us-2 or eu-1 or gov-1 or gov-2
  cid: MYCID-XY
  dockerAPIToken: asdfsfsdfsfsd ( Crowdstrike Artifacotry Token for IAR Image )

```

for daemonset
```
daemonset:
  enabled: true



#optional. Use If in EKS / or EC2 required Roles. See Section IAM Roles fopr more details
serviceAccount:
  # Annotations to add to the service account
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::532730071073:role/svc-devtest-cwpp-oidc-eks


#optional. Use if target registries are private with secret. See section Authentication for Private Registries for more details
privateRegistries
  credentials

image:
  repository: "[CROWDSTREIKE_IMAGE_REGISTRY]/falcon-imageanalyzer/us-1/release/falcon-imageanalyzer"
  tag: 1.0.3

  # OPTIONAL
  # Value must be base64. This setting conflicts with image.pullSecret
  # The base64 encoded string of the docker config json for the pull secret can be
  # gotten through:
  # $ cat ~/.docker/config.json | base64 -
  registryConfigJSON:

crowdstrikeConfig:
  clientID: "xxxxxxxxxxx"
  clientSecret: "yyyyyyyyyyyy"
  clusterName: my-test-cluster
  agentRegion: us-1 or us-2 or eu-1 or gov-1 or gov-2
  agentRuntime: containerd or crio or podman or docker
  cid: MYCID-XY
  dockerAPIToken: asdfsfsdfsfsd ( Crowdstrike Artifacotry Token for IAR Image )

```

If the IAR image is already pulled in advance and pushed to another customer private registry then use that in place
of `[CROWDSTREIKE_IMAGE_REGISTRY]` and the secret for that should be passed in the
`image.registryConfigJSON` with explanation above and `crowdstrikeConfig.dockerAPIToken` should NOT be used

## Installing on Kubernetes cluster nodes



### Deployment considerations

For a successful deployment, you will want to ensure that:
1. By default, the Helm chart installs in the `default` namespace. Best practices for deploying to Kubernetes is to create a new namespace. This can be done by adding `--create-namespace -n falcon-image-analyzer` to your `helm install` command. The namespace can be any name that you wish to use.
1. You must be a cluster administrator to deploy Helm charts to the cluster.
1. CrowdStrike's Helm chart is a project, not a product, and released to the community as a way to automate sensor deployment to Kubernetes clusters. The upstream repository for this project is [https://github.com/CrowdStrike/falcon-helm](https://github.com/CrowdStrike/falcon-helm).

### Pod Security Standards

Starting with Kubernetes 1.25, Pod Security Standards will be enforced. Setting the appropriate Pod Security Standards policy needs to be performed by adding a label to the namespace. Run the following command, and replace `my-existing-namespace` with the namespace that you have installed the falcon sensors, for example: `falcon-image-analyzer`.
```
kubectl label --overwrite ns my-existing-namespace \
  pod-security.kubernetes.io/enforce=privileged
```

If you want to silence the warning and change the auditing level for the Pod Security Standard, add the following labels:
```
kubectl label ns --overwrite my-existing-namespace pod-security.kubernetes.io/audit=privileged
kubectl label ns --overwrite my-existing-namespace pod-security.kubernetes.io/warn=privileged
```

### IAM Roles  ( EKS or Partially Managed using EC2 Instances)
- For the IAR to detect cloud as AWS it should be able to retrieve sts token to assume role to retrieve ECR Tokens.
  There are 2 options for  that . If your EKS cluster us using the **kiam** or **kube2iam** admission controller, add annotations
  for the IAR service account in the values.yaml as stated below, before installing. Make sure the roles have trust-relationship to allow
  the serviceaccount in the `falcon-image-analyzer` namespace
```
serviceAccount:
  # Annotations to add to the service account
  annotations:
    iam.amazonaws.com/role: role-name-with-s2sassume-role-permission -> NOTE That is role name ONLY Not the full ARN
```

Make sure the above role `role-name-with-s2sassume-role-permission` in **AWS** has the as policy with ECR all permissions as IAR will need to pull images and assume ECR Tokens
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecr:*"
            ],
            "Resource": "*"
        }
    ]
}
```
The above role is important so that IAR can read/pull/list from all ECR registries if any workload is launched with an image from any ECR.
Modify the resource part of the role above to restrict to specific registry or AWS Account. Keep the actions as atleast get* and gist*.
Consult the AWS IAM Role Guide/Wizard for syntax and avoid typos.

Make sure the trust-relationship of the has  principal role of `kiam` or `kube2iam` service with `s2s:assumeRole` permissions.

- For the EKS Cluster using the OIDC providers add the annotation as below.Make sure the roles have trust-relationship to allow
  the serviceaccount in the `falcon-image-analyzer` namespace


```
serviceAccount:
  # Annotations to add to the service account
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::111122223333:role/my-role
```

Make sure the above role `arn:aws:iam::111122223333:role/my-role` in **AWS** has the as policy with ECR all permissions as IAR will need to pull images and assume ECR Tokens
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecr:*"
            ],
            "Resource": "*"
        }
    ]
}
```

and a trust-relationship as
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "EKS-OIDC-ARN"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "EKS-OIDC-ARN:aud": "sts.amazonaws.com",
                    "EKS-OIDC-ARN:sub": "system:serviceaccount:falcon-image-analyzer:imageanalyzer-falcon-image-analyzer"
                }
            }
        }
    ]
}
```

Here `falcon-image-analyzer` is the namespace of IAR and `imageanalyzer-falcon-image-analyzer` is the name of the iar Service Account

### Authentication for Private Registries
- If you are using ECR or cloud based Private Registries then assigning the IAM role to the iar service-account in `falcon-image-analyzer` namespace should be enough

- If you are using a 3rd party private registry such as jfrog artifactory for running all your workload images, etc then use the below param in the values.yaml
```
privateRegistries:
  credentials: ""
```
to provide the comma separated registry secrets of the form `"namsepace1:secretname1,namespace:secret2"`
each secret should be of type docker-registry for each of the private registry that is used.
for e.g.  a docker-registry secret can be created as below
```
 kubectl create secret docker-registry regcred \
--docker-server=my-artifactory.jfrog.io \
--docker-username=read-only \
--docker-password=my-super-secret-pass \
--docker-email=johndoe@example.com  -n my-app-ns

 kubectl create secret docker-registry regcred2 \
--docker-server=my2ndregistry-artifactory.jfrog.io \
--docker-username=2nd-read-only \
--docker-password=2nd-my-super-secret-pass \
--docker-email=johndoe@example.com  -n my-app-ns

```
use the above secret as `"my-app-ns:regcred,my-app-ns:regcred2"`

### Install CrowdStrike Falcon Helm chart on Kubernetes nodes

Before you install IAR, set the Helm chart variables and add them to the `values.yaml` file. Then, run the following to install IAR:

```
helm upgrade --install -f path-to-my-values.yaml \
      --create-namespace -n falcon-image-analyzer imageanalyzer crowdstrike/falcon-image-analyzer
```


For more details, see the [falcon-helm](https://github.com/CrowdStrike/falcon-helm) repository.

```
helm show values crowdstrike/falcon-image-analyzer
```

## Uninstall Helm chart

To uninstall, run the following command:
```
helm uninstall imageanalyzer -n falcon-image-analyzer && kubectl delete namespace falcon-image-analyzer
```
