# ansible-role-apt-cacher-ng

Simply installs and start apt-cacher-ng on boot. Get more informations about apt-cacher-ng at https://www.unix-ag.uni-kl.de/~bloch/acng/

## Requirements
Ubuntu or Debian

## Role Variables
* `apt_cacher_ng_port: 3142`
* `apt_cacher_ng_cache_dir: /var/cache/apt-cacher-ng`
* `apt_cacher_ng_setup_ufw: False` Add a ufw rule to allow apt-cacher-ng
* `apt_cacher_ng_expiration_start_tradeoff: 500m` Size of local cache, expiration run is suppressed, until limited is surpassed
* `apt_cacher_ng_experition_threshold: 4` Days before purging
* `apt_cacher_ng_passthrough_pattern:` Not set by default

## Dependencies
None.

## Example Playbook

```yaml
- hosts: servers
  remote_user: root
  roles:
   - { role: postcert.apt_cacher_ng }
```

## Client configuration
### with ansible
Set apt_proxy as a host var

	[host:vars]
	apt_proxy=http://apt.example.com:3142/

**For the whole system:**

```yaml
- name: Set up apt proxy
  template: src=templates/apt_proxy.conf dest=/etc/apt/apt.conf.d/01proxy owner=root group=root mode=0644
    when: ansible_os_family == "Debian" and apt_proxy is defined
```

templates/apt_proxy.conf:

	# {{ ansible_managed }}
	Acquire::http { Proxy "{{ apt_proxy }}"; };
	Acquire::https { Proxy "https://"; };

**Only for one task:**

```yaml
- apt: name=ufw state=installed
  environment:
    http_proxy: "{{ apt_proxy }}"
```

### without ansible
Replace server IP/FQDN!

	$ echo 'Acquire::http { Proxy "http://apt.example.com:3142"; };' > /etc/apt/apt.conf.d/01proxy

## Import localhost cache

	$ echo 'Acquire::http { Proxy "http://localhost:3142"; };' > /etc/apt/apt.conf.d/01proxy
	$ apt-get update
	$ apt-get autoclean
	$ mkdir -p /var/cache/apt-cacher-ng/_import
	$ ln -s /var/cache/apt /var/cache/apt-cacher-ng/_import/apt
	$ wget "http://localhost:3142/acng-report.html?abortOnErrors=aOe&doImport=Start+Import&calcSize=cs&asNeeded=an#bottom"

After the import has finished, you can remove the symlink with:

	$ rm /var/cache/apt-cacher-ng/_import/apt

## License

MIT

## Author Information

postcert <postcert@gmail.com>
elnappo <elnappo@nerdpol.io>
