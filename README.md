# IBM Cloud AirGapped Cluster with GitOps Configuration

The IBM Go To Market Assets and Architecture team has developed automation that will allow you to deploy an AirGapped Cluster with Gitops Configuration in [TechZone](https://techzone.ibm.com/collection/production-deployment-guides#tab-6) based on the principles outlined in our [Cloud Pak Production Deployment Guides](https://production-gitops.dev/infrastructure/restricted-networks/).

## Reference Architecture

![Reference Architecture](./media/refarch.svg)

---

Leveraging a pre-existing [IBM Cloud VPC Network](https://cloud.ibm.com/docs/vpc), we deploy a Bastion Host into a subnet with a public gateway, and a Private Image Registry into a subnet with no outbound access.  Only traffic allowed between the two subnets is SSH and HTTPS traffic.

On the Bastion Host, a 1TB disk is mounted in /data for any post-deployment activities you may need to perform.  Although SSH access from the bastion server to the image registry server is allowed, we encourage you not to SSH into the image server directly, as this would not mimic a production environment.

On the Registry Server, we deploy [Harbor](https://goharbor.io/) to provide the private registry, and [Gitea](https://gitea.io/) to provide a private Github repository, prepopulated with the [Multi-Tenancy Gitops Framework](https://github.com/cloud-native-toolkit/multi-tenancy-gitops) from the [Cloud Native Toolkit](https://cloudnativetoolkit.dev/).

Finally, an OpenShift cluster is deployed into another set of subnets with no public gateways, and so the cluster will have no outbound access.

A VPN Server is deployed in each region to provide access to the OpenShift Cluster, as well as the UIs of the OpenShift Cluster, Image Registry and GitHub server.



## Welcome Email

After the environment is deployed from TechZone, you will receive an email from TechZone with your environment details.  The email will include:
  - Next Steps:  A link to this repository
  - Bastion Host and Registry Host IP Addresses
  - Bastion Host SSH Key
  - Details of your OpenShift Cluster, with links to the IBM Cloud Cluster page
  - Harbor URL and Login Credentials
  - Gitea URL and Login Credentials
  - ArgoCD URL and Login Credentials
  - VPN Configuration to access the environment

Some fields in this email will be unformated.  You will be able to download the SSH Key and VPN Configuration by going to the [My Reservations](https://techzone.ibm.com/my/reservations) page in TechZone, clicking your provisioned environment, and clicking on the `Download SSH Key` and `Download OpenVPN Config` buttons.

