======================
Monitor a MinIO Tenant
======================

.. default-domain:: minio

.. contents:: Table of Contents
   :local:
   :depth: 1


This procedure documents monitoring a MinIO object storage tenant deployed on
|vcf| 4.2.

Monitor MinIO using Prometheus
------------------------------

MinIO exports `Prometheus <https://prometheus.io/>`__ compatible data as an
authorized endpoint. Users can monitor MinIO tenants using any
Prometheus-compatible application by scraping data from this endpoint.

This procedure assumes an existing Prometheus deployment with access to the 
MinIO tenant. Documentation for deploying and configuring Prometheus is 
outside the scope of this documentation.

1) Generate a Bearer Token
~~~~~~~~~~~~~~~~~~~~~~~~~~

The MinIO scrape endpoint requires authentication. Prometheus supports a 
bearer token approach to authenticate scrape requests. Use the 
``mc admin prometheus generate`` command to generate the required bearer token.
See `https://min.io/download <https://min.io/download?jmp=docs-vsphere>`__ 
for instructions on downloading ``mc``.

.. code-block:: shell
   :class: copyable

   mc alias set <TENANT-NAME> <TENANT-URL> <ACCESS_KEY> <SECRET_KEY>

   mc admin prometheus generate <TENANT-NAME>

- Replace ``<TENANT-NAME>`` with the name of the MinIO tenant.

- Replace ``<TENANT-URL>`` with the :ref:`endpoint 
  <minio-vcenter-create-connect>` for the MinIO tenant. 

- Replace ``<ACCESS_KEY>`` and ``<SECRET_KEY>`` with the 
  :ref:`root credentials <minio-vcenter-create-credentials>` for the MinIO 
  tenant.

- Add ``--insecure`` if using self signed certificates.

The command returns a bearer token similar to the following:

.. code-block:: shell

   scrape_config:
   - job_name: minio-job
     bearer_token: TOKEN
     metrics_path: <PATH>
     scheme: http
     static_configs:
     - targets: ['HOSTNAME:9000']

If using a self signed certificate, add the following to skip verification.

.. code-block:: shell

   scrape_config:
   - job_name: minio-job
     bearer_token: TOKEN
     metrics_path: <PATH>
     scheme: http
     static_configs:
     - targets: ['HOSTNAME:9000']
     tls_config:
       insecure_skip_verify: true
       
The value of ``metrics_path`` depends on the version of ``mc`` is used to
generate the bearer token:

- For ``mc`` version  
  :minio-git:`RELEASE.2021-01-16T02-45-34Z
  <mc/releases/tag/RELEASE.2021-01-16T02-45-34Z>` or earlier, 
  ``metrics_path`` is ``/minio/prometheus/metrics``. This includes the default
  MinIO version used by tenants (|minio-server-default|).

- For MinIO tenants using MinIO version 
  :minio-git:`RELEASE.RELEASE.2021-01-30T00-50-42Z
  <mc/releases/tag/RELEASE.2021-01-30T00-50-42Z>` or later, the
  ``metrics_path`` is ``/minio/v2/metrics/cluster``.

2) Configure Prometheus for Scraping MinIO 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Prometheus uses the 
`scrape_config <https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config>`__
section of its `configuration file <https://prometheus.io/docs/prometheus/latest/configuration/configuration/>`__
to define endpoints for scraping metrics. 

- For existing Prometheus deployments, modify the ``scrape_config`` section 
  to include the new ``job_name`` element and restart Prometheus.

- For new Prometheus deployments, ensure the ``scrape_config`` section includes
  the MinIO `job_name` element before starting Prometheus.

The Prometheus service *must* have access to the MinIO tenant. For Prometheus
servers deployed as part of the |vcf| Tanzu infrastructure, you need to 
modify the configuration map used to instantiate each Prometheus pods with
the new scrape job.

3) Validate Metric Collection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Prometheus has a built-in 
`expression browser <https://prometheus.io/docs/visualization/browser/>`__
for ad-hoc queries. 

Alternatively, you can use any application which supports querying Prometheus,
such as `Grafana <https://prometheus.io/docs/visualization/grafana/>`__.
MinIO publishes a first-party 
`Grafana Dashboard <https://grafana.com/grafana/dashboards/11568>`__ 
designed to collect all monitoring stats for MinIO tenants.

Monitor MinIO using VMware Skyline Health
-----------------------------------------

MinIO reports the health status of each tenant to VMware 
`Skyline Health 
<https://docs.vmware.com/en/VMware-Skyline-Health-Diagnostics/index.html>`__. 

From the vCenter interface, select the cluster in which you want to deploy
the MinIO tenant. 

Click the :guilabel:`Monitor` tab, then open the :guilabel:`vSAN` section and 
select :guilabel:`Skyline Health`. 

.. image:: /images/vsphere/vsphere-skyline-health.png
   :align: center
   :width: 90%
   :class: no-scaled-link
   :alt: vCenter Skyline Health Monitoring.

MinIO lists health checks related to each tenant under the 
:guilabel:`MinIO Service` section.

