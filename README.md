# Let's Encrypt MyDNSHost Service

This container uses the [MyDNSHost](https://www.mydnshost.co.uk/) API with
[dehydrated](https://github.com/lukas2511/dehydrated) to
automatically obtain SSL certs from [Let's Encrypt](https://letsencrypt.org/).

Multiple domains, as well as SANs, are supported. Certificates will be
renewed automatically, and obtained automatically as soon as new domains
are added.

## Usage

### Accepting Let's Encrypt's terms

In order to issue certificates with Let's Encrypt, you must agree to the
Let's Encrypt terms of service. You can do this by running the command
`/dehydrated --register --accept-terms` from within the container.

For ease of automation, you can define the `ACCEPT_CA_TERMS` env var
(with any non-empty value) to automatically accept the terms. Be warned
that doing so will automatically accept any future changes to the terms
of service.

### Defining domains

The container defines one volume at `/letsencrypt`, and expects there to be
a list of domains in `/letsencrypt/domains.txt`. Certificates are output to
`/letsencrypt/certs/{domain}`.

domains.txt should contain one line per certificate. If you want alternate
names on the cert, these should be listed after the primary domain. e.g.

```
example.com www.example.com
admin.example.com
```

This will request two certificates: one for example.com with a SAN of
www.example.com, and a separate one for admin.example.com.

The container uses inotify to monitor the domains.txt file for changes,
so you can update it while the container is running and changes will be
automatically applied.

### Authorisation

To verify that you own the domain, a TXT record needs to be automatically
created for it. In order to create these records using the MyDNSHost API,
you must supply an API key.

The API client expects the username and API key as environment variables:

```
docker run ... \
  -e "MYDNSHOST_AUTH_USER=email@address.com" \
  -e "MYDNSHOST_AUTH_KEY=api-key-here"
```

### Other configuration

For testing purposes, you can set the `STAGING` environment variable to
a non-empty value. This will use the Let's Encrypt staging server, which
has much more relaxed limits.

You should pass in a contact e-mail address by setting the `EMAIL` env var.
This is passed on to Let's Encrypt, and may be used for important service
announcements.

### Running

Here's a full worked example:

```bash
# The directory we'll use to store the domain list and certificates.
# You could use a docker volume instead.
mkdir /tmp/letsencrypt
echo "domain.com www.domain.com" > /tmp/letsencrypt/domains.txt

docker run -d --restart=always \
  -e "EMAIL=admin@domain.com" \
  -e "STAGING=true" \
  -e "MYDNSHOST_AUTH_USER=email@address.com" \
  -e "MYDNSHOST_AUTH_KEY=api-key-here"
  -v /tmp/letsencrypt:/letsencrypt \
  csmith/letsencrypt-mydnshost:latest
```
