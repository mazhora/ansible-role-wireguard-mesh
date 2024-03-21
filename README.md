# Ansible Role: Wireguard mesh network

Installation and configuration of WireGuard on Ubuntu servers for building a mesh network.

What problem are we solving? Infrastructure is often represented by a large array of heterogeneous entities. This is especially true for mature infrastructures when you don't want to rely on a single cloud provider. This means you might lease VMs from various cloud providers (such as Gcore, MCS, AWS EC2, and so on), and you might also have bare metal servers in addition to virtual ones. In this scenario, there's often a need to establish an internal mesh network among all hosts. Since you're using resources from various providers, creating a single virtual switch is not feasible. This is particularly challenging when you're dealing with more than one provider. This is the exact issue that this Ansible role addresses. It creates a unified internal mesh network among all your devices.

The main requirements I set during its development were as follows:
- No dependencies. Apart from the servers themselves, no additional components should be needed to deploy the network.
- The ability to modify the network configuration after deployment. This includes various operational actions such as changing server addressing, adding new nodes to the network, and removing old ones.
- Full functionality of the role when run with limits. This might be considered a whim of mine. Personally, I find it daunting to roll out changes to the production environment across all hosts at once, so I believe it's important to have the ability to deploy changes gradually to a limited number of hosts.

## Requirements

Be careful. This ansible role patches the standard `wg-quick` script and `wg-quick@.service` from the wireguard package. If you already have any wireguard network on this host, exercise additional caution regarding the compatibility of these changes with your existing infrastructure.

## Role Variables

The variables you can use are detailed below, including their default settings (refer to defaults/main.yml):

### `wireguard_port`

- WireGuard port.
- Default value: `51820`

### `wireguard_path`

- Path to the directory where the configuration file and WireGuard host keys will be stored.
- Default value: `/etc/wireguard`

### `wireguard_network_name`

- Name of the WireGuard network interface for the mesh network.
- Default value: `wg0`

### `wireguard_group`

- The variable is used to change the default group name that should contain all hosts between which the mesh network is deployed.
- Default value: `wireguard`

### `wireguard_ip_address_var`

- The variable is used to change the default variable name that contains the internal address of the host in the mesh network.
- Default value: `wg_ip_address`

### `wireguard_supernets`

- List of subnets for which traffic will be routed into the mesh network.
- Default value: `[]`

## Dependencies

None.

## Quick start

It's necessary to place all hosts in a group named wireguard (you can change the default group name using wireguard_group). Each host must be assigned an IP address that will be used within the mesh network, assigned to the variable wg_ip_address (you can change the default variable name using wireguard_ip_address_var). The list wireguard_supernets should contain subnets for which traffic will be routed through the mesh network. Generally, this is the subnet that includes all addresses assigned to the hosts.

    - hosts: wireguard
      become: yes
      roles:
        - mazhora.wireguard-mesh

*Inside `inventory.yml`*:

    [all]
    host-1 wg_ip_address=10.128.0.1
    host-2 wg_ip_address=10.128.0.2
    host-3 wg_ip_address=10.128.0.3
    host-4 wg_ip_address=10.128.0.4
    
    [wireguard]
    host-1
    host-2
    host-3
    host-4


*Inside `group_vars/all.yml`*:

    wireguard_supernets:
      - "10.128.0.0/24"

## License

MIT / BSD

## Author Information

This Ansible role was developed in 2024 by [Eugene Mazhora](https://mazhora.ru/).

## To do

Plans for the future:

- Implement Continuous Integration (CI).
- Integrate molecule testing.
- Enhance performance within set boundaries.
- Add support for CentOS.
