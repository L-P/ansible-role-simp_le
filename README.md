ansible-role-simp_le
====================
Install [simp_le](https://github.com/kuba/simp_le.git), generate certificates
and renew them automatically on Debian/Ubuntu servers.

Renewal happens every month via a cron job run by the Ansible remote user.

See the role on Ansible Galaxy: [L-P.simp_le](https://galaxy.ansible.com/detail#/role/6627)

## Variables
### Variables with defaults
```yaml
# Variables with defaults
simp_le_repo: # simp_le repository
simp_le_version: # commit/version to clone
simp_le_dest: # where to clone, by default in ~/.cache
```

### Required variables
```yaml
simp_le_vhosts: []
```

A list of virtual hosts for which we'll generate certificates. eg.:
```yaml
simp_le_vhosts:
  - vhost: "example.com"
    root: "/path/to/challenges" # accessible via HTTP
    output: "/path/to/output/dir" # where to write the certificates
```

## Server configuration
Your server needs to serve the challenge files over HTTP, here is an example
configuration you can use for _nginx_ that will redirect every HTTP request to
HTTPS except for the challenges:

```
location /.well-known/acme-challenge/ {
    alias /var/www/challenges/.well-known/acme-challenge/;
    try_files $uri @forward_https;
}
location @forward_https {
    return 301 https://example.com$request_uri;
}
location / {
    return 301 https://example.com$request_uri;
}
```

## Example playbook
```
- hosts: all
  roles:
    - {role: "L-P.simp_le", sudo: no}
```
