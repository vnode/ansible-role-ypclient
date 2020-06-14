# Ansible role: ypclient
This role installs and configures the YP/NIS client that is part of OpenBSD and other BSD* operating systems.
Currently, this role only supports OpenBSD, with the intent on adding FreeBSD and NetBSD in a near-future update.

Where applicable, this role refers to system documentation, e.g. the [`yp(8)`](https://man.openbsd.org/yp) man page.


# Requirements

## Operation
No external roles and/or modules are required to use this role.

## Testing & Development
For testing and development, this role depends on the following roles and external tools:
- Role vnode.ypserver
- Vagrant (supporting either the VirtualBox or VMWare provider)
- VagrantCloud (specifically, the `generic/openbsd6` box)


# Role Variables
Available variables are listed below, including their default values (see `defaults/main.yml`).
All of these *should* be implemented. If you find they are not, please [open an issue](https://github.com/vnode/ansible-role-ypclient/issues) in the GitHub repository.


## Required variables
The following variables need to be set when using the role.

```yaml
ypclient_domain: ""
```
*Required*, must have a valid NIS domainname. This is the name of the NIS domain that you intend to configure.

```yaml
ypclient_servers: []
```
*Required*, must list the set of NIS servers for the domain.

```yaml
ypclient_serverinfo: {}
```
*Required*, but may be empty if the NIS servers for the domain can be found in DNS or `/etc/hosts`.

If non-empty, this dictionary lists IPv4 and/or IPv6 addresses for the domain's servers. The role will then populate `/etc/hosts` with the required lines. If servers cannot be reached or resolved, the NIS code will hang. See [`yp(8)`](https://man.openbsd.org/yp) for more details.

The example below lists addresses for the `master` and `slave` servers in a dual-stack network.

```yaml
ypserver_serverinfo:
  master:
    - "192.0.2.1"
    - "2001:db8::111:1"
  slave:
    - "192.0.2.2"
    - "2001:db8::111:2"
```


## Optional variables

```yaml
ypclient_usedns: true
```
Specifies that the YP/NIS maps can use DNS for lookups of host names. Recommended to leave at `true `. When set to `false`, make sure that `ypclient_serverinfo` and/or `ypclient_set_hosts` are set correctly so that your NIS servers can be resolved.

```yaml
ypclient_lookup_maps:
  - name: 'passwd'
    file: '/etc/master.passwd'
    pattern: '+:*::::::::'
    validate: 'pwd_mkdb -c %s'
    notify: "regen master.passwd"
    mode: '0600'
    owner: 'root'
    group: 'wheel'
  - name: 'group'
    file: '/etc/group'
    pattern: '+:*::'
    mode: '0644'
    owner: 'root'
    group: 'wheel'
```
This list contains which lookup maps you want to use in the domain and which files need to be edited, as well as the pattern. Specifying the validation and handler is optional. Naturally, the maps must exist on the NIS servers as well. For other supported maps, please see [`Makefile.yp(8)`](https://man.openbsd.org/Makefile.yp).


## Variables to support multiple YP/NIS domains on a client
This variable is intended to allow the hosting of multiple NIS domains on a server. This was not the originally intended use case, so if you find issues, please report this as an [issue on GitHub](https://github.com/vnode/ansible-role-ypserver/issues).

```yaml
ypclient_set_domainname: true
```
Recommended to leave at `true` for the domain that is intended as your 'main' (default) domain name. Must be set to `false` if you want to keep another domain as default domain.


## Additional settings
These variables are not required for role invocations and their defaults should be fine.

```yaml
ypclient_set_hosts: false
```

If set to `true`, the role adds the IP information for the NIS servers to the `/etc/hosts` file. This is typically useful when the domain does not use DNS lookups (`ypclient_usedns` set to `false`). Note that this does require IP information for *each* NIS server in the `ypclient_serverinfo` variable.


## Internal variables
These variables are used in the role internally and are not intended for user modification. Change these at your own peril. Typically, they correspond to hard-coded values on the underlying operating system.


# Dependencies
None.


# Example Playbook
Below is an example to create a simple YP//NIS domain client that connects to the hosts in the `ypservers` group. The domain is called `legacy`.

```yaml
---
- hosts: ypclients
  roles:
    - role: vnode.ypclient
      vars:
        ypclient_ypdomain: legacy
        ypclient_servers: "{{ groups['ypservers'] }}"
```


# License
MIT


# Author Information
This role was created in 2020 by [Rogier Krieger](https://vnode.net/).
