# ha_cluster_modified

An Ansible role for managing High Availability Clustering, customized for <redacted>.  Original READme.md (for ha_cluster) can be found here: https://github.com/linux-system-roles/ha_cluster#readme .  For <redacted>  modifications not found in official ha_cluster documentation, search this page for 'SoCo'.  ha_cluster_modified is not an official role published by Red Hat.  It is a customized variation of the ha_cluster role, developed by Cory Beaver for the means of meeting engagement objectives.

## Limitations

* Supported OS: RHEL 8.3+, Fedora 31+
* Systems running RHEL are expected to be registered and have High-Availability
  repositories accessible.
* The role replaces the configuration of HA Cluster on specified nodes. Any
  settings not specified in the role variables will be lost.
* The role is capable of configuring:
  * single-link or multi-link cluster
  * Corosync transport options including compression and encryption
  * Corosync totem options
  * Corosync quorum options
  * SBD
  * Pacemaker cluster properties
  * stonith and resources
  * resource constraints

## Role Variables

### Defined in `defaults/main.yml`

#### `ha_cluster_enable_repos`

boolean, default: `yes`

RHEL and CentOS only, enable repositories contaning needed packages

#### `ha_cluster_cluster_present`

boolean, default: `yes`

If set to `yes`, HA cluster will be configured on the hosts according to other
variables. If set to `no`, all HA Cluster configuration will be purged from
target hosts.

#### `ha_cluster_start_on_boot`

boolean, default: `yes`

If set to `yes`, cluster services will be configured to start on boot. If set
to `no`, cluster services will be configured not to start on boot.

#### `ha_cluster_fence_agent_packages`

list of fence agent packages to install, default: fence-agents-all, fence-virt

#### `ha_cluster_extra_packages`

list of additional packages to be installed, default: no packages

This variable can be used to install additional packages not installed
automatically by the role, for example custom resource agents.

It is possible to specify fence agents here as well. However,
`ha_cluster_fence_agent_packages` is preferred for that, so that its default
value is overriden.

#### `ha_cluster_hacluster_password`

string, no default - must be specified

Password of the `hacluster` user. This user has full access to a cluster. It is
recommended to vault encrypt the value, see
https://docs.ansible.com/ansible/latest/user_guide/vault.html for details.

#### `ha_cluster_corosync_key_src`

path to Corosync authkey file, default: `null`

Authentication and encryption key for Corosync communication. It is highly
recommended to have a unique value for each cluster. The key should be 256
bytes of random data.

If value is provided, it is recommended to vault encrypt it. See
https://docs.ansible.com/ansible/latest/user_guide/vault.html for details.

If no key is specified, a key already present on the nodes will be used. If
nodes don't have the same key, a key from one node will be distributed to other
nodes so that all nodes have the same key. If no node has a key, a new key will
be generated and distributed to the nodes.

If this variable is set, `ha_cluster_regenerate_keys` is ignored for this key.

#### `ha_cluster_pacemaker_key_src`

path to Pacemaker authkey file, default: `null`

Authentication and encryption key for Pacemaker communication. It is highly
recommended to have a unique value for each cluster. The key should be 256
bytes of random data.

If value is provided, it is recommended to vault encrypt it. See
https://docs.ansible.com/ansible/latest/user_guide/vault.html for details.

If no key is specified, a key already present on the nodes will be used. If
nodes don't have the same key, a key from one node will be distributed to other
nodes so that all nodes have the same key. If no node has a key, a new key will
be generated and distributed to the nodes.

If this variable is set, `ha_cluster_regenerate_keys` is ignored for this key.

#### `ha_cluster_fence_virt_key_src`

path to fence-virt or fence-xvm pre-shared key file, default: `null`

Authentication key for fence-virt or fence-xvm fence agent.

If value is provided, it is recommended to vault encrypt it. See
https://docs.ansible.com/ansible/latest/user_guide/vault.html for details.

If no key is specified, a key already present on the nodes will be used. If
nodes don't have the same key, a key from one node will be distributed to other
nodes so that all nodes have the same key. If no node has a key, a new key will
be generated and distributed to the nodes.

If this variable is set, `ha_cluster_regenerate_keys` is ignored for this key.

If you let the role to generate new key, you are supposed to copy the key to
your nodes' hypervisor to ensure that fencing works.

#### `ha_cluster_pcsd_public_key_src`, `ha_cluster_pcsd_private_key_src`

path to pcsd TLS certificate and key, default: `null`

TLS certificate and private key for pcsd. If this is not specified, a
certificate - key pair already present on the nodes will be used. If
certificate - key pair is not present, a random new one will be generated.

If private key value is provided, it is recommended to vault encrypt it. See
https://docs.ansible.com/ansible/latest/user_guide/vault.html for details.

If these variables are set, `ha_cluster_regenerate_keys` is ignored for this
certificate - key pair.

#### `ha_cluster_regenerate_keys`

boolean, default: `no`

If this is set to `yes`, pre-shared keys and TLS certificates will be
regenerated.
See also:
[`ha_cluster_corosync_key_src`](#ha_cluster_corosync_key_src),
[`ha_cluster_pacemaker_key_src`](#ha_cluster_pacemaker_key_src),
[`ha_cluster_fence_virt_key_src`](#ha_cluster_fence_virt_key_src),
[`ha_cluster_pcsd_public_key_src`](#ha_cluster_pcsd_public_key_src-ha_cluster_pcsd_private_key_src),
[`ha_cluster_pcsd_private_key_src`](#ha_cluster_pcsd_public_key_src-ha_cluster_pcsd_private_key_src)

#### `ha_cluster_pcs_permission_list`

structure and default value:
```yaml
ha_cluster_pcs_permission_list:
  - type: group
    name: haclient
    allow_list:
      - grant
      - read
      - write
```

This configures permissions to manage a cluster using pcsd. The items are as
follows:

* `type` - `user` or `group`
* `name` - user or group name
* `allow_list` - Allowed actions for the specified user or group:
  * `read` - allows to view cluster status and settings
  * `write` - allows to modify cluster settings except permissions and ACLs
  * `grant` - allows to modify cluster permissions and ACLs
  * `full` - allows unrestricted access to a cluster including adding and
    removing nodes and access to keys and certificates

#### `ha_cluster_cluster_name`

string, default: `my-cluster`

Name of the cluster.

#### `ha_cluster_transport`

structure, default: no settings

```yaml
ha_cluster_transport:
  type: knet
  options:
    - name: option1_name
      value: option1_value
    - name: option2_name
      value: option2_value
  links:
    -
      - name: option1_name
        value: option1_value
      - name: option2_name
        value: option2_value
    -
      - name: option1_name
        value: option1_value
      - name: option2_name
        value: option2_value
  compression:
    - name: option1_name
      value: option1_value
    - name: option2_name
      value: option2_value
  crypto:
    - name: option1_name
      value: option1_value
    - name: option2_name
      value: option2_value
```

* `type` (optional) - Transport type: `knet`, `udp` or `udpu`. Defaults to
  `knet` if not specified.
* `options` (optional) - List of name-value dictionaries with transport options.
* `links` (optional) - List of lists of name-value dictionaries. Each list of
  name-value dictionaries holds options for one Corosync link. It is
  recommended to set `linknumber` value for each link. Otherwise, the first
  list of dictionaries is assigned to the first link, the second one to the
  second link and so on. Only one link is supported with `udp` and `udpu`
  transports.
* `compression` (optional) - List of name-value dictionaries configuring
  transport compression. Only available for `knet` transport.
* `crypto` (optional) - List of name-value dictionaries configuring transport
  encryption. Only available for `knet` transport, where encryption is enabled
  by default. Encryption is always disabled with `udp` and `udpu` transports.

For a list of allowed options, see `pcs -h cluster setup` or `pcs(8)` man page,
section 'cluster', command 'setup'. For a detailed description, see
`corosync.conf(5)` man page.

You may take a look at [an example](#advanced-corosync-configuration).

#### `ha_cluster_totem`

structure, default: no totem settings

```yaml
ha_cluster_totem:
  options:
    - name: option1_name
      value: option1_value
    - name: option2_name
      value: option2_value
```

Corosync totem configuration. For a list of allowed options, see `pcs -h
cluster setup` or `pcs(8)` man page, section 'cluster', command 'setup'. For a
detailed description, see `corosync.conf(5)` man page.

You may take a look at [an example](#advanced-corosync-configuration).

#### `ha_cluster_quorum`

structure, default: no quorum settings

```yaml
ha_cluster_quorum:
  options:
    - name: option1_name
      value: option1_value
    - name: option2_name
      value: option2_value
```

Cluster quorum configuration. For now, it is possible to configure quorum
options: `auto_tie_breaker`, `last_man_standing`, `last_man_standing_window`,
`wait_for_all`. Quorum options are documented in `votequorum(5)` man page.

You may take a look at [an example](#advanced-corosync-configuration).

#### `ha_cluster_sbd_enabled`

boolean, default: `no`

Defines whether to use SBD.

You may take a look at [an example](#configuring-cluster-to-use-sbd).

#### `ha_cluster_sbd_options`

list, default: `[]`

List of name-value dictionaries specifying SBD options. Supported options are:
`delay-start` (defaults to `no`), `startmode` (defaults to `always`),
`timeout-action` (defaults to `flush,reboot`) and `watchdog-timeout` (defaults
to `5`). See `sbd(8)` man page, section 'Configuration via environment' for
their description.

You may take a look at [an example](#configuring-cluster-to-use-sbd).

Watchdog and SBD devices are configured on a node to node basis in
[inventory](#sbd-watchdog-and-devices).

#### `ha_cluster_cluster_properties`

structure, default: no properties

```yaml
ha_cluster_cluster_properties:
  - attrs:
      - name: property1_name
        value: property1_value
      - name: property2_name
        value: property2_value
```

List of sets of cluster properties - Pacemaker cluster-wide configuration.
Currently, only one set is supported.

You may take a look at [an example](#configuring-cluster-properties).

#### `ha_cluster_resource_primitives`

structure, default: no resources

```yaml
ha_cluster_resource_primitives:
  - id: resource-id
    agent: resource-agent
    instance_attrs:
      - attrs:
          - name: attribute1_name
            value: attribute1_value
          - name: attribute2_name
            value: attribute2_value
    meta_attrs:
      - attrs:
          - name: meta_attribute1_name
            value: meta_attribute1_value
          - name: meta_attribute2_name
            value: meta_attribute2_value
    operations:
      - action: operation1-action
        attrs:
          - name: operation1_attribute1_name
            value: operation1_attribute1_value
          - name: operation1_attribute2_name
            value: operation1_attribute2_value
      - action: operation2-action
        attrs:
          - name: operation2_attribute1_name
            value: operation2_attribute1_value
          - name: operation2_attribute2_name
            value: operation2_attribute2_value
```

This variable defines Pacemaker resources (including stonith) configured by the
role. The items are as follows:

* `id` (mandatory) - ID of a resource.
* `agent` (mandatory) - Name of a resource or stonith agent, for example
  `ocf:pacemaker:Dummy` or `stonith:fence_xvm`. It is mandatory to use
  `stonith:` for stonith agents. For resource agents, it is possible to use a
  short name, such as `Dummy` instead of `ocf:pacemaker:Dummy`. However, if
  several agents with the same short name are installed, the role will fail as
  it will be unable to decide which agent should be used. Therefore, it is
  recommended to use full names.
* `instance_attrs` (optional) - List of sets of the resource's instance
  attributes. Currently, only one set is supported. The exact names and values
  of attributes, as well as whether they are mandatory or not, depends on the
  resource or stonith agent.
* `meta_attrs` (optional) - List of sets of the resource's meta attributes.
  Currently, only one set is supported.
* `operations` (optional) - List of the resource's operations.
  * `action` (mandatory) - Operation action as defined by Pacemaker and the
    resource or stonith agent.
  * `attrs` (mandatory) - Operation options, at least one option must be
    specified.

You may take a look at
[an example](#creating-a-cluster-with-fencing-and-several-resources).

#### `ha_cluster_resource_groups`

structure, default: no resource groups

```yaml
ha_cluster_resource_groups:
  - id: group-id
    resource_ids:
      - resource1-id
      - resource2-id
    meta_attrs:
      - attrs:
          - name: group_meta_attribute1_name
            value: group_meta_attribute1_value
          - name: group_meta_attribute2_name
            value: group_meta_attribute2_value
```

This variable defines resource groups. The items are as follows:

* `id` (mandatory) - ID of a group.
* `resources` (mandatory) - List of the group's resources. Each resource is
  referenced by its ID and the resources must be defined in
  [`ha_cluster_resource_primitives`](#ha_cluster_resource_primitives). At least
  one resource must be listed.
* `meta_attrs` (optional) - List of sets of the group's meta attributes.
  Currently, only one set is supported.

You may take a look at
[an example](#creating-a-cluster-with-fencing-and-several-resources).

#### `ha_cluster_resource_clones`

structure, default: no resource clones

```yaml
ha_cluster_resource_clones:
  - resource_id: resource-to-be-cloned
    promotable: yes
    id: custom-clone-id
    meta_attrs:
      - attrs:
          - name: clone_meta_attribute1_name
            value: clone_meta_attribute1_value
          - name: clone_meta_attribute2_name
            value: clone_meta_attribute2_value
```

This variable defines resource clones. The items are as follows:

* `resource_id` (mandatory) - Resource to be cloned. The resource must be
  defined in
  [`ha_cluster_resource_primitives`](#ha_cluster_resource_primitives) or
  [`ha_cluster_resource_groups`](#ha_cluster_resource_groups).
* `promotable` (optional) - Create a promotable clone, yes or no.
* `id` (optional) - Custom ID of the clone. If no ID is specified, it will be
  generated. Warning will be emitted if this option is not supported by the
  cluster.
* `meta_attrs` (optional) - List of sets of the clone's meta attributes.
  Currently, only one set is supported.

You may take a look at
[an example](#creating-a-cluster-with-fencing-and-several-resources).

#### `ha_cluster_resource_bundles`

structure, default: no bundle resources

```yaml
- id: bundle-id
  resource_id: resource-id
  container:
    type: container-type
    options:
      - name: container_option1_name
        value: container_option1_value
      - name: container_option2_name
        value: container_option2_value
  network_options:
      - name: network_option1_name
        value: network_option1_value
      - name: network_option2_name
        value: network_option2_value
  port_map:
    -
      - name: option1_name
        value: option1_value
      - name: option2_name
        value: option2_value
    -
      - name: option1_name
        value: option1_value
      - name: option2_name
        value: option2_value
  storage_map:
    -
      - name: option1_name
        value: option1_value
      - name: option2_name
        value: option2_value
    -
      - name: option1_name
        value: option1_value
      - name: option2_name
        value: option2_value
    meta_attrs:
      - attrs:
          - name: bundle_meta_attribute1_name
            value: bundle_meta_attribute1_value
          - name: bundle_meta_attribute2_name
            value: bundle_meta_attribute2_value
```

This variable defines resource bundles. The items are as follows:

* `id` (mandatory) - ID of a bundle.
* `resource_id` (optional) - Resource to be placed into the bundle. The
  resource must be defined in
  [`ha_cluster_resource_primitives`](#ha_cluster_resource_primitives).
* `container.type` (mandatory) - Container technology, such as `docker`,
  `podman` or `rkt`.
* `container.options` (optional) - List of name-value dictionaries. Depending
  on the selected `container.type`, certain options may be required to be
  specified, such as `image`.
* `network_options` (optional) - List of name-value dictionaries. When a
  resource is placed into a bundle, some `network_options` may be required to
  be specified, such as `control-port` or `ip-range-start`.
* `port_map` (optional) - List of lists of name-value dictionaries defining
  port forwarding from the host network to the container network. Each list of
  name-value dictionaries holds options for one port forwarding.
* `storage_map` (optional) - List of lists of name-value dictionaries mapping
  directories on the host's filesystem into the container. Each list of
  name-value dictionaries holds options for one directory mapping.
* `meta_attrs` (optional) - List of sets of the bundle's meta attributes.
  Currently, only one set is supported.

Note, that the role does not install container launch technology automatically.
However, you can install it by listing appropriate packages in
[`ha_cluster_extra_packages`](#ha_cluster_extra_packages) variable.

Note, that the role does not build and distribute container images. Please, use
other means to supply a fully configured container image to every node allowed
to run a bundle depending on it.

You may take a look at
[an example](#creating-a-cluster-with-fencing-and-several-resources).

#### `ha_cluster_constraints_location`

structure, default: no constraints

This variable defines resource location constraints. They tell the cluster
which nodes a resource can run on. Resources can be specified by their ID or a
pattern matching more resources. Nodes can be specified by their name or a
rule.

Structure for constraints with resource ID and node name:
```yaml
ha_cluster_constraints_location:
  - resource:
      id: resource-id
    node: node-name
    id: constraint-id
    options:
      - name: score
        value: score-value
      - name: option-name
        value: option-value
```
* `resource` (mandatory) - Specification of a resource the constraint applies
  to.
* `node` (mandatory) - Name of a node the resource should prefer or avoid.
* `id` (optional) - ID of the constraint. If not specified, it will be
  autogenerated.
* `options` (optional) - List of name-value dictionaries.
  * `score` - Score sets weight of the constraint.
    * Positive value means the resource prefers running on the node.
    * Negative value means the resource should avoid running on the node.
    * `-INFINITY` means the resource must avoid running on the node.
    * If not specified, `score` defaults to `INFINITY`.

You may take a look at
[an example](#creating-a-cluster-with-resource-constraints).

Structure for constraints with resource pattern and node name:
```yaml
ha_cluster_constraints_location:
  - resource:
      pattern: resource-pattern
    node: node-name
    id: constraint-id
    options:
      - name: score
        value: score-value
      - name: resource-discovery
        value: resource-discovery-value
```
* This is the same as the previous type, except the resource specification.
* `pattern` (mandatory) - POSIX extended regular expression resource IDs are
  matched against.

You may take a look at
[an example](#creating-a-cluster-with-resource-constraints).

Structure for constraints with resource ID and a rule:
```yaml
ha_cluster_constraints_location:
  - resource:
      id: resource-id
      role: resource-role
    rule: rule-string
    id: constraint-id
    options:
      - name: score
        value: score-value
      - name: resource-discovery
        value: resource-discovery-value
```
* `resource` (mandatory) - Specification of a resource the constraint applies
  to.
  * `id` (mandatory) - Resource ID.
  * `role` (optional) - You may limit the constraint to the specified resource
    role: `Started`, `Unpromoted`, `Promoted`.
* `rule` (mandatory) - Constraint rule written using pcs syntax. See `pcs(8)`
  man page, section `constraint location` for details.
* Other items have the same meaning as above.

You may take a look at
[an example](#creating-a-cluster-with-resource-constraints).

Structure for constraints with resource pattern and a rule:
```yaml
ha_cluster_constraints_location:
  - resource:
      pattern: resource-pattern
      role: resource-role
    rule: rule-string
    id: constraint-id
    options:
      - name: score
        value: score-value
      - name: resource-discovery
        value: resource-discovery-value
```
* This is the same as the previous type, except the resource specification.
* `pattern` (mandatory) - POSIX extended regular expression resource IDs are
  matched against.

You may take a look at
[an example](#creating-a-cluster-with-resource-constraints).

#### `ha_cluster_constraints_colocation`

structure, default: no constraints

This variable defines resource colocation constraints. They tell the cluster
that the location of one resource depends on the location of another one. There
are two types of colocation constraints: a simple one for two resources, and a
set constraint for multiple resources.

Structure for simple constraints:
```yaml
ha_cluster_constraints_colocation:
  - resource_follower:
      id: resource-id1
      role: resource-role1
    resource_leader:
      id: resource-id2
      role: resource-role2
    id: constraint-id
    options:
      - name: score
        value: score-value
      - name: option-name
        value: option-value
```
* `resource_follower` (mandatory) - A resource that should be located relative
  to `resource_leader`.
  * `id` (mandatory) - Resource ID.
  * `role` (optional) - You may limit the constraint to the specified resource
    role: `Started`, `Unpromoted`, `Promoted`.
* `resource_leader` (mandatory) - The cluster will decide where to put this
  resource first and then decide where to put `resource_follower`.
  * `id` (mandatory) - Resource ID.
  * `role` (optional) - You may limit the constraint to the specified resource
    role: `Started`, `Unpromoted`, `Promoted`.
* `id` (optional) - ID of the constraint. If not specified, it will be
  autogenerated.
* `options` (optional) - List of name-value dictionaries.
  * `score` (optional) - Score sets weight of the constraint.
    * Positive values indicate the resources should run on the same node.
    * Negative values indicate the resources should run on different nodes.
    * Values of `+INFINITY` and `-INFINITY` change "should" to "must".
    * If not specified, `score` defaults to `INFINITY`.

You may take a look at
[an example](#creating-a-cluster-with-resource-constraints).

Structure for set constraints:
```yaml
ha_cluster_constraints_colocation:
  - resource_sets:
      - resource_ids:
          - resource-id1
          - resource-id2
        options:
          - name: option-name
            value: option-value
    id: constraint-id
    options:
      - name: score
        value: score-value
      - name: option-name
        value: option-value
```
* `resource_sets` (mandatory) - List of resource sets.
  * `resource_ids` (mandatory) - List of resources in a set.
  * `options` (optional) - List of name-value dictionaries fine-tuning how
    resources in the sets are treated by the constraint.
* `id` (optional) - Same as above.
* `options` (optional) - Same as above.

You may take a look at
[an example](#creating-a-cluster-with-resource-constraints).

#### `ha_cluster_constraints_order`

structure, default: no constraints

This variable defines resource order constraints. They tell the cluster the
order in which certain resource actions should occur. There are two types of
order constraints: a simple one for two resources, and a set constraint for
multiple resources.

Structure for simple constraints:
```yaml
ha_cluster_constraints_order:
  - resource_first:
      id: resource-id1
      action: resource-action1
    resource_then:
      id: resource-id2
      action: resource-action2
    id: constraint-id
    options:
      - name: score
        value: score-value
      - name: option-name
        value: option-value
```
* `resource_first` (mandatory) - Resource that the `resource_then` depends on.
  * `id` (mandatory) - Resource ID.
  * `action` (optional) - The action that the resource must complete before  an
    action can be initiated for the `resource_then`. Allowed values: `start`,
    `stop`, `promote`, `demote`.
* `resource_then` (mandatory) - The dependent resource.
  * `id` (mandatory) - Resource ID.
  * `action` (optional) - The action that the resource can execute only after
    the action on the `resource_first` has completed. Allowed values: `start`,
    `stop`, `promote`, `demote`.
* `id` (optional) - ID of the constraint. If not specified, it will be
  autogenerated.
* `options` (optional) - List of name-value dictionaries.

You may take a look at
[an example](#creating-a-cluster-with-resource-constraints).

Structure for set constraints:
```yaml
ha_cluster_constraints_order:
  - resource_sets:
      - resource_ids:
          - resource-id1
          - resource-id2
        options:
          - name: option-name
            value: option-value
    id: constraint-id
    options:
      - name: score
        value: score-value
      - name: option-name
        value: option-value
```
* `resource_sets` (mandatory) - List of resource sets.
  * `resource_ids` (mandatory) - List of resources in a set.
  * `options` (optional) - List of name-value dictionaries fine-tuning how
    resources in the sets are treated by the constraint.
* `id` (optional) - Same as above.
* `options` (optional) - Same as above.

You may take a look at
[an example](#creating-a-cluster-with-resource-constraints).

#### `ha_cluster_constraints_ticket`

structure, default: no constraints

This variable defines resource ticket constraints. They let you specify the
resources depending on a certain ticket. There are two types of ticket
constraints: a simple one for two resources, and a set constraint for multiple
resources.

Structure for simple constraints:
```yaml
ha_cluster_constraints_ticket:
  - resource:
      id: resource-id
      role: resource-role
    ticket: ticket-name
    id: constraint-id
    options:
      - name: loss-policy
        value: loss-policy-value
      - name: option-name
        value: option-value
```
* `resource` (mandatory) - Specification of a resource the constraint applies
  to.
  * `id` (mandatory) - Resource ID.
  * `role` (optional) - You may limit the constraint to the specified resource
    role: `Started`, `Unpromoted`, `Promoted`.
* `ticket` (mandatory) - Name of a ticket the resource depends on.
* `id` (optional) - ID of the constraint. If not specified, it will be
  autogenerated.
* `options` (optional) - List of name-value dictionaries.
  * `loss-policy` (optional) - Action that should happen to the resource if the
    ticket is revoked.

You may take a look at
[an example](#creating-a-cluster-with-resource-constraints).

Structure for set constraints:
```yaml
ha_cluster_constraints_ticket:
  - resource_sets:
      - resource_ids:
          - resource-id1
          - resource-id2
        options:
          - name: option-name
            value: option-value
    ticket: ticket-name
    id: constraint-id
    options:
      - name: option-name
        value: option-value
```
* `resource_sets` (mandatory) - List of resource sets.
  * `resource_ids` (mandatory) - List of resources in a set.
  * `options` (optional) - List of name-value dictionaries fine-tuning how
    resources in the sets are treated by the constraint.
* `ticket` (mandatory) - Same as above.
* `id` (optional) - Same as above.
* `options` (optional) - Same as above.

You may take a look at
[an example](#creating-a-cluster-with-resource-constraints).

#### `qdevice` 

SoCo -- Package to be installed on cluster node (used in tasks/check-and-prepare-role-variables.yml, line 66)

### `qnetd` 

SoCo -- Package to be installed on quorum device (used in tasks/check-and-prepare-role-variables.yml, line 66)

### Defined in `vars/main.yml`

SoCo - The following variables are modified to fit <redacted> needs:

```yaml
__ha_cluster_fullstack_node_packages:
  - corosync
  - libknet1-plugins-all
  - resource-agents
  - pacemaker
  - pcs
  - watchdog # installing watchdog to enable fencing to start on boot - SoCo.
  - openssl  # used in the role for generating preshared keys

__ha_cluster_services:
  - corosync
  - corosync-qdevice
  - pacemaker
  - watchdog # added for <redacted> engagement - SoCo
  - lvmlockd # added for <redacted> engagement - SoCo
```

### Inventory

#### Nodes' names and addresses
Nodes' names and addresses can be configured in inventory. This is optional. If
no names or addresses are configured, play's targets will be used.

Example inventory with targets `node1` and `node2`:
```yaml
all:
  hosts:
    node1:
      ha_cluster:
        node_name: node-A
        pcs_address: node1-address
        corosync_addresses:
          - 192.168.1.11
          - 192.168.2.11
      mp_key: "0x10"
    node2:
      ha_cluster:
        node_name: node-B
        pcs_address: node2-address:2224
        corosync_addresses:
          - 192.168.1.12
          - 192.168.2.12
      mp_key: "0x11"
```

* `node_name` - the name of a node in a cluster
* `pcs_address` - an address used by pcs to communicate with the node, it can
  be a name, FQDN or an IP address and it can contain port
* `corosync_addresses` - list of addresses used by Corosync, all nodes must
  have the same number of addresses and the order of the addresses matters
* `mp_key` - SoCo - reservation_key value to be used in /etc/multipath.conf (https://access.redhat.com/solutions/2766611)

#### SBD watchdog and devices
When using SBD, you may optionally configure watchdog and SBD devices for each
node in inventory. Even though all SBD devices must be shared to and accesible
from all nodes, each node may use different names for the devices. Watchdog may
be different for each node as well. See also [SBD
variables](#ha_cluster_sbd_enabled).

Example inventory with targets `node1` and `node2`:
```yaml
all:
  hosts:
    node1:
      ha_cluster:
        sbd_watchdog: /dev/watchdog2
        sbd_devices:
          - /dev/vdx
          - /dev/vdy
    node2:
      ha_cluster:
        sbd_watchdog: /dev/watchdog1
        sbd_devices:
          - /dev/vdw
          - /dev/vdz
```

* `sbd_watchdog` (optional) - Watchdog device to be used by SBD. Defaults to
  `/dev/watchdog` if not set.
* `sbd_devices` (optional) - Devices to use for exchanging SBD messages and for
  monitoring. Defaults to empty list if not set.

## Example Playbooks

Following examples show what the structure of the role variables looks like.
They are not guides or best practices for configuring a cluster.

### Creating a cluster running no resources
```yaml
- hosts: node1 node2
  vars:
    ha_cluster_cluster_name: my-new-cluster
    ha_cluster_hacluster_password: password

  roles:
    - rhel-system-roles.ha_cluster
```

### Advanced Corosync configuration
```yaml
- hosts: node1 node2
  vars:
    ha_cluster_cluster_name: my-new-cluster
    ha_cluster_hacluster_password: password
    ha_cluster_transport:
      type: knet
      options:
        - name: ip_version
          value: ipv4-6
        - name: link_mode
          value: active
      links:
        -
          - name: linknumber
            value: 1
          - name: link_priority
            value: 5
        -
          - name: linknumber
            value: 0
          - name: link_priority
            value: 10
      compression:
        - name: level
          value: 5
        - name: model
          value: zlib
      crypto:
        - name: cipher
          value: none
        - name: hash
          value: none
    ha_cluster_totem:
      options:
        - name: block_unlisted_ips
          value: 'yes'
        - name: send_join
          value: 0
    ha_cluster_quorum:
      options:
        - name: auto_tie_breaker
          value: 1
        - name: wait_for_all
          value: 1

  roles:
    - rhel-system-roles.ha_cluster
```

### Configuring cluster to use SBD
```yaml
- hosts: node1 node2
  vars:
    ha_cluster_cluster_name: my-new-cluster
    ha_cluster_hacluster_password: password
    ha_cluster_sbd_enabled: yes
    ha_cluster_sbd_options:
      - name: delay-start
        value: 'no'
      - name: startmode
        value: always
      - name: timeout-action
        value: 'flush,reboot'
      - name: watchdog-timeout
        value: 5

  roles:
    - rhel-system-roles.ha_cluster
```

### Configuring cluster properties
```yaml
- hosts: node1 node2
  vars:
    ha_cluster_cluster_name: my-new-cluster
    ha_cluster_hacluster_password: password
    ha_cluster_cluster_properties:
      - attrs:
          - name: stonith-enabled
            value: 'true'
          - name: no-quorum-policy
            value: stop

  roles:
    - rhel-system-roles.ha_cluster
```

### Creating a cluster with fencing and several resources
```yaml
- hosts: node1 node2
  vars:
    ha_cluster_cluster_name: my-new-cluster
    ha_cluster_hacluster_password: password
    ha_cluster_resource_primitives:
      - id: xvm-fencing
        agent: 'stonith:fence_xvm'
        instance_attrs:
          - attrs:
              - name: pcmk_host_list
                value: node1 node2
      - id: simple-resource
        agent: 'ocf:pacemaker:Dummy'
      - id: resource-with-options
        agent: 'ocf:pacemaker:Dummy'
        instance_attrs:
          - attrs:
              - name: fake
                value: fake-value
              - name: passwd
                value: passwd-value
        meta_attrs:
          - attrs:
              - name: target-role
                value: Started
              - name: is-managed
                value: 'true'
        operations:
          - action: start
            attrs:
              - name: timeout
                value: '30s'
          - action: monitor
            attrs:
              - name: timeout
                value: '5'
              - name: interval
                value: '1min'
      - id: dummy-1
        agent: 'ocf:pacemaker:Dummy'
      - id: dummy-2
        agent: 'ocf:pacemaker:Dummy'
      - id: dummy-3
        agent: 'ocf:pacemaker:Dummy'
      - id: simple-clone
        agent: 'ocf:pacemaker:Dummy'
      - id: clone-with-options
        agent: 'ocf:pacemaker:Dummy'
      - id: bundled-resource
        agent: 'ocf:pacemaker:Dummy'
    ha_cluster_resource_groups:
      - id: simple-group
        resource_ids:
          - dummy-1
          - dummy-2
        meta_attrs:
          - attrs:
              - name: target-role
                value: Started
              - name: is-managed
                value: 'true'
      - id: cloned-group
        resource_ids:
          - dummy-3
    ha_cluster_resource_clones:
      - resource_id: simple-clone
      - resource_id: clone-with-options
        promotable: yes
        id: custom-clone-id
        meta_attrs:
          - attrs:
              - name: clone-max
                value: '2'
              - name: clone-node-max
                value: '1'
      - resource_id: cloned-group
        promotable: yes
    ha_cluster_resource_bundles:
      - id: bundle-with-resource
        resource-id: bundled-resource
        container:
          type: podman
          options:
            - name: image
              value: my:image
        network_options:
          - name: control-port
            value: 3121
        port_map:
          -
            - name: port
              value: 10001
          -
            - name: port
              value: 10002
            - name: internal-port
              value: 10003
        storage_map:
          -
            - name: source-dir
              value: /srv/daemon-data
            - name: target-dir
              value: /var/daemon/data
          -
            - name: source-dir-root
              value: /var/log/pacemaker/bundles
            - name: target-dir
              value: /var/log/daemon
        meta_attrs:
          - attrs:
              - name: target-role
                value: Started
              - name: is-managed
                value: 'true'

  roles:
    - rhel-system-roles.ha_cluster
```

### Creating a cluster with resource constraints
```yaml
- hosts: node1 node2
  vars:
    ha_cluster_cluster_name: my-new-cluster
    ha_cluster_hacluster_password: password
    # In order to use constraints, we need resources the constraints will apply
    # to.
    ha_cluster_resource_primitives:
      - id: xvm-fencing
        agent: 'stonith:fence_xvm'
        instance_attrs:
          - attrs:
              - name: pcmk_host_list
                value: node1 node2
      - id: dummy-1
        agent: 'ocf:pacemaker:Dummy'
      - id: dummy-2
        agent: 'ocf:pacemaker:Dummy'
      - id: dummy-3
        agent: 'ocf:pacemaker:Dummy'
      - id: dummy-4
        agent: 'ocf:pacemaker:Dummy'
      - id: dummy-5
        agent: 'ocf:pacemaker:Dummy'
      - id: dummy-6
        agent: 'ocf:pacemaker:Dummy'
    # location constraints
    ha_cluster_constraints_location:
      # resource ID and node name
      - resource:
          id: dummy-1
        node: node1
        options:
          - name: score
            value: 20
      # resource pattern and node name
      - resource:
          pattern: dummy-\d+
        node: node1
        options:
          - name: score
            value: 10
      # resource ID and rule
      - resource:
          id: dummy-2
        rule: '#uname eq node2 and date in_range 2022-01-01 to 2022-02-28'
      # resource pattern and rule
      - resource:
          pattern: dummy-\d+
        rule: node-type eq weekend and date-spec weekdays=6-7
    # colocation constraints
    ha_cluster_constraints_colocation:
      # simple constraint
      - resource_leader:
          id: dummy-3
        resource_follower:
          id: dummy-4
        options:
          - name: score
            value: -5
      # set constraint
      - resource_sets:
          - resource_ids:
              - dummy-1
              - dummy-2
          - resource_ids:
              - dummy-5
              - dummy-6
            options:
              - name: sequential
                value: "false"
        options:
          - name: score
            value: 20
    # order constraints
    ha_cluster_constraints_order:
      # simple constraint
      - resource_first:
          id: dummy-1
        resource_then:
          id: dummy-6
        options:
          - name: symmetrical
            value: "false"
      # set constraint
      - resource_sets:
          - resource_ids:
              - dummy-1
              - dummy-2
            options:
              - name: require-all
                value: "false"
              - name: sequential
                value: "false"
          - resource_ids:
              - dummy-3
          - resource_ids:
              - dummy-4
              - dummy-5
            options:
              - name: sequential
                value: "false"
    # ticket constraints
    ha_cluster_constraints_ticket:
      # simple constraint
      - resource:
          id: dummy-1
        ticket: ticket1
        options:
          - name: loss-policy
            value: stop
      # set constraint
      - resource_sets:
          - resource_ids:
              - dummy-3
              - dummy-4
              - dummy-5
        ticket: ticket2
        options:
          - name: loss-policy
            value: fence

  roles:
    - rhel-system-roles.ha_cluster
```

### Purging all cluster configuration
```yaml
- hosts: node1 node2
  vars:
    ha_cluster_cluster_present: no

  roles:
    - rhel-system-roles.ha_cluster
```

### <redacted> Customizations - SoCo

For <redacted> cluster deployments, the workflow of your playbook looks like this:

* Deploy the quorum server (play: deploy quorum server)
* Install filesystem requirements on cluster nodes (play: Set up shared filesystems requirements)
* Select & validate LUNs presented to cluster nodes (play: configure storage (select & validate LUNs))
* Deploy first set of cluster resources (mpath,floating_ip,nfs_server,dlm,lvmlockd,locking group, locking group clone) (play: deploy cluster (mpath, lvmlockd, dlm resources))
* Initiate and configure LVM (play: configure storage (lvm & filesystems))
* Deploy lvm & filesystem resources (play: deploy cluster (lvm, filesystems))

Order of plays matters very much for these deployments.

## Playbook Variables

### Defined in `/home/ansible/ansible/secret.yml`

SoCo - This is where we define cluster name and password.

#### `cluster_name`

string, default: 

Encrypted value to pass unders `vars` section of plays, which defines the role variable `ha_cluster_cluster_name`.

#### `cluster_password`

string, default: 

Encrypted value to pass unders `vars` section of plays, which defines the role variable `ha_cluster_hacluster_password`.

### Defined in `/home/ansible/ansible/lvmlockd_dlm.yml`

SoCo - This is where we define cluster resources for the first set of resources

#### `ha_cluster_cluster_properties`

structure, default: no properties

```yaml
ha_cluster_cluster_properties: 
  - attrs:
      - name: no-quorum-policy
        value: freeze
```

#### `ha_cluster_resource_primitives`

structure, default: no resources

```yaml
ha_cluster_resource_primitives:
  - id: mpath
    agent: 'stonith:fence_mpath'
    instance_attrs:
      - attrs:
          - name: devices
            value: "{{ mpath_devices }}"  #/dev/mapper/mpathd,/dev/mapper/mpathe,/dev/mapper/mpathf
          - name: pcmk_host_map
            value: "{{ mpath_device_pcmk }}"  #hosta.domain.com:10;hostb.domain.com:11
    meta_attrs:
      - attrs:
          - name: provides
            value: unfencing
    operations:
      - action: monitor
        attrs:
          - name: interval
            value: '60s'
  - id: floating_ip
    agent: 'ocf:heartbeat:IPaddr'
    instance_attrs:
      - attrs:
          - name: ip
            value: "{{ floating_ip }}"
    operations:
      - action: monitor
        attrs:
          - name: interval
            value: '10s'
          - name: timeout
            value: '20s'
      - action: start
        attrs:
          - name: interval
            value: '0s'
          - name: timeout
            value: '20s'
      - action: stop
        attrs:
          - name: interval
            value: '0s'
          - name: timeout
            value: '20s'
  - id: nfs_server
    agent: 'service:nfs-server'
    operations:
      - action: monitor
        attrs:
          - name: id
            value: "{{ nfs_server_monitor_interval  }}"
          - name: interval
            value: '30s'
          - name: timeout
            value: '100s'
      - action: start
        attrs:
          - name: id
            value: "{{ nfs_server_start_interval }}"
          - name: interval
            value: '0s'
          - name: timeout
            value: '100s'
      - action: stop
        attrs:
          - name: id
            value: "{{ nfs_server_stop_interval }}"
          - name: interval
            value: '0s'
          - name: timeout
            value: '100s'
  - id: dlm
    agent: 'ocf:pacemaker:controld'
    operations:
      - action: monitor
        attrs:
          - name: id
            value: dlm-monitor-interval-30s
          - name: interval
            value: '30s'
          - name: on-fail
            value: fence
      - action: start
        attrs:
          - name: id
            value: dlm-start-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '90s'
      - action: stop
        attrs:
          - name: id
            value: dlm-stop-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '100s'
  - id: lvmlockd
    agent: 'ocf:heartbeat:lvmlockd'
    operations:
      - action: monitor
        attrs:
          - name: id
            value: lvmlockd-monitor-interval-30s
          - name: interval
            value: '30s'
          - name: on-fail
            value: fence
      - action: start
        attrs:
          - name: id
            value: lvmlockd-start-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '90s'
      - action: stop
        attrs:
          - name: id
            value: lvmlockd-stop-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '90s'
```
#### `ha_cluster_resource_groups`

structure, default: no resource groups

```yaml
ha_cluster_resource_groups: 
      - id: locking
        resource_ids:
          - dlm
          - lvmlockd
```

#### `ha_cluster_resource_clones`

structure, default: no resource groups

```yaml
ha_cluster_resource_clones: 
  - resource_id: locking
    meta_attrs:
      - attrs:
          - name: interleave
            value: true
```

### Defined in `/home/ansible/ansible/fs_resources.yml`

SoCo - This is where we define cluster resources for the second set of resources, focused on filesystems

In addition to what's already defined in `/home/ansible/ansible/lvmlockd_dlm.yml`, `/home/ansible/ansible/fs_resources.yml` includes the following variables:

```yaml
ha_cluster_resource_primitives: 
  - id: home_oua_lv
    agent: 'ocf:heartbeat:LVM-activate'
    instance_attrs:
      - attrs:
          - name: activation_mode
            value: shared
          - name: lvname
            value: home_oua_lv
          - name: vg_access_mode
            value: lvmlockd
          - name: vgname
            value: cldata_vg_01
    operations:
      - action: monitor
        attrs:
          - name: id
            value: home_oua_lv-monitor-interval-30s
          - name: interval
            value: '30s'
          - name: timeout
            value: '90s'
      - action: start
        attrs:
          - name: id
            value: home_oua_lv-start-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '90s'
      - action: stop
        attrs:
          - name: id
            value: home_oua_lv-stop-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '90s'
  - id: home_oma_lv
    agent: 'ocf:heartbeat:LVM-activate'
    instance_attrs:
      - attrs:
          - name: activation_mode
            value: shared
          - name: lvname
            value: home_oma_lv
          - name: vg_access_mode
            value: lvmlockd
          - name: vgname
            value: cldata_vg_02
    operations:
      - action: monitor
        attrs:
          - name: id
            value: home_oma_lv-monitor-interval-30s
          - name: interval
            value: '30s'
          - name: timeout
            value: '90s'
      - action: start
        attrs:
          - name: id
            value: home_oma_lv-start-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '90s'
      - action: stop
        attrs:
          - name: id
            value: home_oma_lv-stop-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '90s'
  - id: home_flex_lv
    agent: 'ocf:heartbeat:LVM-activate'
    instance_attrs:
      - attrs:
          - name: activation_mode
            value: shared
          - name: lvname
            value: home_flex_lv
          - name: vg_access_mode
            value: lvmlockd
          - name: vgname
            value: cldata_vg_03
    operations:
      - action: monitor
        attrs:
          - name: id
            value: home_flex_lv-monitor-interval-30s
          - name: interval
            value: '30s'
          - name: timeout
            value: '90s'
      - action: start
        attrs:
          - name: id
            value: home_flex_lv-start-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '90s'
      - action: stop
        attrs:
          - name: id
            value: home_flex_lv-stop-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '90s'
  - id: home_oua_res
    agent: 'ocf:heartbeat:Filesystem'
    instance_attrs:
      - attrs:
          - name: device
            value: '/dev/cldata_vg_01/home_oua_lv'
          - name: directory
            value: '/home_oua'
          - name: fstype
            value: gfs2
          - name: options
            value: noatime
    operations:
      - action: monitor
        attrs:
          - name: id
            value: home_oua_res-monitor-interval-10s
          - name: interval
            value: '10s'
          - name: on-fail
            value: fence
      - action: start
        attrs:
          - name: id
            value: home_oua_res-start-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '60s'
      - action: stop
        attrs:
          - name: id
            value: home_oua_res-stop-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '60s'
  - id: home_oma_res
    agent: 'ocf:heartbeat:Filesystem'
    instance_attrs:
      - attrs:
          - name: device
            value: '/dev/cldata_vg_02/home_oma_lv'
          - name: directory
            value: '/home_oma'
          - name: fstype
            value: gfs2
          - name: options
            value: noatime
    operations:
      - action: monitor
        attrs:
          - name: id
            value: home_oma_res-monitor-interval-10s
          - name: interval
            value: '10s'
          - name: on-fail
            value: fence
      - action: start
        attrs:
          - name: id
            value: home_oma_res-start-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '60s'
      - action: stop
        attrs:
          - name: id
            value: home_oma_res-stop-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '60s'
  - id: home_flex_res
    agent: 'ocf:heartbeat:Filesystem'
    instance_attrs:
      - attrs:
          - name: device
            value: '/dev/cldata_vg_03/home_flex_lv'
          - name: directory
            value: '/home_flex'
          - name: fstype
            value: gfs2
          - name: options
            value: noatime
    operations:
      - action: monitor
        attrs:
          - name: id
            value: home_flex_res-monitor-interval-10s
          - name: interval
            value: '10s'
          - name: on-fail
            value: fence
      - action: start
        attrs:
          - name: id
            value: home_flex_res-start-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '60s'
      - action: stop
        attrs:
          - name: id
            value: home_flex_res-stop-interval-0s
          - name: interval
            value: '0s'
          - name: timeout
            value: '60s'

ha_cluster_resource_groups:
      - id: cldata_vg_01
        resource_ids:
          - home_oua_lv
          - home_oua_res
      - id: cldata_vg_02
        resource_ids:
          - home_oma_lv
          - home_oma_res
      - id: cldata_vg_03
        resource_ids:
          - home_flex_lv
          - home_flex_res

ha_cluster_resource_clones:
  - resource_id: cldata_vg_01
    meta_attrs:
      - attrs:
          - name: interleave
            value: true
  - resource_id: cldata_vg_02
    meta_attrs:
      - attrs:
          - name: interleave
            value: true
  - resource_id: cldata_vg_03
    meta_attrs:
      - attrs:
          - name: interleave
            value: true

ha_cluster_constraints_colocation:
  - resource_sets:
      - resource_ids:
          - cldata_vg_01-clone
          - cldata_vg_02-clone
          - cldata_vg_03-clone
          - locking-clone

ha_cluster_constraints_order:
  - resource_first:
      id: locking-clone
    resource_then:
      id: cldata_vg_01-clone
  - resource_first:
      id: locking-clone
    resource_then:
      id: cldata_vg_02-clone
  - resource_first:
      id: locking-clone
    resource_then:
      id: cldata_vg_03-clone
```

## SoCo - Customized tasks files

* pcs-auth-quorum.yml
* pcs-auth-pcs-0.10-quorum.yml
* quorum_services.yml

### pcs-auth-quorum.yml

This tasks file runs against cluster nodes and checks authentication status of the quorum server.  If not already authenticated, pcs-auth-pcs-0.10-quorum.yml runs to authenticate the quorum server.  Once authentication has been verified, the file uses the command module to add the quorum device to the cluster like so:

```yaml
pcs quorum device add model net algorithm=ffsplit host={% for node in groups.quorum_node %}{{ hostvars[node].ha_cluster.node_name | quote }}{% endfor %}
```

### pcs-auth-pcs-0.10-quorum.yml

This tasks file executes when the quorum server shows as not authenticated by `pcs-auth-quorum.yml`.  After authorizing the quorum device, just like in `pcs-auth-quorum.yml`, we use the command module to add the quorum server to the cluster.

```yaml
---
- name: Pcs auth using pcs-0.10
  command:
    # Always auth all nodes to prevent possible corner cases with synchronizing
    # pcs auth tokens in the cluster when not all nodes are auth-ed.
    cmd: >
      pcs host auth -u hacluster --
      {% for node in groups.quorum_node %}
        {{ hostvars[node].ha_cluster.node_name | quote }}
        {% if hostvars[node].ha_cluster.pcs_address | default("") %}
          addr={{ hostvars[node].ha_cluster.pcs_address | quote }}
        {% endif %}
      {% endfor %}
    stdin: "{{ ha_cluster_hacluster_password }}"
  run_once: yes
  changed_when: yes

- name: Add quorum device to cluster
  command:
    cmd: >
      pcs quorum device add model net algorithm=ffsplit host={% for node in groups.quorum_node %}{{ hostvars[node].ha_cluster.node_name | quote }}{% endfor %}
```

### quorum-services.yml

This tasks file simply starts and enables pcsd.  Also, it checks the status of QNet daemon on the quorum device.  If the command `pcs qdevice status net` fails, the code starts the QNet daemon on the quorum device.

## Example Playbook

### Playbook developed during the May '23 Red Hat engagement
```yaml
---
- name: deploy quorum server
  hosts: quorum_node
  vars:
    ha_cluster_cluster_name: "{{ cluster_name }}"
    ha_cluster_hacluster_password: "{{ cluster_password }}"
    ha_cluster_cluster_present: false
    __ha_cluster_qdevice_in_use: yes
  vars_files: 
    - /home/ansible/ansible/secret.yml
  roles:
    - rhel-system-roles.ha_cluster_modified
  tags: deploy_quorum

- name: Set up shared filesystems requirements
  hosts: cluster_nodes
  vars_files: 
    - /home/ansible/ansible/secret.yml
  tasks:
  - name: Prep all cluster nodes for using shared storage
    include_role: 
      name: configure_storage
      tasks_from: prep_storage.yml 
  tags: prep_storage

- name: configure storage (select & validate LUNs)
  hosts: hosta.domain.com
  vars:
    ha_cluster_cluster_name: "{{ cluster_name }}"
    select_LUNs: yes 
    cluster_name: 
    LUN_list: []
    __150GB_LUN_list: []    # **Both** 150GB LUNs must be either dynamically defined or statically defined. 
    __500GB_LUN: ""   # i.e. dm-name-mpathd  note: Leave this blank for dynamic creation of variable
    __150GB_LUN_1: ""       # i.e. dm-name-mpathe  note: Leave this blank for dynamic creation of variable
    __150GB_LUN_2: ""       # i.e. dm-name-mpathf  note: Leave this blank for dynamic creation of variable
  roles:
  - { role: select_LUN, when: select_LUNs | bool }  
  tags: select_LUNs

- name: deploy cluster (mpath, lvmlockd, dlm resources)
  hosts: cluster_nodes
  vars:
    ha_cluster_cluster_name: "{{ cluster_name }}"
    ha_cluster_hacluster_password: "{{ cluster_password }}"
    __ha_cluster_qdevice_in_use: yes
    mpath_devices: /dev/mapper/mpathd,/dev/mapper/mpathe,/dev/mapper/mpathf
    mpath_device_pcmk: "hosta.domain.com:10;hostb.domain.com:11"
    nfs_server_monitor_interval: example_nfsserver-monitor-interval-60
    nfs_server_start_interval: example_nfsserver-start-interval-0s
    nfs_server_stop_interval: example_nfsserver-stop-interval-0s
    floating_ip: 10.1.2.3
  vars_files: 
    - /home/ansible/ansible/lvmlockd_dlm.yml
    - /home/ansible/ansible/secret.yml  
  roles:
    - rhel-system-roles.ha_cluster_modified
  tags: deploy_cluster

- name: configure storage (lvm & filesystems)
  hosts: cluster_nodes
  vars:
    ha_cluster_cluster_name: "{{ cluster_name }}"
    configure_storage: yes 
    cluster_name: 
  vars_files: 
    - /home/ansible/ansible/secret.yml 
  roles:
    - { role: configure_storage, when: configure_storage | bool }
  tags: configure_storage

- name: deploy cluster (lvm, filesystems)
  hosts: cluster_nodes
  vars:
    ha_cluster_cluster_name: "{{ cluster_name }}"
    ha_cluster_hacluster_password: "{{ cluster_password }}"
    __ha_cluster_qdevice_in_use: yes
    mpath_devices: /dev/mapper/mpathd,/dev/mapper/mpathe,/dev/mapper/mpathf
    mpath_device_pcmk: "hosta.domain.com:10;hostb.domain.com:11"
    nfs_server_monitor_interval: example_nfsserver-monitor-interval-60
    nfs_server_start_interval: example_nfsserver-start-interval-0s
    nfs_server_stop_interval: example_nfsserver-stop-interval-0s
    floating_ip: 10.1.2.3
  vars_files: 
    - /home/ansible/ansible/fs_resources.yml
    - /home/ansible/ansible/secret.yml

  roles:
    - rhel-system-roles.ha_cluster_modified
  tags: deploy_shared_filesystems
```

## License

MIT

## Author Information

Tomas Jelinek (cited for ha_cluster docs), Cory Beaver
