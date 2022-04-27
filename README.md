# nss-shim

** work in progress - see Actions for latest artifacts **

Linux NSS module to create users on-demand when information is requested about them via services like ssh, id, and so on.

Forked from [osallou/nss-external](https://github.com/osallou/nss-external).

## Motivation

Even with custom PAM modules and AuthorizedKeysCommand configured, OpenSSH requires users to exist already (via NSS) before PAM is used for authentication. Thus, users cannot be created dynamically on SSH session setup with PAM alone - an NSS shim is also required.

## Details

- Configuration is read from nss_external.conf
- As no password is set, user auth must be performed by PAM or a service like SSH after NSS

## Flow

If a username is validated:

1. A corresponding local user account is created with no password (not empty, but set to block password login entirely)
2. The user is assigned to the specified group
3. A home directory is created.

## Build

Note: use Go 1.12 - https://github.com/protosam/go-libnss/issues/7

    CGO_CFLAGS="-g -O2 -D __LIB_NSS_NAME=external" go build --buildmode=c-shared -o libnss_external.so.2 nss-external.go

## Config

Create `/etc/nss_external.conf`:

    users: []
    nss:
    prefix:
        - "@elixir-europe.org"
    groupid: 1000
    minuid: 10000
    bash: "/bin/bash"
    home: "/home/external/%s"

Update `/etc/nsswitch.conf`:

    passwd:         compat external
    group:          compat
    shadow:         compat external
