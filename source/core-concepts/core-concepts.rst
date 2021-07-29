=============
Core Concepts
=============

.. default-domain:: minio

.. contents:: Table of Contents
   :local:
   :depth: 1

.. _minio-erasure-coding:

Erasure Coding
--------------

For more complete documentation on MinIO Erasure Coding, see
:ref:`Erasure Coding <baremetal:minio-erasure-coding>`. This section 
summarizes erasure coding in context of MinIO tenants on |vcf|.

MinIO protects data with per-object inline erasure coding written in 
assembly code to deliver the highest possible performance. MinIO uses 
Reed-Solomon code to stripe objects into data and parity blocks with 
user-configurable redundancy levels. MinIO's erasure coding performs healing 
at the object level and can heal multiple objects independently.

At the maximum parity of ``N/2``, where ``N`` is the total number of volumes
in a MinIO tenant zone, MinIO's implementation can ensure uninterrupted 
read and write operations with only ``((N/2)+1)`` operational volumes 
in the deployment. For example, in a 16-drive setup, MinIO shards objects
across 8 data and 8 parity drives and can reliably write new objects or 
reconstruct existing objects with only 9 drives remaining in the deployment.

MinIO Erasure Coding is supported by Erasure Sets. MinIO automatically
calculates and creates Erasure Sets in each zone. MinIO then splits objects on
write into data and parity blocks, distributing those blocks among drives in the
erasure set. You can view the erasure set distribution when 
:ref:`Deploying a Tenant <minio-vcenter-create-tenant-size>` or 
:ref:`Expanding a Tenant <minio-vcenter-expand-tenant>`.

A zone may have multiple erasure sets, all of the same calculated size. MinIO
uses the ``EC:N`` notation to define the ratio of parity blocks to data blocks
for objects stored in a given erasure set. The number of parity blocks
determines the Tenant's ability to continue servicing read and write requests in
the event of drive or pod failure. Specifically, for ``M = Erasure Set Drives``:

- For write operations (write quorum), the MinIO tenant requires *at least*
  ``(M-EC:N)+1`` drives.

- For read operations (read quorum), the MinIO tenant requires *at least*
  ``(M-EC:N)`` drives.

A MinIO tenant with enough erasure set drives to satisfy write quorum implicitly
has enough drives to satisfy read quorum.

MinIO tenants default Erasure Coding parity to 1/2 of the drives in the Erasure
Set(s) for the initial zone. For example, a zone with 16-drive Erasure Sets
defaults to ``EC:8``, while a zone with 10-drive Erasure Sets defaults to
``EC:5``. 

Since parity blocks require storage space, higher levels of parity provide
increased tolerance to drive or pod failure at the cost of total usable storage
capacity. You can specify your preferred erasure code parity setting when
:ref:`Deploying a Tenant <minio-vcenter-create-tenant-size>`. 

The following table lists the outcome of varying ``EC`` levels on a MinIO Tenant
with 4 pods and 4 1Ti drives per node. MinIO creates a single Erasure Set 
consisting of 16 drives for this tenant.

.. list-table:: Outcome of Parity Settings on a 16 Drive MinIO Tenant
   :header-rows: 1
   :widths: 20 20 20 20 20
   :width: 100%

   * - Parity
     - Total Storage
     - Storage Ratio
     - Minimum Drives for Read Operations
     - Minimum Drives for Write Operations

   * - ``EC: 4``
     - 12 Tebibytes
     - 0.750
     - 12
     - 13

   * - ``EC: 6``
     - 10 Tebibytes
     - 0.625
     - 10
     - 16

   * - ``EC: 8`` (Default)
     - 8 Tebibytes
     - 0.500
     - 8
     - 9

