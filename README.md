# IBM Cloud AirGapped Cluster with GitOps Configuration

The IBM Go To Market Assets and Architecture team has developed automation that will allow you to deploy an AirGapped Cluster with Gitops Configuration in [TechZone](https://techzone.ibm.com/collection/production-deployment-guides#tab-6) based on the principles outlined in our [Cloud Pak Production Deployment Guides](https://production-gitops.dev/infrastructure/restricted-networks/).

## Reference Architecture

![Reference Architecture](./media/refarch.svg)

---

Leveraging pre-existing infrastructure in IBM Cloud, we deploy a Bastion Host into a subnet with a public gateway, and a Private Image Registry into a subnet with no outbound access.  Only traffic allowed between the two subnets is SSH and HTTPS traffic.

On the Bastion Host, a 1TB disk is mounted in /data for any post-deployment activities you may need to perform.  Although SSH access from the bastion server to the image registry server is allowed, we encourage you not to SSH into the image server directly, as this would not mimic a production environment.


We leverage the [IBM Cloud VPC Network](https://cloud.ibm.com/docs/vpc) to provide a secure, isolated, and isolated network for the AirGapped Cluster.  A VPN Server is deployed in each region to provide access to the OpenShift Cluster, as well as the UIs

### OpenShift Cluster
