# Certbot Ansible role

Author: Brad House<br/>
License: MIT<br/>
Original Repository: https://github.com/bradh352/ansible-role-service-certbot

## Overview

Role to make it easier to generate TLS certificates via certbot using DNS-01
challenge via the API endpoint of the hosting DNS server and LetsEncrypt. Often
multiple services in an environment need to generate certificates but differ
in slight ways (services to be restarted, proxies, etc).

## First Run Notes

Likely this role should be run *prior* to the service that needs this role.  That
adds a complication that maybe the certificate destination does not yet exist
or perhaps the service adds its own user so ownership cannot be set yet.

To work around these issues, the deployment script will auto-create the
specified destination if it does not exist, and will ignore ownership change
errors.

In the subsequent service role that needs the certificates, it is therefore
recommended to perform the necessary chown operations on the certificates
and/or directories.

## Variables

- `certbot_email`: Required. Used for certbot to specify email to provider.
- `certbot_provider`: Required. DNS provider in use for performing the DNS-01
  challenge.  Valid values are currently: `godaddy`, `cloudflare`
- `certbot_apikey`: Required. API Key for the DNS provider to be able to create
  a TXT record for `_acme-challenge.{{ inventory_hostname }}`.  This API should
  be restricted to exactly that access and nothing more.  Use `Key:Secret` for
  Godaddy keys. For GoDaddy see some information here:
  https://community.letsencrypt.org/t/godaddy-dns-and-certbot-dns-challenge-scripts/210189
- `certbot_proxy`: Optional.  If a proxy is needed to connect to the DNS
  provider and letsencrypt specify it.
  E.g. `https://secureproxy.testenv.bradhouse.dev:8080`
- `certbot_certs`: List of certificates to generate with various settings.
  - `domain`: Required. Primary fully qualified domain name of certificate.
  - `aliases`: Optional. List of domain aliases to be added as SubjectAltNames
    in the certificate.
  - `path`: Requied. Output directory.  Will write the files as:
    `{{ domain }}.key` (privkey), `{{ domain }}.crt` (cert),
    `{{ domain }}.cabundle` (chain), and `{{ domain }}.fullcrt` (fullchain).
    If the `domain` starts with `*.` that will be stripped from the output
    filename.
  - `owner`: Required. Owner of the output files.  The output files will be
    written with `600` permissions so this is necessary to get correct.
  - `service`: Optional.  If a service name needs to be restarted when a
    certificate is updated, specify the name here.
  - `command`: Optional. To run a custom command on deployment, specify that
    command here.
  - `apikey`: Optional.  If a different APIKey is needed for this domain it will
    be used as an override.
