# Ansible Collection - puppeteers.plausible

## Introduction

This collection contains roles for managing [Plausible Community
Edition](https://github.com/plausible/community-edition), a self-hosted, open
source analytics software. Currently only RHEL 10 and derivatives are
tested and supported.

## Roles

### puppeteers.plausible.ce

This role sets up Plausible CE so that it can run from Podman Compose. This
role affects a bunch of things:

* Creates a dedicated, unprivileged system user for running Plausible
* Sets up an nginx reverse proxy running on the host in front of Plausible compose stack
* Configures Plausible CE
    * Basic settings
    * SMTP settings (required as user invites depend on working SMTP)
* Optional: firewalld configuration
    * Port forwards host port 443/tcp to nginx running on port 8443/tcp (for Plausible traffic)
    * Port forwards host port 80/tcp to nginx running on port 8080/tcp (for Letsencrypt traffic)
* Optional: sets up Letsencrypt certificates with the HTTP-01 challenge
    * Creates initial Letsencrypt certs
    * Adds cronjob for renewing Letsencrypt certificates weekly
    * Automatically synchronizes Letsencrypt certs from plausible user's home to /etc/letsencrypt and reloads nginx if needed

This role does not, at the moment, launch the plausible-ce compose stack. It
would be better to run it as Podman Quadlets (i.e. systemd services), but for
now you can just "podman-compose up" it manually in a tmux session.

The following ports are used:

* 8000/tcp (plausible)
* 8080/tcp
    * nginx for forwarding traffic to 8443
    * temporarily used by standalone certbot during weekly certbot renewals
* 8443/tcp (nginx for normal web traffic
* 80/tcp (forwarded to 8080/tcp when firewalld management is enabled)
* 443/tcp (forwarded to 8443/tcp when firewalld management is enabled)

Nginx configuration assumes the SSL certificates to be present under
*/etc/letsencrypt/live/my_site_name*, even if you handle certificates yourself.
If this is a blocker, please refactor and issue a PR.

Example usage:

    puppeteers_plausible_ce_manage_firewalld: true
    puppeteers_plausible_ce_manage_letsencrypt: true
    puppeteers_plausible_ce_ssh_authorized_key: "<ssh-authorized-key-for-plausible-system-user>
    puppeteers_plausible_ce_letsencrypt_email: letsencrypt@example.com
    puppeteers_plausible_ce_server_name: plausible.example.com
    puppeteers_plausible_ce_base_url: https://plausible.example.com
    puppeteers_plausible_ce_secret_key_base: <plausible secret key - check plausible documentation>
    puppeteers_plausible_ce_http_port: 8000
    puppeteers_plausible_ce_mailer_email: plausible@domain.com
    puppeteers_plausible_ce_mailer_name: Plausible
    puppeteers_plausible_ce_smtp_host_addr: smtp.example.com
    puppeteers_plausible_ce_smtp_host_port: 587
    puppeteers_plausible_ce_smtp_user_name: "plausible@example.com"
    puppeteers_plausible_ce_smtp_user_pwd: <SMTP user password>
    puppeteers_plausible_ce_smtp_host_ssl_enabled: false

See [roles/ce/defaults/main.yml](defaults/main.yml) and
[roles/ce/tasks/main.yml](tasks/main.yml) for details.
