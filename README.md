# Ansible Role: Certbot with Cloudflare DNS Challenge

Installs certbot via either pip or snap and uses Cloudflare DNS Challenge for wildcard cert generation

## Requirements

- Cloudflare DNS Zone
- Cloudflare API Token [Certbot Cloudflare DNS Docs](https://certbot-dns-cloudflare.readthedocs.io/en/stable/)
- Wildcard domain setup (for wildcard certs) [*.domain.com or *.subdomain.domain.com]

## Role Variables

***Define these variables in group_vars or host_vars***

Set API for token for Cloudflare

    certbot_cloudflare_api_token: ''

***Secrets should be stored in Ansible Vault or some other secrets management platform, do not commit secrets plain text!***

Certificates to generate which can include wildcards or not

```yaml
certbot_certs:
  - email: {{certbot_cloudflare_email}}
    domains:
      - example.com
      - *.example.com
  - email: {{certbot_cloudflare_email}}
    domains:
      - example2.com
      - *.example2.com
```

Method of how to install certbot, defaults to snap. Valid options are snap, package or pip

    certbot_install_method: pip

Enable certbot's HTTP Strict Transport Security (HSTS)

    certbot_hsts: true

Use Let's Encrypt staging server, defaults to no.

    certbot_testmode: true

The create command which instructs the role to use DNS-01 Challenge. This is defined in the role defaults should not needed to be changed.

```yaml
certbot_create_command:
  - "{{ certbot_script }}"
  - certonly
  - "{{ '--hsts' if certbot_hsts else omit }}"
  - "{{ '--test-cert' if certbot_testmode else omit }}"
  - --dns-cloudflare
  - "--dns-cloudflare-credentials={{ certbot_config_dir }}/cloudflare.ini"
  - --dns-cloudflare-propagation-seconds
  - "60"
  - --noninteractive
  - --agree-tos
  - "--email={{ cert_item.email | default(certbot_admin_email) }}"
  - "-d"
  - "{{ cert_item.domains | join(',') }}"
  - "{{ '--pre-hook=' ~ certbot_config_dir ~ '/renewal-hooks/pre/stop_services'
       if certbot_create_stop_services else omit }}"
  - "{{ '--post-hook=' ~ certbot_config_dir ~ '/renewal-hooks/post/start_services'
       if certbot_create_stop_services else omit }}"
```

### Author Information

This role was created by [glennbrown](http://github.com/glennbrown) and is based on [Jeff Geerlings](https://github.com/geerlingguy/ansible-role-certbot/tree/master) role and this role by [Michael Porter](https://github.com/michaelpporter/ansible-role-certbot-cloudflare)