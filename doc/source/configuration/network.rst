=====================
Network Configuration
=====================

Kayobe provides a flexible mechanism for configuring the networks in a system.
Kayobe networks are assigned a name which is used as a prefix for variables
that define the network's attributes.  For example, to configure the ``cidr``
attribute of a network named ``arpanet``, we would use a variable named
``arpanet_cidr``.

Global Network Configuration
============================

Global network configuration is stored in
``${KAYOBE_CONFIG_PATH}/networks.yml``.  The following attributes are
supported:

``cidr``
    CIDR representation (<IP>/<prefix length>) of the network's IP subnet.
``allocation_pool_start``
    IP address of the start of Kayobe's allocation pool range.
``allocation_pool_end``
    IP address of the end of Kayobe's allocation pool range.
``inspection_allocation_pool_start``
    IP address of the start of ironic inspector's allocation pool range.
``inspection_allocation_pool_end``
    IP address of the end of ironic inspector's allocation pool range.
``neutron_allocation_pool_start``
    IP address of the start of neutron's allocation pool range.
``neutron_allocation_pool_end``
    IP address of the end of neutron's allocation pool range.
``gateway``
    IP address of the network's default gateway.
``inspection_gateway``
    IP address of the gateway for the hardware introspection network.
``neutron_gateway``
    IP address of the gateway for a neutron subnet based on this network.
``vlan``
    VLAN ID.
``mtu``
    Maximum Transmission Unit (MTU).
``routes``
    List of static IP routes. Each item should be a dict containing the
    item ``cidr``, and optionally ``gateway`` and ``table``. ``cidr`` is the CIDR
    representation of the route's destination. ``gateway`` is the IP address of
    the next hop. ``table`` is the name or ID of a routing table to which the
    route will be added.
``rules``
    List of IP routing rules. Each item should be an ``iproute2`` IP routing
    rule.
``physical_network``
    Name of the physical network on which this network exists. This aligns with
    the physical network concept in neutron.
``libvirt_network_name``
    A name to give to a Libvirt network representing this network on the seed
    hypervisor.

Configuring an IP Subnet
------------------------

An IP subnet may be configured by setting the ``cidr`` attribute for a network
to the CIDR representation of the subnet.

To configure a network called ``example`` with the ``10.0.0.0/24`` IP subnet:

.. code-block:: yaml
   :caption: ``networks.yml``

   example_cidr: 10.0.0.0/24

Configuring an IP Gateway
-------------------------

An IP gateway may be configured by setting the ``gateway`` attribute for a
network to the IP address of the gateway.

To configure a network called ``example`` with a gateway at ``10.0.0.1``:

.. code-block:: yaml
   :caption: ``networks.yml``

   example_gateway: 10.0.0.1

This gateway will be configured on all hosts to which the network is mapped.
Note that configuring multiple IP gateways on a single host will lead to
unpredictable results.

Configuring Static IP Routes
----------------------------

Static IP routes may be configured by setting the ``routes`` attribute for a
network to a list of routes.

To configure a network called ``example`` with a single IP route to the
``10.1.0.0/24`` subnet via ``10.0.0.1``:

.. code-block:: yaml
   :caption: ``networks.yml``

   example_routes:
     - cidr: 10.1.0.0/24
       gateway: 10.0.0.1

These routes will be configured on all hosts to which the network is mapped.

Configuring a VLAN
------------------

A VLAN network may be configured by setting the ``vlan`` attribute for a
network to the ID of the VLAN.

To configure a network called ``example`` with VLAN ID ``123``:

.. code-block:: yaml
   :caption: ``networks.yml``

   example_vlan: 123

IP Address Allocation
=====================

IP addresses are allocated automatically by Kayobe from the
allocation pool
defined by ``allocation_pool_start`` and ``allocation_pool_end``.  The
allocated addresses are stored in
``${KAYOBE_CONFIG_PATH}/network-allocation.yml`` using the global per-network
attribute ``ips`` which maps Ansible inventory hostnames to allocated IPs.

If static IP address allocation is required, the IP allocation file
``network-allocation.yml`` may be manually populated with the required
addresses.

Configuring Dynamic IP Address Allocation
-----------------------------------------

To configure a network called ``example`` with the ``10.0.0.0/24`` IP subnet
and an allocation pool spanning from ``10.0.0.4`` to ``10.0.0.254``:

.. code-block:: yaml
   :caption: ``networks.yml``

   example_cidr: 10.0.0.0/24
   example_allocation_pool_start: 10.0.0.4
   example_allocation_pool_end: 10.0.0.254

.. note::

   This pool should not overlap with an inspection or neutron allocation pool
   on the same network.

Configuring Static IP Address Allocation
----------------------------------------

To configure a network called ``example`` with statically allocated IP
addresses for hosts ``host1`` and ``host2``:

.. code-block:: yaml
   :caption: ``network-allocation.yml``

   example_ips:
     host1: 10.0.0.1
     host2: 10.0.0.2

Advanced: Policy-Based Routing
------------------------------

Policy-based routing can be useful in complex networking environments,
particularly where asymmetric routes exist, and strict reverse path filtering
is enabled.

Configuring IP Routing Tables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Custom IP routing tables may be configured by setting the global variable
``network_route_tables`` in ``${KAYOBE_CONFIG_PATH}/networks.yml`` to a list of
route tables. These route tables will be added to ``/etc/iproute2/rt_tables``.

To configure a routing table called ``exampleroutetable`` with ID ``1``:

.. code-block:: yaml
   :caption: ``networks.yml``

   network_route_tables:
     - name: exampleroutetable
       id: 1

To configure route tables on specific hosts, use a host or group variables
file.

Configuring IP Routing Policy Rules
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

IP routing policy rules may be configured by setting the ``rules`` attribute
for a network to a list of rules. The format of a rule is the string which
would be appended to ``ip rule <add|del>`` to create or delete the rule.

To configure a network called ``example`` with an IP routing policy rule to
handle traffic from the subnet ``10.1.0.0/24`` using the routing table
``exampleroutetable``:

.. code-block:: yaml
   :caption: ``networks.yml``

   example_rules:
     - from 10.1.0.0/24 table exampleroutetable

These rules will be configured on all hosts to which the network is mapped.

Configuring IP Routes on Specific Tables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A route may be added to a specific routing table by adding the name or ID of
the table to a ``table`` attribute of the route:

To configure a network called ``example`` with a default route and a
'connected' (local subnet) route to the subnet ``10.1.0.0/24`` on the table
``exampleroutetable``:

.. code-block:: yaml
   :caption: ``networks.yml``

   example_routes:
     - cidr: 0.0.0.0/0
       gateway 10.1.0.1
       table: exampleroutetable
     - cidr: 10.1.0.0/24
       table: exampleroutetable

Per-host Network Configuration
==============================

Some network attributes are specific to a host's role in the system, and
these are stored in
``${KAYOBE_CONFIG_PATH}/inventory/group_vars/<group>/network-interfaces``.
The following attributes are supported:

``interface``
    The name of the network interface attached to the network.
``bootproto``
    Boot protocol for the interface. Valid values are ``static`` and ``dhcp``.
    The default is ``static``. When set to ``dhcp``, an external DHCP server
    must be provided.
``bridge_ports``
    For bridge interfaces, a list of names of network interfaces to add to the
    bridge.
``bond_mode``
    For bond interfaces, the bond's mode, e.g. 802.3ad.
``bond_slaves``
    For bond interfaces, a list of names of network interfaces to act as slaves
    for the bond.
``bond_miimon``
    For bond interfaces, the time in milliseconds between MII link monitoring.
``bond_updelay``
    For bond interfaces, the time in milliseconds to wait before declaring an
    interface up (should be multiple of ``bond_miimon``).
``bond_downdelay``
    For bond interfaces, the time in milliseconds to wait before declaring an
    interface down (should be multiple of ``bond_miimon``).
``bond_xmit_hash_policy``
    For bond interfaces, the xmit_hash_policy to use for the bond.
``bond_lacp_rate``
    For bond interfaces, the lacp_rate to use for the bond.

IP Addresses
------------

An interface will be assigned an IP address if the associated network has a
``cidr`` attribute. The IP address will be assigned from the range defined by
the ``allocation_pool_start`` and ``allocation_pool_end`` attributes, if one
has not been statically assigned in ``network-allocation.yml``.

Configuring Ethernet Interfaces
-------------------------------

An Ethernet interface may be configured by setting the ``interface`` attribute
for a network to the name of the Ethernet interface.

To configure a network called ``example`` with an Ethernet interface on
``eth0``:

.. code-block:: yaml
   :caption: ``inventory/group_vars/<group>/network-interfaces``

   example_interface: eth0

.. _configuring-bridge-interfaces:

Configuring Bridge Interfaces
-----------------------------

A Linux bridge interface may be configured by setting the ``interface``
attribute of a network to the name of the bridge interface, and the
``bridge_ports`` attribute to a list of interfaces which will be added as
member ports on the bridge.

To configure a network called ``example`` with a bridge interface on
``breth1``, and a single port ``eth1``:

.. code-block:: yaml
   :caption: ``inventory/group_vars/<group>/network-interfaces``

   example_interface: breth1
   example_bridge_ports:
     - eth1

Bridge member ports may be Ethernet interfaces, bond interfaces, or VLAN
interfaces.  In the case of bond interfaces, the bond must be configured
separately in addition to the bridge, as a different named network.  In the
case of VLAN interfaces, the underlying Ethernet interface must be configured
separately in addition to the bridge, as a different named network.

Configuring Bond Interfaces
---------------------------

A bonded interface may be configured by setting the ``interface`` attribute of
a network to the name of the bond's master interface, and the ``bond_slaves``
attribute to a list of interfaces which will be added as slaves to the master.

To configure a network called ``example`` with a bond with master interface
``bond0`` and two slaves ``eth0`` and ``eth1``:

.. code-block:: yaml
   :caption: ``inventory/group_vars/<group>/network-interfaces``

   example_interface: bond0
   example_bond_slaves:
     - eth0
     - eth1

Optionally, the bond mode and MII monitoring interval may also be configured:

.. code-block:: yaml
   :caption: ``inventory/group_vars/<group>/network-interfaces``

   example_bond_mode: 802.3ad
   example_bond_miimon: 100

Bond slaves may be Ethernet interfaces, or VLAN interfaces.  In the case of
VLAN interfaces, underlying Ethernet interface must be configured separately in
addition to the bond, as a different named network.

Configuring VLAN Interfaces
---------------------------

A VLAN interface may be configured by setting the ``interface`` attribute  of a
network to the name of the VLAN interface.  The interface name must be of the
form ``<parent interface>.<VLAN ID>``.

To configure a network called ``example`` with a VLAN interface with a parent
interface of ``eth2`` for VLAN ``123``:

.. code-block:: yaml
   :caption: ``inventory/group_vars/<group>/network-interfaces``

   example_interface: eth2.123

To keep the configuration DRY, reference the network's ``vlan`` attribute:

.. code-block:: yaml
   :caption: ``inventory/group_vars/<group>/network-interfaces``

   example_interface: "eth2.{{ example_vlan }}"

Ethernet interfaces, bridges, and bond master interfaces may all be parents to
a VLAN interface.

Bridges and VLANs
^^^^^^^^^^^^^^^^^

Adding a VLAN interface to a bridge directly will allow tagged traffic for that
VLAN to be forwarded by the bridge, whereas adding a VLAN interface to an
Ethernet or bond interface that is a bridge member port will prevent tagged
traffic for that VLAN being forwarded by the bridge.

Domain Name Service (DNS) Resolver Configuration
================================================

Kayobe supports configuration of hosts' DNS resolver via ``resolv.conf``.  DNS
configuration should be added to ``dns.yml``.  For example:

.. code-block:: yaml
   :caption: ``dns.yml``

   resolv_nameservers:
     - 8.8.8.8
     - 8.8.4.4
   resolv_domain: example.com
   resolv_search:
     - kayobe.example.com

It is also possible to prevent kayobe from modifying ``resolv.conf`` by setting
``resolv_is_managed`` to ``false``.

Network Role Configuration
==========================

In order to provide flexibility in the system's network topology, Kayobe maps
the named networks to logical network roles.  A single named network may
perform multiple roles, or even none at all.  The available roles are:

Overcloud admin network (``admin_oc_net_name``)
    Name of the network used to access the overcloud for admin purposes, e.g
    for remote SSH access.
Overcloud out-of-band network (``oob_oc_net_name``)
    Name of the network used by the seed to access the out-of-band management
    controllers of the bare metal overcloud hosts.
Overcloud provisioning network (``provision_oc_net_name``)
    Name of the network used by the seed to provision the bare metal overcloud
    hosts.
Workload out-of-band network (``oob_wl_net_name``)
    Name of the network used by the overcloud hosts to access the out-of-band
    management controllers of the bare metal workload hosts.
Workload provisioning network (``provision_wl_net_name``)
    Name of the network used by the overcloud hosts to provision the bare metal
    workload hosts.
Workload cleaning network (``cleaning_net_name``)
    Name of the network used by the overcloud hosts to clean the baremetal
    workload hosts.
Internal network (``internal_net_name``)
    Name of the network used to expose the internal OpenStack API endpoints.
Public network (``public_net_name``)
    Name of the network used to expose the public OpenStack API endpoints.
Tunnel network (``tunnel_net_name``)
    Name of the network used by Neutron to carry tenant overlay network
    traffic.
External networks (``external_net_names``, deprecated: ``external_net_name``)
    List of names of networks used to provide external network access via
    Neutron. If ``external_net_name`` is defined, ``external_net_names``
    defaults to a list containing only that network.
Storage network (``storage_net_name``)
    Name of the network used to carry storage data traffic.
Storage management network (``storage_mgmt_net_name``)
    Name of the network used to carry storage management traffic.
Workload inspection network (``inspection_net_name``)
    Name of the network used to perform hardware introspection on the bare
    metal workload hosts.

These roles are configured in ``${KAYOBE_CONFIG_PATH}/networks.yml``.

Configuring Network Roles
-------------------------

To configure network roles in a system with two networks, ``example1`` and
``example2``:

.. code-block:: yaml
   :caption: ``networks.yml``

   admin_oc_net_name: example1
   oob_oc_net_name: example1
   provision_oc_net_name: example1
   oob_wl_net_name: example1
   provision_wl_net_name: example2
   internal_net_name: example2
   public_net_name: example2
   tunnel_net_name: example2
   external_net_name: example2
   storage_net_name: example2
   storage_mgmt_net_name: example2
   inspection_net_name: example2
   cleaning_net_name: example2

Overcloud Admin Network
-----------------------

The admin network is intended to be used for remote access to the overcloud hosts.
Kayobe will use the address assigned to the host on this network as the
``ansible_host`` when executing playbooks. It is therefore a necessary requirement
to configure this network.

By default Kayobe will use the overcloud provisioning network as the admin network.
It is, however, possible to configure a separate network. To do so, you should
override ``admin_oc_net_name`` in your networking configuration.

If a separate network is configured, the following requirements should be taken into
consideration:

* The admin network must be configured to use the same physical network interface
  as the provisioning network. This is because the PXE MAC address is used to
  lookup the interface for the cloud-init network configuration that occurs
  during bifrost provisioning of the overcloud.

* If the admin network is configured as a tagged VLAN, you must configure Kayobe
  to upgrade cloud-init. This is a temporary workaround for a bug in the current
  version of cloud-init shipped with CentOS 7.5. Please see :ref:`workaround-cloud-init`
  for more details.

Overcloud Provisioning Network
------------------------------

If using a seed to inspect the bare metal overcloud hosts, it is necessary to
define a DHCP allocation pool for the seed's ironic inspector DHCP server using
the ``inspection_allocation_pool_start`` and ``inspection_allocation_pool_end``
attributes of the overcloud provisioning network.

.. note::

   This example assumes that the ``example`` network is mapped to
   ``provision_oc_net_name``.

To configure a network called ``example`` with an inspection allocation pool:

.. code-block:: yaml

   example_inspection_allocation_pool_start: 10.0.0.128
   example_inspection_allocation_pool_end: 10.0.0.254

.. note::

   This pool should not overlap with a kayobe allocation pool on the same
   network.

Workload Cleaning Network
-------------------------

A separate cleaning network, which is used by the overcloud to clean baremetal
workload (compute) hosts, may optionally be specified. Otherwise,
the Workload Provisoning network is used. It is necessary to define an IP
allocation pool for neutron using the
``neutron_allocation_pool_start`` and ``neutron_allocation_pool_end``
attributes of the cleaning network. This controls the IP addresses that are
assigned to workload hosts during cleaning.

.. note::

   This example assumes that the ``example`` network is mapped to
   ``cleaning_net_name``.

To configure a network called ``example`` with a neutron provisioning
allocation pool:

.. code-block:: yaml

   example_neutron_allocation_pool_start: 10.0.1.128
   example_neutron_allocation_pool_end: 10.0.1.195

.. note::

   This pool should not overlap with a kayobe or inspection allocation pool on
   the same network.

Workload Provisioning Network
-----------------------------

If using the overcloud to provision bare metal workload (compute) hosts, it is
necessary to define an IP allocation pool for the overcloud's neutron
provisioning network using the ``neutron_allocation_pool_start`` and
``neutron_allocation_pool_end`` attributes of the workload provisioning
network.

.. note::

   This example assumes that the ``example`` network is mapped to
   ``provision_wl_net_name``.

To configure a network called ``example`` with a neutron provisioning
allocation pool:

.. code-block:: yaml

   example_neutron_allocation_pool_start: 10.0.1.128
   example_neutron_allocation_pool_end: 10.0.1.195

.. note::

   This pool should not overlap with a kayobe or inspection allocation pool on
   the same network.

Workload Inspection Network
---------------------------

If using the overcloud to inspect bare metal workload (compute) hosts, it is
necessary to define a DHCP allocation pool for the overcloud's ironic inspector
DHCP server using the ``inspection_allocation_pool_start`` and
``inspection_allocation_pool_end`` attributes of the workload provisioning
network.

.. note::

   This example assumes that the ``example`` network is mapped to
   ``provision_wl_net_name``.

To configure a network called ``example`` with an inspection allocation pool:

.. code-block:: yaml

   example_inspection_allocation_pool_start: 10.0.1.196
   example_inspection_allocation_pool_end: 10.0.1.254

.. note::

   This pool should not overlap with a kayobe or neutron allocation pool on the
   same network.

Neutron Networking
==================

.. note::

   This assumes the use of the neutron ``openvswitch`` ML2 mechanism driver for
   control plane networking.

Certain modes of operation of neutron require layer 2 access to physical
networks in the system.  Hosts in the ``network`` group (by default, this is
the same as the ``controllers`` group) run the neutron networking services
(Open vSwitch agent, DHCP agent, L3 agent, metadata agent, etc.).

The kayobe network configuration must ensure that the neutron Open
vSwitch bridges on the network hosts have access to the external network.  If
bare metal compute nodes are in use, then they must also have access to the
workload provisioning network. This can be done by ensuring that the external
and workload provisioning network interfaces are bridges.  Kayobe will ensure
connectivity between these Linux bridges and the neutron Open vSwitch bridges
via a virtual Ethernet pair.  See :ref:`configuring-bridge-interfaces`.

Network to Host Mapping
=======================

Networks are mapped to hosts using the variable ``network_interfaces``.
Kayobe's playbook group variables define some sensible defaults for this
variable for hosts in the top level standard groups.  These defaults are set
using the network roles typically required by the group.

Seed
----

By default, the seed is attached to the following networks:

* overcloud admin network
* overcloud out-of-band network
* overcloud provisioning network

This list may be extended by setting ``seed_extra_network_interfaces`` to a
list of names of additional networks to attach.  Alternatively, the list may be
completely overridden by setting ``seed_network_interfaces``.  These variables
are found in ``${KAYOBE_CONFIG_PATH}/seed.yml``.

Seed Hypervisor
---------------

By default, the seed hypervisor is attached to the same networks as the seed.

This list may be extended by setting
``seed_hypervisor_extra_network_interfaces`` to a list of names of additional
networks to attach.  Alternatively, the list may be
completely overridden by setting ``seed_hypervisor_network_interfaces``.  These
variables are found in ``${KAYOBE_CONFIG_PATH}/seed-hypervisor.yml``.

Controllers
-----------

By default, controllers are attached to the following networks:

* overcloud admin network
* workload (compute) out-of-band network
* workload (compute) provisioning network
* workload (compute) inspection network
* workload (compute) cleaning network
* internal network
* storage network
* storage management network

In addition, if the controllers are also in the ``network`` group, they are
attached to the following networks:

* public network
* external network
* tunnel network

This list may be extended by setting ``controller_extra_network_interfaces`` to a
list of names of additional networks to attach.  Alternatively, the list may be
completely overridden by setting ``controller_network_interfaces``.  These
variables are found in ``${KAYOBE_CONFIG_PATH}/controllers.yml``.

Monitoring Hosts
----------------

By default, the monitoring hosts are attached to the same networks as the
controllers when they are in the ``controllers`` group.  If the monitoring
hosts are not in the ``controllers`` group, they are attached to the following
networks by default:

* overcloud admin network
* internal network
* public network

This list may be extended by setting ``monitoring_extra_network_interfaces`` to
a list of names of additional networks to attach.  Alternatively, the list may
be completely overridden by setting ``monitoring_network_interfaces``.  These
variables are found in ``${KAYOBE_CONFIG_PATH}/monitoring.yml``.

Virtualised Compute Hosts
-------------------------

By default, virtualised compute hosts are attached to the following networks:

* overcloud admin network
* internal network
* storage network
* tunnel network

This list may be extended by setting ``compute_extra_network_interfaces`` to a
list of names of additional networks to attach.  Alternatively, the list may be
completely overridden by setting ``compute_network_interfaces``.  These
variables are found in ``${KAYOBE_CONFIG_PATH}/compute.yml``.

Other Hosts
-----------

If additional hosts are managed by kayobe, the networks to which these hosts
are attached may be defined in a host or group variables file.  See
:ref:`control-plane-service-placement` for further details.

Complete Example
================

The following example combines the complete network configuration into a single
system configuration.  In our example cloud we have three networks:
``management``, ``cloud`` and ``external``:

.. parsed-literal::

                 +------------+         +----------------+             +----------------+
                 |            |         |                +-+           |                +-+
                 |            |         |                | +-+         |  Bare metal    | +-+
                 |    Seed    |         |  Cloud hosts   | | |         |  compute hosts | | |
                 |            |         |                | | |         |                | | |
                 |            |         |                | | |         |                | | |
                 +-----+------+         +----------------+ | |         +----------------+ | |
                       |                 +-----------------+ |          +-----------------+ |
                       |                   +-----------------+            +-----------------+
                       |                        |  |  |                           |
                       |                        |  |  |                           |
                       |                        |  |  |                           |
                       |                        |  |  |                           |
   management +--------+------------------------+----------------------------------------------+
                                                   |  |                           |
   cloud      +------------------------------------+------------------------------+------------+
                                                      |
   external   +---------------------------------------+----------------------------------------+

The ``management`` network is used to access the servers' BMCs and by the seed
to inspect and provision the cloud hosts.  The ``cloud`` network carries all
internal control plane and storage traffic, and is used by the control plane to
provision the bare metal compute hosts.  Finally, the ``external`` network
links the cloud to the outside world.

We could describe such a network as follows:

.. code-block:: yaml
   :caption: ``networks.yml``

   ---
   # Network role mappings.
   oob_oc_net_name: management
   provision_oc_net_name: management
   oob_wl_net_name: management
   provision_wl_net_name: cloud
   internal_net_name: cloud
   public_net_name: external
   external_net_name: external
   storage_net_name: cloud
   storage_mgmt_net_name: cloud
   inspection_net_name: cloud

   # management network definition.
   management_cidr: 10.0.0.0/24
   management_allocation_pool_start: 10.0.0.1
   management_allocation_pool_end: 10.0.0.127
   management_inspection_allocation_pool_start: 10.0.0.128
   management_inspection_allocation_pool_end: 10.0.0.254

   # cloud network definition.
   cloud_cidr: 10.0.1.0/24
   cloud_allocation_pool_start: 10.0.1.1
   cloud_allocation_pool_end: 10.0.1.127
   cloud_inspection_allocation_pool_start: 10.0.1.128
   cloud_inspection_allocation_pool_end: 10.0.1.195
   cloud_neutron_allocation_pool_start: 10.0.1.196
   cloud_neutron_allocation_pool_end: 10.0.1.254

   # external network definition.
   external_cidr: 10.0.3.0/24
   external_allocation_pool_start: 10.0.3.1
   external_allocation_pool_end: 10.0.3.127
   external_neutron_allocation_pool_start: 10.0.3.128
   external_neutron_allocation_pool_end: 10.0.3.254
   external_routes:
     - cidr 10.0.4.0/24
       gateway: 10.0.3.1

We can map these networks to network interfaces on the seed and controller hosts:

.. code-block:: yaml
   :caption: ``inventory/group_vars/seed/network-interfaces``

   ---
   management_interface: eth0

.. code-block:: yaml
   :caption: ``inventory/group_vars/controllers/network-interfaces``

   ---
   management_interface: eth0
   cloud_interface: breth1
   cloud_bridge_ports:
     - eth1
   external_interface: eth2

We have defined a bridge for the cloud network on the controllers as this will
allow it to be plugged into a neutron Open vSwitch bridge.

Kayobe will allocate IP addresses for the hosts that it manages:

.. code-block:: yaml
   :caption: ``network-allocation.yml``

   ---
   management_ips:
     seed: 10.0.0.1
     control0: 10.0.0.2
     control1: 10.0.0.3
     control2: 10.0.0.4
   cloud_ips:
     control0: 10.0.1.1
     control1: 10.0.1.2
     control2: 10.0.1.3
   external_ips:
     control0: 10.0.3.1
     control1: 10.0.3.2
     control2: 10.0.3.3

Note that although this file does not need to be created manually, doing so
allows for a predictable IP address mapping which may be desirable in some
cases.
