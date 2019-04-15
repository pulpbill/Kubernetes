# efs-provisioner 

The efs-provisioner allows you to mount EFS storage as PersistentVolumes in kubernetes. It consists of a container that has access to an AWS EFS resource. The container reads a configmap which contains the EFS filesystem ID, the AWS region and the name you want to use for your efs-provisioner. This name will be used later when you create a storage class.

## Prerequisites:
. An EFS file system in your cluster's region (replace efs-id at full-deployment.yaml)

. Mount targets and security groups such that any node (in any zone in the cluster's region) can mount the EFS file system by its File system DNS name

Reference: https://github.com/kubernetes-incubator/external-storage/tree/master/aws/efs
