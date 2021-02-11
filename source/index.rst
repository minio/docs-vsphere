=====================================
MinIO for VMware Cloud Foundation 4.2
=====================================

.. default-domain:: minio

Welcome to the MinIO Object Storage documentation for 
`VMware Cloud Foundation 4.2 
<https://core.vmware.com/vmware-cloud-foundation?jmp=minio-docs>`__.

MinIO is a Kubernetes-native object store designed to provide high performance
with an S3-compatible API. |vcf| 4.2 includes MinIO's 
vCenter plugin for provisioning multi-tenant object storage on VMware Tanzu
Kubernetes infrastructure. See 
`MinIO Object Storage on VMware Cloud Foundation with Tanzu 
<https://core.vmware.com/resource/minio-object-storage-vmware-cloud-foundation-tanzu>`__
for additional information.

This documentation assumes familiarity with all referenced Kubernetes
and VMware concepts, utilities, and procedures. While this documentation *may* 
provide guidance for configuring or deploying non-MinIO resources 
on a best-effort basis, it is not a replacement for the official
:kube-docs:`Kubernetes Documentation <>` or 
:vmware-docs:`VMware Documentation <index.html>`.

This documentation summarizes core features and functionality of MinIO Object
Storage. For more complete documentation, see the documentation for
:docs-baremetal:`MinIO Object Storage for Baremetal <>`.

Getting Started
---------------

Use the :doc:`/tutorials/deploy-minio-tenant` guide to create a 
MinIO tenant on using VMware vCenter and the MinIO plugin.

For managing an existing MinIO tenant, see 
:doc:`/tutorials/manage-minio-tenant`.

.. _minio-vcenter-inside-tenant:

Inside a MinIO Tenant
---------------------

A MinIO tenant is a :kube-docs:`StatefulSet 
<concepts/workloads/controllers/statefulset/>` object that represents a 
MinIO deployment. Each tenant consists of 
:kube-docs:`Pods <concepts/workloads/pods/>` and 
:kube-docs:`Services <concepts/services-networking/service/>` that are 
automatically deployed and managed by the MinIO plugin for |vcf|.

Each tenant consists of the following core components:

Zone
   A set of MinIO server pods which pool their drives
   for supporting object storage and retrieval requests. A MinIO tenant can have
   an unlimited number of Zones, where each new Zone expands the total available
   storage on the cluster.

   The :guilabel:`Zone` terminology is unique to MinIO on |vcf| and 
   is equivalent in usage and behavior to :guilabel:`Server Pools` or 
   :guilabel:`Pools` as referenced in other MinIO documentation properties.

Erasure Set
   A set of drives that support MinIO :ref:`Erasure Coding
   <minio-erasure-coding>`. Erasure coding provides high availability,
   reliability, and redundancy of data stored on the tenant.

   Each Zone has at least one Erasure Set, where MinIO automatically 
   calculates the optimal Erasure Set size for the zone. See 
   :ref:`minio-erasure-coding` for more information.
   
   

.. toctree::
   :titlesonly:
   :hidden:

   /core-concepts/core-concepts
   /tutorials/deploy-minio-tenant
   /tutorials/manage-minio-tenant
   /tutorials/monitor-minio-tenant
   /security/security
   /reference/production-recommendations


