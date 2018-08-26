# ansible-role-hardened-ubuntu

Ansible role that applies multiple layers of system hardening. 
A suitable starting point for a wide variety of applications.

A prettier version of this documentation is located at 
https://bedrocksolutions.github.io/ansible-role-hardened-ubuntu

## Features

### Hardening

The following steps are performed to secure the operating system:

* Hardens the SSH client and server using 
[dev-sec.ssh-hardening](https://github.com/dev-sec/ansible-ssh-hardening)
* Hardens the entire operating system using 
[dev-sec.os-hardening](https://github.com/dev-sec/ansible-os-hardening)
* Installs Fail2Ban and configures a jail for SSH(tcp:22)
* Installs Unattended Upgrades to keep security patches up-to-date

### Convenience:

The following convenience features are also included:

* Updates the Apt cache and upgrades all packages
* Authorize one or more SSH keys (optional)
* Configure Fail2Ban to send Slack/Discord notifications upon
IP ban
* Install the Stackdriver monitoring agents (optional)
* Configure a device for use as swap space (optional)

## Installation

To use `bedrock.hardened-ubuntu` in another role, create the file 
`<role_root>/meta/main.yml` with the following structure:

```yaml
dependencies:
  - name: bedrock.hardened-ubuntu
    scm: git
    src: https://github.com/BedrockSolutions/ansible-role-hardened-ubuntu.git
    vars:
      hardened_ubuntu_enabled: no
    version: master
```

If you just want to use this role in a playbook or task file, then
add an entry to the `requirements.yml` file:

```yaml
  - name: bedrock.hardened-ubuntu
    scm: git
    src: https://github.com/BedrockSolutions/ansible-role-hardened-ubuntu.git
    version: master
```
>__Note:__ In both examples, the `version` field can be a branch, tag, or commit hash.

The role can now be imported or included as `bedrock.hardened-ubuntu`.

## Role Variable Scoping

All role variables are passed as a single `dict` 
named `hardened_ubuntu`:

For example:

```yaml
- import_role
    name: bedrock.hardened-ubuntu
  vars:
    hardened_ubuntu: # <-- All vars are scoped under this dict!
      os_swap_device: /dev/sdb
      ssh_agent_forwarding_enabled: yes
      
```

## Role Variables

* __`fail2ban_ignored_ips`:__ optional list of IP networks that 
are excluded from fail2ban monitoring. 

    * type: list\<string\>

> Note: During the setup phase, it can be useful to exclude your 
own private IP address space.

* __`fail2ban_slack_webhook_url`:__ optional Slack or Discord
webhook URL to call when a host is baned.
    * type: string
    * format: uri

* __`os_kernel_ip_forwarding_enabled`:__ should IP forwarding be enabled
in the kernel. 

    * type: boolean
    * default: `false`

> Note: Certain subsystems, such as Docker, require kernel IP
forwarding to be enabled.
      
* __`os_swap_device`:__ optional device to use for swap space
    * type: string

* __`ssh_agent_forwarding_enabled`:__ Enable SSH Agent forwarding
in the SSH daemon

    * type: boolean
    * default: `false`

* __`ssh_allowed_networks`:__ list of IP networks that are allowed
to connect to the SSH daemon. At least one entry is required.
    * type: list\<string\>
    * default: `['0.0.0.0/0']`
    * minLength: 1

* __`ssh_allowed_users`:__ A list of usernames allowed to connect 
to the SSH daemon

    * type: list\<string\>
    * default: `['ubuntu']`
    * minLength: 1

* __`ssh_authorized_keys`:__ An optional list of SSH public keys 
allowed to connect to the SSH daemon

    * type: list\<string\>

* __`ssh_exclusive_keys_enabled`:__ If the `ssh_authorized_keys`
variable is set, removes additional keys from the 
`.ssh/authorized_Keys` file

    * type: boolean
    * default: `false`

* __`ssh_sftp_enabled`:__ Enable SFTP over SSH connections. If
disabled, use SCP instead

    * type: boolean
    * default: `true`
      
* __`ssh_tun_device_forwarding_enabled`:__ Enable SSH TUN device
forwarding in the SSH daemon

    * type: boolean
    * default: `false`
      
* __`stackdriver_enabled`:__ installs the Stackdriver agents and
ingests the following logs: fail2ban, unattended_upgrades 

    * type: boolean
    * default: `false`
