# Pritunl

An Ansible role that installs, configures and manages Pritunl for EL 8.
A requirement for this role is that MongoDB is installed on the host(s).

[This role includes a complete MongoDB](https://github.com/csuka/ansible_role_mongodb) installation for EL 8.

When using Pritunl Enterprise, automatic failover is enabled.
In that case, the following design can be used for a cluster:

```yaml
             host1
          +---------+
          | Pritunl |
     +--->+ MongoDB +<----+
     |    +---------+     |
     |                    |
     |                    |
     |                    |
     |                    |
     v                    v
+---------+          +---------+
| Pritunl |          | Pritunl |
| MongoDB |<-------->| MongoDB |
+---------+          +---------+
   host2                host3
```

## Good2know

 * The /etc/security/limits.conf is by default replaced with a template, to [prevent issues on high load](https://docs.pritunl.com/docs/configuration-5)
 * The config file, /etc/pritunl.conf, is not edited in any way, because all configuration of Pritunl is placed in MongoDB

The epel repo is set by default, because the openvpn package is required.
Also, the pritunl repo itself is placed, because the pritunl package is required. The GPG keys and whatnot are set as [described in the docs](https://docs.pritunl.com/docs/repo).

```yaml
pritunl_epel: true  # to set the epel repo, for the openvpn package
pritunl_repo: true  # to set the pritunl repo
```

By default, a limits file is set in `/etc/security/limits.conf`. Do not place the limits file by setting:

```yaml
pritunl_limits: False
```

The following variables can be changed, [in case of load-balancing](https://docs.pritunl.com/docs/load-balancing):

```yaml
server_port: "app.server_port = 443"
redirect_server: "app.redirect_server = false"
reverse_proxy: "app.reverse_proxy = true"
server_ssl: "app.server_ssl = false"
```

## Installation

After installation, visit the web-gui, provide the [admin credentials and set the Mongo connection string](https://docs.pritunl.com/docs/configuration-5).

## Update

Update with:
```yaml
pritunl_state: latest
```

## Limitations

This role **does not**

  * Manage users
  * Manage organizations / servers
  * Manage routes
  * Manage Pritunl settings
  * etc..

## Example playbook

```yaml
---
- hosts:
    - host-1
    - host-2
    - host-3
  become: True
  vars:
    pritunl_state: present
    pritunl_limits: true
    pritunl_server_port: "app.server_port = 443"
    pritunl_redirect_server: "app.redirect_server = false"
    pritunl_reverse_proxy: "app.reverse_proxy = true"
    pritunl_server_ssl: "app.server_ssl = false"
  roles:
    # - mongodb
    - pritunl
```
