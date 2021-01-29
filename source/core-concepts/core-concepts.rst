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

MinIO Erasure Coding is a data redundancy and availability feature that allows
MinIO deployments to automatically reconstruct objects on-the-fly despite the
loss of multiple drives or pods in the cluster. Erasure Coding provides
object-level healing with less overhead than adjacent technologies such as
RAID or replication. 

MinIO automatically calculates and creates Erasure Sets in each zone. 
An erasure set is between 4 and 16 drives. MinIO uses the Greatest 
Common Divisor (GCD) in that range for all drives in the zone as the erasure 
set drive. MinIO then splits objects on write into data and parity blocks, 
distributing those blocks among drives in the erasure set. 

A zone may have multiple erasure sets, all of the same calculated size. 
MinIO uses the ``EC:N`` notation to define the ratio of parity blocks to 
data blocks for objects stored in a given erasure set. Specifically, for an
erasure set of size ``M``, each object has:

- ``N`` parity blocks and 
- ``M - N`` data blocks

The number of parity blocks determines the Tenant's ability to continue
servicing read and write requests in the event of drive or pod failure. 
Specifically:

- For read operations, the MinIO Tenant can tolerate the loss of up to
  ``N`` drives in the Erasure Set.

- For write operations, the MinIO Tenant can tolerate the loss of up to
  ``N-1`` drives in the Erasure Set.

MinIO sets the default Erasure Coding parity to 1/2 the drives in the Erasure
Set(s) for the initial zone. For example, a zone with 16-drive Erasure Sets 
defaults to ``EC:8``, while a zone with 10-drive Erasure Sets defaults to 
``EC:5``. 

Since parity blocks require storage space, higher levels of parity provide
increased tolerance to drive or pod failure at the cost of total usable storage
capacity. You can specify your preferred erasure code parity setting during the
:doc:`tenant creation process </tutorials/deploy-minio-tenant>`.

The following table lists the outcome of varying ``EC`` levels on a MinIO Tenant
with 4 pods and 4 1Ti drives per node. MinIO creates a single Erasure Set 
consisting of 16 drives for this Tenant.

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

Bucket Versioning
-----------------

MinIO supports keeping multiple "versions" of an object in a single bucket.
Write operations which would normally overwrite an existing object instead
result in the creation of a new versioned object. MinIO versioning protects from
unintended overwrites and deletions while providing support for "undoing" a
write operation. Bucket versioning also supports retention and archive policies.

MinIO does not enable bucket versioning by default. Use the 
`mc version enable` command to explicitly enable versioning on an 
existing bucket. Alternatively, creating a bucket with 
`mc mb --with-lock` automatically enables versioning.

For buckets with versioning enabled, MinIO generates a unique immutable ID for
each newly written object. If a ``PUT`` request contains an object name which
matches an existing object, MinIO does *not* overwrite the "older" object.
Instead, MinIO retains all object versions while considering the most recently
written "version" of the object as "latest". Similarly, a delete operation on a
versioned object creates a delete marker but does *not* delete the previous
versions of that object. Applications retrieve the latest object version by
default, but *may* retrieve any other version in the history of that object.

For buckets with versioning disabled or suspended, enabling versioning does 
not automatically generate version IDs for existing objects in that bucket. 
MinIO lists unversioned objects with a version ID of ``null``. 
