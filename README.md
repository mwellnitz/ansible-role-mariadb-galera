Mariadb/Galera
==============

Install mariadb with galera support on Ubuntu LTS or CentOS/RHEL 7.x

Requirements
------------

On CentOS/RedHat and Ubuntu/Debian the mariadb packages from mariadb.org are used. (I've tried with the software-collections version for RHEL 7, but couldn't get them to build a galera cluster)

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: galera-db-servers
      roles:
        - common
        - mariadb-galera
      vars:
            - galera_wsrep_cluster_name: galera-cluster-1
            - galera_wsrep_cluster_address: "gcomm://{{ foo_IP }},{{ bar_IP }},{{ baz_IP }}"
            - mariadb_conf_dir: "/etc/mysql/conf.d"
            - galera_cluster_nodes: |
                    - foo.domain
                    - bar.domain
                    - baz.domain
            - galera_current_node_name: "{{ ansible_hostname }}.{{ vlan_domain | join }}" # optional makes sense only if ansible_hostname is not in galera_cluster_nodes
            - galera_wsrep_node_address: "{{ node_IP }}" # optional makes sense only if it's not ansible_default_ipv4.address
            - mariadb_install_xtrabackup: true  # optional
            - mariadb_install_backupninja: true # optional
            - ufw_enabled: false                # optional
            - mysql_mount: "/data"              # optional , !DO IT ONLY IF /data IS A MOUNTPOINT! -- /var/lib/mysql will be deleted, /data/mysql will be created and symlink'ed to /var/lib/mysql  

If you have an extra location for your data (eg. separate bcache device, NVMe, etc.) make use of parameter `mysql_mount`. The MariaDB/mysql standard location for data files will be symlinked to your extra location before MariaDB will beinstalled.

You can overwrite the most important parameters and tuning options. Just have a look at the `defaults/main.yml` file. All MariaDB options are set via `tasks/configuration.yml`. If you want to overwrite options which are nonexistant in `defaults/main.yml` configure `mysql_custom_vars`. But be warned: It's not specified in which order MariaDB will read the files. Here we took the safeties way: staring with numbers ;-)

The first node in `galera_cluster_nodes` is used as initial node. When starting mysql on this node fails, a second task tries to run `galera_new_cluster` to bootstrap the cluster. Therefore it's useful to run ansible on the first node first and then on the other nodes. A database shutdown will only be performed if quorum won't be affected. That's a feature, not a bug ;-) You will love it if you ever try to run the playbook on a degraded cluster. But keep in mind there is a little gap between reading the status variable and shutdown the service. If you use a tool like `multissh` or `pssh` you can crash your cluster immediately. Don't say you haven't be warned!

License
-------

MIT

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
