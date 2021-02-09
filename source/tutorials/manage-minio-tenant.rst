=====================
Manage a MinIO Tenant
=====================

.. default-domain:: minio

.. contents:: Table of Contents
   :local:
   :depth: 1

Overview
--------

This page documents several common procedures for managing an existing 
MinIO tenant. You can view the list of tenants at any time from the MinIO 
plugin for vSphere.

.. _minio-vcenter-view-tenants:

.. image:: /images/vsphere/minio-tenant-list.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant View

Select the |vcf| cluster and open the :guilabel:`Configure` tab. 
Navigate to the :guilabel:`MinIO` section and click :guilabel:`Tenants` to 
view all MinIO tenants deployed in that cluster.

.. _minio-vcenter-expand-tenant:

Expand a MinIO Tenant
---------------------

Zone expansion increases the total usable storage capacity of the MinIO tenant.
Each :ref:`zone <minio-vcenter-inside-tenant>` consists of an independent 
pool of hosts and their locally-attached storage. MinIO presents all 
zones in the tenant as a single storage resource for applications to access.

Prerequisites
~~~~~~~~~~~~~

MinIO creates new Tenant pods on available ESXi hosts in the cluster.
Each ESXi host *must* have sufficient compute, memory, and direct-attached 
storage to support each pod provisioned as part of the new tenant zone. 
MinIO requires one available ESXi host for each MinIO server pod deployed 
as part of the zone.

All zones in a MinIO tenant use the same storage policy. Ensure each ESXi
host's locally attached drives are included in the storage policy used by the
MinIO tenant.

1) Open the :guilabel:`Add New Zone` Modal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the vCenter interface, select the cluster in which the MinIO tenant you
want to expand is deployed. Click the :guilabel:`Configure` tab, then open the
:guilabel:`MinIO` section and select :guilabel:`Tenants` to open the
:guilabel:`MinIO Tenants` view.

.. image:: /images/vsphere/minio-tenant-list.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO View All Tenants

Click the radio button next to the tenant you want to expand and select
:guilabel:`Details` to open the :guilabel:`Tenant` view.

From the Tenant view, locate the :guilabel:`Zones` section and click
:guilabel:`Add` to open the :guilabel:`Add New Zone` modal.

.. image:: /images/vsphere/minio-tenant-add-zone.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO View All Tenants

2) Complete the :guilabel:`Add New Zone` Modal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. image:: /images/vsphere/minio-tenant-add-new-zone.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO View All Tenants

The :guilabel:`Add New Zone` modal displays the following fields:

.. list-table::
   :stub-columns: 1
   :widths: 30 70

   * - :guilabel:`Number of Nodes`
     - Specify the number of nodes to create for the new MinIO tenant zone. 

   * - :guilabel:`Zone size`
     - Specify the total amount of storage for the new MinIO tenant zone.

   * - :guilabel:`Memory per Node [Gi]`
     - The amount of memory to allocate to each MinIO object storage pod in 
       the zone.

The modal displays the summary of the new zone below the text inputs.

.. list-table::
   :stub-columns: 1
   :widths: 30 70
   :width: 100%

   * - :guilabel:`Nodes`
     - The number of nodes in the zone. 

   * - :guilabel:`Volumes per Node`
     - The number of Persistent Volume Claims (PVC) that MinIO generates per 
       node in the tenant zone. 
   
   * - :guilabel:`Disk Size`
     - The requested storage capacity for each PVC that MinIO generates for the
       tenant zone.

   * - :guilabel:`Total Number of Volumes`
     - The total number of PVC that MinIO generates for the Tenant.

Click :guilabel:`OK` to add the new Zone.

3) Monitor Zone Creation
~~~~~~~~~~~~~~~~~~~~~~~~

You can view the progress of the zone creation from the :guilabel:`Tenant` view.

.. image:: /images/vsphere/minio-tenant-multi-zone.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant with Multiple Zones

The tenant :guilabel:`Storage` and :guilabel:`Health` may display warnings
while MinIO provisions the pods, Persistent Volume Claims, and Persistent
Volumes required to create the zone. 

Update a MinIO Tenant
---------------------

The MinIO plugin supports upgrading the MinIO tenant through the vCenter 
interface. You can update the following components:

- The MinIO Object Storage :minio-git:`server version  <minio/tags>`
- The MinIO Console :minio-git:`version <console/tags>`

Updating the MinIO tenant requires restarting multiple pods simultaneously. 
The update interrupts ongoing API operations until complete. Consider 
performing the update during a maintenance period to minimize disruption to 
applications.

.. important::

   You cannot downgrade the MinIO Object Storage server used by a tenant 
   once deployed. Use a staging or development tenant for testing the 
   desired MinIO server version *before* making upgrades in a production
   tenant.

1) Open the :guilabel:`Update MinIO Version` Modal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the vCenter interface, select the cluster in which the MinIO tenant you
want to expand is deployed. Click the :guilabel:`Configure` tab, then open the
:guilabel:`MinIO` section and select :guilabel:`Tenants` to open the
:guilabel:`MinIO Tenants` view.

.. image:: /images/vsphere/minio-tenant-list.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO View All Tenants

Click the radio button next to the tenant you want to expand and select
:guilabel:`Details` to open the :guilabel:`Tenant` view.

From the Tenant view, click the pencil icon on the 
:guilabel:`Version` row to open the :guilabel:`Update MinIO Version` modal.

.. image:: /images/vsphere/minio-tenant-upgrade.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant Upgrade

2) Complete the :guilabel:`Update MinIO Version` Modal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. image:: /images/vsphere/minio-tenant-update-modal.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant Update Modal

The modal displays the following fields:

.. list-table::
   :stub-columns: 1
   :widths: 30 70
   :width: 100%

   * - :guilabel:`MinIO's Image`
     - The Docker image to use for updating the MinIO tenant. The MinIO 
       plugin uses the ``minio/minio`` image repository hosted at 
       `hub.docker.com <https://hub.docker.com/r/minio/minio/tags>`__ by 
       default. 

       Leave blank to use the latest stable ``minio`` image.

   * - :guilabel:`Console's Image`
     - The Docker image to use for updating the MinIO 
       :minio-git:`Console <console>` deployed as part of the MinIO tenant.
       The MinIO plugin uses the ``minio/console`` image repository hosted 
       at `hub.docker.com <https://hub.docker.com/r/minio/console/tags>`__
       by default.

       Leave blank to use the latest stable ``console`` image.

   * - :guilabel:`Set Custom Image Registry`
     - Enables using a private Docker repository for retrieving Docker 
       images when updating the MinIO tenant. Enter the 
       :guilabel:`Endpoint` of the repository along with the 
       :guilabel:`Username` and :guilabel:`Password` required to access the 
       repository.

   * - :guilabel:`Enable prometheus integration`
     - Enables annotations that allow Prometheus to scrape the 
       MinIO tenant using the Kubernetes REST API. See 
       the `Prometheus documenation 
       <https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config>`__
       for more information.

Click :guilabel:`OK` to begin the update. MinIO updates *all* pods in the 
tenant simultaneously. The tenant may have a brief period of downtime 
during the update process.
