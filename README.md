chrony-server
=============

This Ansible role installs and configures chrony to operate as an NTP server on
Fedora and RHEL/CentOS (with EPEL) systems. It can configure iptables to accept
requests from NTP clients and disable connection tracking for NTP packets in
order to not waste resources with a large number of clients (e.g. on a public
NTP server). It can also install and configure collectd with plugins to collect
statistics (chrony's tracking and incoming/outgoing NTP packet rates) and
generate web pages with graphs in a specified directory.

Role Variables
--------------

The variables that can be passed to this role and their default values are as
follows:

```
# Configuration added to a chrony.conf template
chrony_configuration:

# Command-line options for chronyd
chronyd_options: -r

# Flag enabling use of NTP servers obtained from DHCP
dhcp_ntp_servers: no

# Flag enabling the ntp-firewall service, which configures iptables to accept
# NTP requests, count the requests and responses, and disable connection
# tracking for NTP packets
ntp_firewall: yes

# Flag enabling collectd and a cron job to generate web pages with graphs
ntp_stats: no

# Hostname for the statistics
ntp_stats_hostname: {{ ansible_hostname }}

# Directory for generated web pages
ntp_stats_www_dir: /var/www/html/ntp-stats

# Location of a helper script
helper_path: /usr/local/sbin/chrony-server-helper
```

Example Playbook
----------------

This is a minimal example which configures an NTP server synchronized to
public servers from the http://www.pool.ntp.org project.

```
- hosts: ntp-servers
  vars:
    chrony_configuration: |
      pool 2.pool.ntp.org iburst
  roles:
    - chrony-server
```

The second example uses a different chrony configuration and enables the
statistics. It also installs and enables httpd to serve the generated web
pages. (A web server role might be preferred for that.)

```
- hosts: ntp-servers
  vars:
    chrony_configuration: |
      server foo.example.net iburst maxpoll 6
      server bar.example.net iburst maxpoll 6
      server baz.example.net iburst maxpoll 6
      server qux.example.net iburst maxpoll 6
      clientloglimit 100000000
      ratelimit interval 1 burst 8
      leapsectz right/UTC
    ntp_stats: yes
  roles:
    - chrony-server

  tasks:
    - name: Install httpd
      package: name=httpd state=present

    - name: Enable httpd
      service: name=httpd state=started enabled=yes
```
