ansible-role-simp_le
====================
Install [simp_le](https://github.com/kuba/simp_le.git), generate certificates
and renew them automatically on Debian/Ubuntu servers.

Renewal happens every month via a cron job run by the Ansible remote user.

See the role on Ansible Galaxy: [L-P.simp_le](https://galaxy.ansible.com/detail#/role/6627)

## Required variables
A list of virtual hosts for which we'll generate certificates:
```yaml
simp_le_vhosts:
  - domains: ["www.example.com", "example.com"]
    root: "/path/to/challenges" # accessible via HTTP
    output: "/path/to/output/dir" # where to write the certificates
```

An email address LetsEncrypt will use to identify you and send renewal notices:
```yaml
simp_le_email: "your.email@example.com"
```

See `defaults/main.yml` for more configuration.

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

While most of the operations are done without `sudo`, it is still used to
create the various directories with the proper permissions and owners.
