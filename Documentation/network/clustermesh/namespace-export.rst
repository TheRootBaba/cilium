.. _clustermesh_namespace_export:

****************************
ClusterMesh Namespace Export
****************************

ClusterMesh can limit which namespaces are exported to connected clusters.
This controls synchronization of CiliumEndpoints, CiliumIdentities, and
services, reducing the amount of state shared by the mesh.

Global and local namespaces
###########################

A *global* namespace is included in ClusterMesh export. Its endpoint and
identity information is available to peer clusters, enabling cross-cluster
pod-to-pod connectivity and network policy enforcement based on remote pod
labels. A *local* namespace is not exported. Cross-cluster connectivity and
label-based policy enforcement for pods in local namespaces are not supported.

The default for namespaces without an explicit annotation is controlled by
the Helm value ``clustermesh.defaultGlobalNamespace`` (the
``clustermesh-default-global-namespace`` setting in the ClusterMesh API
server):

- ``true`` (default): namespaces are global unless explicitly marked local.
- ``false``: namespaces are local unless explicitly marked global.

Set the default when installing or upgrading Cilium:

.. code-block:: yaml

   clustermesh:
     defaultGlobalNamespace: false

The namespace annotation overrides the default. Namespace annotations are
cluster-local, so mark the namespace global in every cluster where it must be
exported. For example:

.. code-block:: shell-session

   kubectl annotate namespace production clustermesh.cilium.io/global="true"

Mark a namespace local:

.. code-block:: shell-session

   kubectl annotate namespace internal clustermesh.cilium.io/global="false"

Removing the annotation returns the namespace to the configured default.
Changing the annotation updates which namespace resources are exported.

Global Services
###############

A service is exported as a Global Service only when both conditions hold:

- Its namespace is global.
- The Service has the ``service.cilium.io/global: "true"`` annotation.

For example, setting the service annotation does not export a service from a
local namespace when ``clustermesh.defaultGlobalNamespace`` is ``false``.
See :ref:`gs_clustermesh_global_services` for the Global Services setup.

Multi-Cluster Services API
##########################

MCS-API exports are also scoped to global namespaces. A ``ServiceExport``
created in a local namespace is not exported; the corresponding ServiceImport
is not usable for cross-cluster service discovery. Mark the namespace global
before creating or expecting an MCS-API export. See
:ref:`gs_clustermesh_mcsapi` for MCS-API configuration and examples.

Network policy and connectivity considerations
###############################################

For cross-cluster communication with label-based policy enforcement, both the
source and destination pods must be in global namespaces. Endpoint and
identity information from local namespaces is not distributed to peer
clusters, so remote endpoint selectors cannot match labels for those pods.
Do not rely on cross-cluster NetworkPolicy enforcement for workloads in local
namespaces.

When troubleshooting missing remote endpoints or services, check the namespace
annotation and the configured default on every cluster in the mesh.
