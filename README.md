ansible-role-simp_le
====================
Install [simp_le](https://github.com/kuba/simp_le.git), generate certificates
and renew them automatically on Debian/Ubuntu servers.

Renewal will be attempted daily via a cron job run by the Ansible remote user.

See the role on Ansible Galaxy: [L-P.simp_le](https://galaxy.ansible.com/detail#/role/6627)

**Warning**: as of 2016-09-15, simp_le does not seem to be maintained
and I don't use this role anymore.
I started using [acmetool](https://github.com/L-P/ansible-role-acmetool).
It has no dependencies, it is easier to install, you don't need to build
anything, and it runs on ARM ports where many packages are missing.

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

There are three optional keys you can set on hosts:

- `user` and `group` to specifiy who will own the keys, challenges and their parent directory
  The owner defaults to `www-data:www-data`.
- `extra_args` to pass extra arguments to simp_le, this can be used to use the
  LetsEncrypt staging server or to tell simp_le to reuse the key pair when
  renewing the certificate. This is useful if you are using TLSA records, you
  can then use Selector type 1 (SubjectPublicKeyInfo) and your TLSA record will
  not need changing when the certificate is renewed.
- `update_action` a command to be run when a certificate is renewed,
   e.g. `systemctl restart apache2`

Example:
```yaml
simp_le_vhosts:
  - domains: ["smtp.example.com", "mail.example.com"]
    root: "/path/to/challenges"
    output: "/path/to/output/dir"
    user: "Debian-exim"
    group: "Debian-exim"
    extra_args: "--reuse_key --server https://acme-staging.api.letsencrypt.org/directory"
    update_action: "/bin/systemctl restart exim4"
```

See `defaults/main.yml` for more configuration.

## Server configuration
Your server needs to serve the challenge files over HTTP, here is an example
configuration you can use for _nginx_ that will redirect every HTTP request to
HTTPS except for the challenges:

```nginx
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
```yaml
- hosts: all
  roles:
    - {role: "L-P.simp_le", become: no}
```

While most of the operations are done without `sudo`, it is still used to
create the various directories with the proper permissions and owners.
