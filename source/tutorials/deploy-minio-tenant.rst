=====================
Deploy a MinIO Tenant
=====================

.. default-domain:: minio

.. contents:: Table of Contents
   :local:
   :depth: 2


This procedure documents deploying a MinIO object storage tenant using the 
MinIO integration for the VMware Data Persistence platform (DPp) in |vcf| 4.2. 

MinIO takes full advantage of the vSAN Direct and vSAN SNA storage policies 
available with VMware DPp. This procedure assumes that the |vcf| deployment 
has one or more ESXi hosts with direct-attached storage, where a storage 
policy exists for either vSAN Direct or vSAN SNA. 

Prerequisites
-------------

MinIO Plugin for vCenter
~~~~~~~~~~~~~~~~~~~~~~~~
   
You must enable the MinIO plugin in vSphere prior to beginning this
procedure. To enable the plugin, log into vSphere and select the cluster in
which you want to deploy MinIO tenants. 

Click :guilabel:`Configure`, then navigate to :guilabel:`Supervisor Services`
and select :guilabel:`Services`. Locate the 
:guilabel:`MinIO` row and activate the radio button. Click 
:guilabel:`Enable` to enable the service.

ESXi Hosts with Local Storage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each ESXi host *must* have sufficient compute, memory, and direct-attached local
storage devices (SSD or HDD) to support each pod provisioned as part of the
tenant. MinIO requires one available ESXi host for each MinIO server pod
deployed as part of the tenant.

MinIO showcases best performance with locally-attached storage. MinIO
therefore *strongly recommends* vSAN Direct storage policies when 
provisioning MinIO tenant. See 
:vmware-docs:`Set Up vSAN Direct for vSphere with Tanzu 
<7.0/vmware-vsphere-with-tanzu/GUID-B096E155-D58C-4F01-9FF1-DB29B4801C6D.html>`
for more complete instructions. 
   
Namespace per MinIO Tenant
~~~~~~~~~~~~~~~~~~~~~~~~~~

MinIO limits one MinIO tenant per namespace. Create a new namespace *prior*
to beginning the procedure. 

You can view and create namespaces in the vSphere interface by clicking the
:guilabel:`Namespaces` tab for the cluster. Click :guilabel:`New Namespace`
to create a new namespace for the MinIO tenant. After creating the namespace,
return to the :guilabel:`Namespaces` tab and click on the namespace you
created.

- Click :guilabel:`Edit Storage` on the :guilabel:`Storage` card to attach
  the necessary vSAN Direct or vSAN SNA storage policies to the 
  namespace. The MinIO tenant can only use storage policies attached to the 
  namespace in which its deployed.

- Click :guilabel:`Add Permissions` on the :guilabel:`Permissions` card to 
  configure the necessary permissions for vCenter users. Specifically, only
  users with the :guilabel:`Can Edit` permission on the namespace can create
  MinIO tenants in the namespace.

.. important::

  |vcf| allows setting namespace-level limitations on CPU, memory, and
  storage. MinIO tenants are subject to these limitations. For CPU limits
  specifically, |vcf| allows setting a *frequency* limitation. The MinIO
  tenant respects this limitation for every allocated vCPU. If the namespace
  CPU frequency limitation is too low, you may see performance throttling
  even if the tenant has a large number of allocated vCPU.

Procedure
---------

.. _minio-vsphere-create:

1) Open the Tenant Creation Modal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the vSphere interface, select the cluster in which you want to 
deploy the MinIO tenant. 

Click the :guilabel:`Configure` tab, then open the 
:guilabel:`MinIO` section and select :guilabel:`Tenants` to open the 
:guilabel:`MinIO Tenants` view.

Click :guilabel:`ADD` to open the MinIO :guilabel:`Tenant Creation` modal.

.. image:: /images/vsphere/minio-tenant-add.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: Add new MinIO Tenant

.. _minio-vsphere-create-tenant-configuration:

2) Complete the :guilabel:`Tenant Configuration` Step
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. image:: /images/vsphere/minio-tenant-configuration.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: Tenant Configuration step

The :guilabel:`Tenant Configuration` step displays the following fields:

.. list-table::
   :stub-columns: 1
   :widths: 30 70

   * - :guilabel:`Name`
     - The name of the MinIO tenant

   * - :guilabel:`Namespace`
     - The namespace in which to deploy the tenant. The namespace 
       *must not* contain any other MinIO tenants.

   * - :guilabel:`Storage Class`
     - The |vcf| storage class for the MinIO tenant to use when 
       provisioning volumes. MinIO strongly recommends using 
       vSAN Direct storage policies for MinIO tenants.

   * - :guilabel:`Advanced Mode`
     - Displays the following additional configuration modals:
         
       - :guilabel:`Configure` - enable custom docker images and Prometheus integration
       - :guilabel:`IDP` - configure external IDentity Providers
       - :guilabel:`Security` - configure custom TLS
       - :guilabel:`Encryption` - configure server-side encryption of objects

Click :guilabel:`Next` to proceed to the next step.

.. _minio-vsphere-create-configure:

3) Complete the :guilabel:`Configure` Step
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note:: 

   This section is only visible if you selected :guilabel:`Advanced Mode` in the
   :guilabel:`Tenant Configuration` section.

.. image:: /images/vsphere/minio-tenant-configure.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: Tenant Configure Docker Image and Prometheus

The :guilabel:`Configure` step displays the following fields:

.. tabs::

   .. tab:: :guilabel:`Use custom image`

      .. list-table::
         :stub-columns: 1
         :widths: 30 70
         :width: 100%

         * - :guilabel:`Use custom image`
           - Enables using a custom Docker image for deploying pods on the MinIO
             Tenant. 

         * - :guilabel:`MinIO's Image`
           - The custom Docker image to use for deploying MinIO server pods

             Only visible if :guilabel:`Use custom image` is activated.

         * - :guilabel:`Console's Image`
           - The custom Docker image to use for deploying MinIO Console pods.

             Only visible if :guilabel:`Use custom image` is activated.

   .. tab:: :guilabel:`Set Custom Image Registry`

      .. list-table::
         :stub-columns: 1
         :widths: 30 70
         :width: 100%

         * - Field
           - Description

         * - :guilabel:`Set Custom Image Registry`
           - Enables using a private Docker repository for retrieving docker
             images for deploying the MinIO Tenant.

         * - :guilabel:`Endpoint`
           - The URL endpoint for the private Docker repository.

             Only visible if :guilabel:`Set Custom Image Registry` is activated.

         * - :guilabel:`Username`
           - The username for the specified :guilabel:`Endpoint`

             Only visible if :guilabel:`Set Custom Image Registry` is activated.

         * - :guilabel:`Password`
           - The username for the specified :guilabel:`Endpoint`.

             Only visible if :guilabel:`Set Custom Image Registry` is activated. 

.. _minio-vsphere-create-idp:

4) Complete the :guilabel:`IDP` Step
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note:: 

   This section is only visible if you selected :guilabel:`Advanced Mode` in the
   :guilabel:`Tenant Configuration` section.

MinIO has built-in identity management for managing identities on the tenant.
MinIO also supports external IDentity Providers (IDP) for centralized access 
and permission management.

The :guilabel`IDP` section contains configuration settings for using an external
IDentity Provider (IDP) for client authentication and authorization. Select the
radio button that corresponds to the type of IDP you want the MinIO tenant to
use. The default is :guilabel:`None`, or MinIO-managed identities.

.. tabs::

   .. tab:: :guilabel:`OpenID`

      .. image:: /images/vsphere/minio-idp-openid.png
         :align: center
         :width: 90%
         :class: no-scaled-link
         :alt: MinIO Tenant Configure OpenID for external IDP

      .. list-table::
         :stub-columns: 1
         :widths: 30 70
         :width: 100%

         * - :guilabel:`OpenID`
           - Enables using an OpenID Provider for external management of client
             access to the MinIO Tenant. Mutually exclusive with 
             :guilabel:`Active Directory`.

         * - :guilabel:`URL`
           - Specify the URL for the OpenID Provider. Ensure the configured
             network access rules grant the MinIO Tenant access to the specified
             URL endpoint.

         * - :guilabel:`Client ID`
           - Specify the Client ID to use for connecting to the OpenID Provider.

         * - :guilabel:`Secret ID`
           - Specify the Secret ID to use for connecting to the OpenID Provider.

   .. tab:: :guilabel:`Active Directory`

      .. image:: /images/vsphere/minio-idp-active-directory.png
         :align: center
         :width: 90%
         :class: no-scaled-link
         :alt: MinIO Tenant Configure active-directory for external IDP

      .. list-table::
         :stub-columns: 1
         :widths: 30 70
         :width: 100%

         * - :guilabel:`Active Directory`
           - Enables using Microsoft Active Directory *or* an LDAP service for
             external management of client access to the MinIO Tenant. Mutually
             exclusive with :guilabel:`OpenID`.

         * - :guilabel:`URL`
           - The endpoint for the Active Directory or LDAP service. Ensure the
             configured network access rules grant the MinIO tenant access to 
             the specified URL endpoint.
      
         * - :guilabel:`Skip TLS Verification`
           - Directs MinIO to skip verification of TLS certificates and connect to
             Active Directory or LDAP services presenting untrusted certificates
             (e.g. self-signed).

         * - :guilabel:`Server Insecure`
           - Allows plain text connections to the Active Directory or LDAP server.

         * - :guilabel:`User Search Filter`
           - Specify the LDAP query MinIO executes as part of client
             authentication/authorization. For example: ``(userPrincipalName=%s)``

             MinIO substitutes the username provided by the client into the
             ``%s`` placeholder *before* executing the query.

         * - :guilabel:`Group Search Base DN`
           - Specify the base Distinguished Name (DN) MinIO uses when
             querying for LDAP groups in which the authenticated user has membership.
             Specify multiple DNs as a semicolon-separated list.

         * - :guilabel:`Group Search Filter`
           - Specify the LDAP query MinIO executes as part of client
             authentication/authorization.

         * - :guilabel:`Group Name Attribute`
           - Specify the Common Name (CN) attribute MinIO uses when querying for
             LDAP groups in which the authenticated user has membership.

.. _minio-vsphere-create-security:

5) :guilabel:`Security`
~~~~~~~~~~~~~~~~~~~~~~~

.. note::

   This section is only visible if you selected :guilabel:`Advanced Mode` in the
   :guilabel:`Tenant Configuration` section.

The :guilabel:`Security` section contains configuration settings for automatic
and custom TLS certificate generation for resources in the MinIO Tenant:

.. tabs::

   .. tab:: :guilabel:`Autocert`

      .. image:: /images/vsphere/minio-tenant-security-autocert.png
         :align: center
         :width: 90%
         :class: no-scaled-link
         :alt: MinIO Tenant Security Automatic TLS Certificates

      .. list-table::
         :stub-columns: 1
         :widths: 30 70
         :width: 100%

         * - :guilabel:`Enable TLS`
           - Enables TLS authentication for the MinIO Tenant.

             MinIO *strongly recommends* enabling TLS regardless of the
             deployment environment (e.g. development, staging, or production).

         * - :guilabel:`Autocert`
           - Enables automatic generation of self-signed certificates for use by
             resources in the MinIO Tenant. 

             Clients may need to explicitly disable TLS certificate verification
             to connect to the MinIO Tenant, as self-signed certificates
             are typically not trusted by default.

             Only visible if :guilabel:`Enable TLS` is activated.

   .. tab:: :guilabel:`Custom Certificate`

      .. image:: /images/vsphere/minio-tenant-security-custom-certificate.png
         :align: center
         :width: 90%
         :class: no-scaled-link
         :alt: MinIO Tenant Security Custom TLS Certificates

      .. list-table::
         :stub-columns: 1
         :widths: 30 70
         :width: 100%

         * - :guilabel:`Enable TLS`
           - Enables TLS authentication for the MinIO Tenant.

             MinIO *strongly recommends* enabling TLS regardless of the
             deployment environment (e.g. development, staging, or production).

         * - :guilabel:`Custom Certificate`
           - Enables specifying one or more pre-generated TLS x.509 certificates
             for use by resources in the MinIO Tenant. 

         * - :guilabel:`MinIO TLS Certs`
           - Specify a :guilabel:`Key` private key and :guilabel:`Cert` public
             certificate. MinIO uses these certificates when configuring Pod TLS
             and for enabling TLS with SNI support on each pod. Specifically,
             MinIO copies all specified certificates to each MinIO server pod
             and service in the cluster. When the pod/service responds to a TLS
             connection request, it uses SNI to select the certificate with
             matching ``subjectAlternativeName``.

             You can specify additional certificates by clicking the
             :guilabel:`Add One More` button.

         * - :guilabel:`Console TLS Certs`
           - Specify a :guilabel:`Key` private key and :guilabel:`Cert` public
             certificate. MinIO uses these certificates when configuring Pod TLS
             and for enabling TLS with SNI support on each pod. Specifically,
             MinIO copies all specified certificates to each MinIO Console pod
             and service in the cluster. When the pod/service responds to a TLS
             connection request, it uses SNI to select the certificate with
             matching ``subjectAlternativeName``.

.. _minio-vsphere-create-encryption:

6) :guilabel:`Encryption`
~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

   This section is only visible if you selected :guilabel:`Advanced Mode` in the
   :guilabel:`Tenant Configuration` section.

The :guilabel:`Encryption` section contains configuration settings for
Server-Side Encryption of Objects (SSE-S3) stored on the MinIO Tenant. 

.. tabs::

   .. tab:: :guilabel:`Vault`

      .. image:: /images/vsphere/minio-tenant-encryption-vault.png
         :align: center
         :width: 90%
         :class: no-scaled-link
         :alt: MinIO Tenant Encryption Hashicorp Vault

      .. list-table::
         :stub-columns: 1
         :widths: 30 70
         :width: 100%

         * - :guilabel:`Enable Server Side Encryption`
           - Enables configuring SSE of objects on the MinIO Tenant.

         * - :guilabel:`Vault`
           - Enables SSE using Hashicorp Vault as the Key Management Service
             (KMS).

         * - :guilabel:`Endpoint`
           - Specify the URL endpoint for the Vault service. Ensure the
             configured network access rules grant the MinIO Tenant access
             to the specified URL endpoint.

         * - :guilabel:`Engine`
           - Specify the path of the Vault engine to use for storing keys
             generated for supporting SSE-S3.

         * - :guilabel:`Namespace`
           - Specify the namespace on the Vault in which MinIO stores keys
             generated for supporting SSE-S3.

         * - :guilabel:`Prefix`
           - Specify the string prefix to apply when MinIO stores keys
             generated for supporting SSE-S3.

         * - :guilabel:`App Role`
           - Specify the credentials MinIO uses to perform AppRole
             authentication to the Vault server.

             - :guilabel:`Engine` - Specify the engine to use for 
               authentication.

             - :guilabel:`Id` - Specify the AppRole ID to use for
               authentication.

             - :guilabel:`Secret` - Specify the AppRole Secret to use for
               authentication.

             - :guilabel:`Retry` - Specify the number of seconds to wait before
               retrying connections to the Vault server.
         
         * - :guilabel:`TLS`
           - Specify the TLS certificates to use when connecting to the Vault
             server.
             
             - :guilabel:`Key` - Specify the private key ``*.key`` file.

             - :guilabel:`Cert` - Specify the public key ``*.cert`` file.

             - :guilabel:`CA` - Specify the Certificate Authority ``*.crt`` 
               file used to sign the *Vault* TLS certificates.

         * - :guilabel:`Status`
           - Specify how often MinIO should check the status of the Vault
             server. Set :guilabel:`Ping` to the amount of time to wait between
             status checks.

   .. tab:: :guilabel:`AWS`

      .. image:: /images/vsphere/minio-tenant-encryption-aws.png
         :align: center
         :width: 90%
         :class: no-scaled-link
         :alt: MinIO Tenant Encryption AWS Key Management Service

      .. list-table::
         :stub-columns: 1
         :widths: 30 70
         :width: 100%

         * - :guilabel:`Enable Server Side Encryption`
           - Enables configuring Server-Side Encryption of objects on the
             MinIO Tenant.

         * - :guilabel:`AWS`
           - Enables SSE using Amazon Web Service Key Management System
             (AWS KMS) as the Key Management Service (KMS).

         * - :guilabel:`Endpoint`
           - Specify the URL endpoint for the AWS KMS service. Ensure the
             configured network access rules grant the MinIO Tenant access
             to the specified URL endpoint.

         * - :guilabel:`Region`
           - Specify the AWS region of the AWS KMS service.

         * - :guilabel:`KMS Key`
           - The AWS KMS Customer Master Key (CMK) to use for cryptographic
             key operations related to SSE.

         * - :guilabel:`Credentials`
           - Specify the credential to use when making requests to the
             AWS KMS service.

             - :guilabel:`Access Key` - Specify an AWS Access Key.

             - :guilabel:`Secret Key` - Specify the corresponding Secret Key.

             - :guilabel:`Token` - Specify the AWS Token.

   .. tab:: :guilabel:`Gemalto`

      .. image:: /images/vsphere/minio-tenant-encryption-gemalto.png
         :align: center
         :width: 90%
         :class: no-scaled-link
         :alt: MinIO Tenant Encryption Thales Ciphertrust / Gemalto KeyVault

      .. list-table::
         :stub-columns: 1
         :widths: 30 70
         :width: 100%

         * - :guilabel:`Enable Server Side Encryption`
           - Enables configuring Server-Side Encryption of objects on the
             MinIO Tenant.

         * - :guilabel:`Gemalto`
           - Enable SSE using Gemalto KeyVault or Thales CipherTrust as the
             Key Management Service.

         * - :guilabel:`Endpoint`
           - Specify the URL endpoint for the KeyVault or CipherTrust 
             service. Ensure the configured network access rules grant the
             MinIO Tenant access to the specified URL endpoint.

         * - :guilabel:`Credentials`
           - Specify the credentials to use when making requests to the
             KeyVault or CipherTrust service.

             - :guilabel:`Token` - Specify a KeyVault or CipherTrust access 
               token.

             - :guilabel:`Domain` - Specify the domain of the user associated
               to the access token.

             - :guilabel:`Retry` - Specify the number of seconds to wait before
               retrying connections to the KeyVault or CipherTrust service.

         * - :guilabel:`TLS`
           - Specify the Certificate Authority ``*.crt`` file used to sign the
             *KeyVault/CipherTrust* TLS certificates.

.. _minio-vsphere-create-tenant-size:

7) :guilabel:`Tenant Size`
~~~~~~~~~~~~~~~~~~~~~~~~~~

The :guilabel:`Tenant Size` section contains configuration settings for
nodes in the MinIO Tenant:

.. image:: /images/vsphere/minio-tenant-size.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant Size

.. list-table::
   :stub-columns: 1
   :widths: 30 70
   :width: 100%

   * - :guilabel:`Number of Nodes`
     - Specify the number of nodes to create for the MinIO Tenant.

   * - :guilabel:`Storage Size`
     - Specify the total amount of storage in the cluster.

       MinIO automatically calculates the number of volumes per node based on
       the specified storage size and the Storage Class selected in the
       :ref:`Tenant Configuration <minio-vsphere-create-tenant-configuration>`
       step.

       The requested storage *must* be less than or equal to the available
       storage in the specified Storage Class.

   * - :guilabel:`Memory per Node`
     - Specify the amount of RAM to allocate to each node on the MinIO Tenant.
       MinIO recommends a *minimum* of 2Gi of RAM per node.
       
       Click the exclamation mark :guilabel:`!` hint to view recommended 
       memory allocations based on total available storage.

   * - :guilabel:`Erasure Code Parity`
     - The Erasure Code parity setting to apply to the MinIO tenant. 
       Defaults to ``EC:(N/2)``, where ``N`` is the number of volumes in the 
       tenant. The :guilabel:`Resource Allocation` section displays the number 
       of volumes per node based on the selected :guilabel:`Number of Nodes`
       and :guilabel:`Storage Size`.
       
       For more complete documentation on erasure code parity, see 
       :ref:`minio-erasure-coding`.

The :guilabel:`Resource Allocation` section displays the results of the
specified configuration settings:

.. image:: /images/vsphere/minio-tenant-size-resource-allocation.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant Resource Allocation

.. list-table::
   :stub-columns: 1
   :widths: 30 70
   :width: 100%

   * - :guilabel:`Volumes per Node`
     - The number of Persistent Volume Claims (PVC) that MinIO generates 
       per node in the Tenant. 

       MinIO calculates this value based on the requested 
       :guilabel:`Storage Size` and the number of available disks in the
       :guilabel:`Storage Class`.

   * - :guilabel:`Disk Size`
     - The requested storage capacity for each PVC that MinIO generates for
       the Tenant.

       MinIO calculates this value based on the requested 
       :guilabel:`Storage Size` and the number of available disks in the
       :guilabel:`Storage Class`.

   * - :guilabel:`Total Number of Volumes`
     - The total number of PVC that MinIO generates for the Tenant. 

       MinIO calculates this value based on the requested 
       :guilabel:`Storage Size` and the number of available disks in the
       :guilabel:`Storage Class`.

   * - :guilabel:`Erasure Code Parity`
     - The :ref:`Erasure Code parity <minio-erasure-coding>` selected 
       during the :ref:`minio-vsphere-create-tenant-size` step.

   * - :guilabel:`Raw Capacity`
     - The total raw capacity of storage based on the 
       requested :guilabel:`Storage Size`.

   * - :guilabel:`Usable Capacity`
     - The total estimated usable storage capacity based on the
       :guilabel:`Erasure Code Parity`.

       The actual usable capacity depends on the erasure code parity used
       in practice during regular workloads. For example, lowering the
       erasure code parity settings after creating the Tenant would increase
       the total estimated usable storage on the cluster.


.. _minio-vsphere-create-preview:

8) :guilabel:`Preview Configuration`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The :guilabel:`Preview Configuration` section contains the details for the
MinIO Tenant. Review the summary *before* proceeding to the next step.

.. image:: /images/vsphere/minio-tenant-preview.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant Preview Configuration

- The :guilabel:`Name`, :guilabel:`Namespace`, and
  :guilabel:`Storage Class` settings are derived from the 
  :ref:`minio-vsphere-create-tenant-configuration` section. 

- The :guilabel:`Nodes`, :guilabel:`Total Number of Volumes`, 
  :guilabel:`Volumes per Node`, :guilabel:`Disk Size`, 
  :guilabel:`Erasure Code Parity`, :guilabel:`Raw Capacity`, and
  :guilabel:`Usable Capacity` settings are derived from the
  :ref:`minio-vsphere-create-tenant-size` section.

.. _minio-vsphere-create-credentials:

9) :guilabel:`Credentials`
~~~~~~~~~~~~~~~~~~~~~~~~~~

The :guilabel:`Credentials` section contains the credentials required for
connecting to the MinIO Tenant and MinIO Console.

.. image:: /images/vsphere/minio-tenant-credentials.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant Credentials

- The :guilabel:`MinIO's Access Key` and :guilabel:`MinIO's Secret Key` 
  are the credentials for the root MinIO user.

- The :guilabel:`Console's Access Key` and :guilabel:`Console's Secret Key`
  are the credentials for accessing the MinIO Console.

MinIO displays the Access Keys *once*. Click the :guilabel:`Copy Credentials`
button to copy the keys to your system clipboard. Store the keys in a secure
location, such as a password-protected key vault. 

.. important::

   The MinIO Access Key and Secret Key credentials are associated to the
   root user for the MinIO Tenant. Any client
   which accesses the MinIO Tenant with these credentials has superuser 
   access to perform *any* operation on the Tenant.

Click :guilabel:`Finish` to close the :guilabel:`Create Tenant` modal and
return to the :guilabel:`Cluster` view.

10) Monitor Tenant Creation
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can monitor the Tenant creation from the :guilabel:`Tenants` subsection of
the :guilabel:`MinIO` section of the cluster :guilabel:`Configure` tab. Click
the radio button to the left of the tenant, then select :guilabel:`Details`.

.. image:: /images/vsphere/minio-tenant-list.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant List

The :guilabel:`Tenant` view shows the current state of the tenant.

.. image:: /images/vsphere/minio-tenant-ready.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant View

The :guilabel:`Current State` describes the stage of Tenant deployment. The 
cluster :guilabel:`Tasks` view provides a more granular view of MinIO as it
creates the required resources for the Tenant.

When the :guilabel:`Current State` reads as :guilabel:`Initialized`, the Tenant
is ready to access.

11) Connect to your Tenant
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. image:: /images/vsphere/minio-tenant-ready.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: MinIO Tenant View

The :guilabel:`MinIO Endpoint` displays the IP address to use for connecting
to the MinIO Tenant. 

You can specify this endpoint along with the MinIO Access Key and Secret Key
to connect to the Tenant and begin performing operations on it. For example,
the following operation uses the ``mc`` command line tool to configure
an alias for the new MinIO Tenant and retrieve its status:

.. code-block:: shell

   mc alias --insecure set vmw-minio-tenant ENDPOINT ACCESSKEY SECRETKEY

   mc admin --insecure info vmw-minio-tenant

The ``--insecure`` option allows connecting to an endpoint using
self-signed certificates, and may be required for Tenants created using
:ref:`Autocert <minio-vsphere-create-security>` TLS certificate generation.

You can also connect to the MinIO Console by clicking the 
:guilabel:`Console Endpoint` URL and entering in the Console access key and 
secret key.

.. _minio-vsphere-create-vCPU-allocation:

12) Modify vCPU Allocations for Tenant
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

|vcf| 4.2 defaults MinIO pods to 1vCPU. You can modify the number of vCPU 
allocated to each MinIO pod after deploying the tenant by doing the following:

1. Download and install the 
   :vmware-docs:`VMware Commandline Tools </7.0/vmware-vsphere-with-tanzu/GUID-0F6E45C4-3CB1-4562-9370-686668519FCA.html>`

2. Use ``kubectl vsphere login`` to create a context for accessing the 
   |vcf| cluster. 
   
   The vSphere user specified to the command *must* have 
   permission to access and perform operations on the namespace in which 
   the tenant is deployed. See 
   :vmware-docs:`Connecting to vSphere with Tanzu Clusters </7.0/vmware-vsphere-with-tanzu/GUID-FBB9722C-1BB4-4CF2-AB4C-A3ADB5FCC971.html>`
   for more specific instructions.

3. Run the following command to modify the vCPU allocation for the tenant:

   .. code-block:: shell
      :class: copyable

      kubectl -n <NAMESPACE> patch tenant <TENANT-NAME> --type='json' \
      -p='[
             {
                "op": "replace", 
                "path": "/spec/zones/0/resources/limits/cpu", 
                "value": <CPU-LIMIT> 
             },
             {
                "op": "replace", 
                "path": "/spec/zones/0/resources/requests/cpu", 
                "value": <CPU-LIMIT>
             }
          ]' 

   - Replace ``<NAMESPACE>`` with the namespace in which the tenant is deployed.
   - Replace ``<TENANT-NAME>`` with the name of the MinIO tenant.
   - Replace ``<CPU-LIMIT>`` with the number of vCPU to allocate to each tenant 
     pod.

   For tenants with multiple zones, re-issue the command and increment the 
   value of ``/spec/zones/0`` for each zone in the tenant 
   (``spec/zones/1``, ``spec/zones/2``, etc.).
