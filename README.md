# acme-dns-certbot-joohoi

An example [Certbot](https://certbot.eff.org) client hook for [acme-dns](https://github.com/joohoi/acme-dns). 

This authentication hook automatically registers acme-dns accounts and prompts the user to manually add the CNAME records to their main DNS zone on initial run. Subsequent automatic renewals by Certbot cron job / systemd timer run in the background non-interactively.

Requires Certbot >= 0.10, Python 2.7 and requests library.

## Installation

1) Install Certbot using instructions at [https://certbot.eff.org](https://certbot.eff.org)

2) Make sure you have the [python-requests](http://docs.python-requests.org/en/master/) library installed.

3) Download the authentication hook script and make it executable:
```
$ curl -o /etc/letsencrypt/acme-dns-auth.py https://raw.githubusercontent.com/joohoi/acme-dns-certbot-joohoi/master/acme-dns-auth.py
$ chmod 0700 /etc/letsencrypt/acme-dns-auth.py
```

4) Configure `acme-dns-auth` to point to your acme-dns instance by setting environment variables. The only values that you must set are `ACMEDNSAUTH_URL` (which points to your `acme-dns` instance) and `ACMEDNSAUTH_ENV_VERSION` (which is used to ensure compatibility across upgrades).

The easiest way to do this is by generating setup template using this command:

````
$ python /etc/letsencrypt/acme-dns-auth.py --setup
```

That will print out a template that you can customize

```
# ---------- CUSTOMIZE THE BELOW ----------
    
# required settings
#
# URL to acme-dns instance
export ACMEDNSAUTH_URL="https://acme-dns.example.com"
# used to maintain compatibility across future versions
export ACMEDNSAUTH_ENV_VERSION="1"

# optional settings
#
# Path for acme-dns credential storage
export ACMEDNSAUTH_STORAGE_PATH="/etc/letsencrypt/acmedns.json"
# Whitelist for address ranges to allow the updates from
# this must be a list encoded as a json string
# Example: `export ACMEDNSAUTH_ALLOW_FROM='["192.168.10.0/24", "::1/128"]'`
export ACMEDNSAUTH_ALLOW_FROM='[]'
# Force re-registration. Overwrites the already existing acme-dns accounts.
export ACMEDNSAUTH_FORCE_REGISTER="False"

# ----------                     ----------
```

## Usage

On initial run:
```
$ export ACMEDNSAUTH_URL="https://acme-dns.example.com"
$ export ACMEDNSAUTH_ENV_VERSION="1"
$ certbot certonly --manual --manual-auth-hook /etc/letsencrypt/acme-dns-auth.py \
   --preferred-challenges dns --debug-challenges                                 \
   -d example.org -d \*.example.org
```
Note that the `--debug-challenges` is mandatory here to pause the Certbot execution before asking Let's Encrypt to validate the records and let you to manually add the CNAME records to your main DNS zone.

After adding the prompted CNAME records to your zone(s), wait for a bit for the changes to propagate over the main DNS zone name servers. This takes anywhere from few seconds up to a few minutes, depending on the DNS service provider software and configuration. Hit enter to continue as prompted to ask Let's Encrypt to validate the records.

After the initial run, Certbot is able to automatically renew your certificates using the stored per-domain acme-dns credentials. You will still need to `export` the environment variables before running Certbot.

Whenever you update the script, you should ensure the script is compatible with your existing environment configuration.  You can do check this by invoking the script with a `--version` argument.

````
$ python /etc/letsencrypt/acme-dns-auth.py --version
```

If the version has changed, the best way to update your variables is to generate a new template by invoking the script again with `--setup`, and copy/alter your existing setup as needed

````
$ python /etc/letsencrypt/acme-dns-auth.py --setup
```
