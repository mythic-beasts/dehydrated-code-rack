# Hooks

The hooks in this directory are ready to be symlinked to from whichever hook points you wish to use them for.

## `reload_`

Reload a specific service.  To use, create a symlink to it in the appropriate hook directory, with the name `reload_{service}` where `{service}` is  the name of the service you wish to reload.  For example, to reload postfix when a certificate is deployed, create a symlink to `reload_` with the name `reload_postfix` in the `deploy-cert` directory.  Combine with `ifdomain_` (see below) to reload services only when the SSL certficiate they use is updated.

## `ifdomain_`

Only runs the hook if the domain triggering it matches.

Due to the restrictions of run-parts (only letters, numbers, dashes and underscores are permitted), domains in the filenames needs to have the periods replaced with dashes.  For example, 'google.co.uk' would be 'google-co-uk' in the filename.

Use it by creating a symlink with the name `ifdomain_{domain}_{hook to run}`.  The hook you want to run must exist in the same directory as the `ifdomain_` script your symlink points to.  For example, `ifdomain_google-co-uk_reload_nginx` will call the `reload_nginx` hook if the domain is 'google.co.uk'.  (Note that this means you need to create a symlink, in the same place as `ifdomain_` is, called `reload_nginx` pointing to `reload_`, if you want to use that).


## HP iLO scripts

### Configuration

To use these scripts, create an ini-style configuration file in _/etc/dehydrated/hp-ilo_ called `{domain}.ini` - for example `my-server-ilo.my-domain.tld.ini` for _my-server-ilo.my-domain.tld_ (note there is no substituion of periods in the domain in this filename) - that looks like this:

```ini
[ilo]
login = username
password = password

[certificate]
organizational_unit = my department
organization = my organisation
locality = My city
state = My county
country = GB
```

It seems the certificate settings are essentially ignored by Let's Encrypt, however without them you get the HP iLO's built in defaults and I wrote it to use them before I realised it was pointless so they are currently required for the script to work.

**Make sure to secure the ini file** so only the dehydrated user can read it, as it contains the username and password for your iLO.  I highly recommend creating a dedicated user for dehydrated, so it is easy to revoke rights if compromised and it does not need the ability to modify users, access the console or any of the other ilo features (just configure the iLO).

### `hp-ilo-csr`

Gets a CSR from a HP iLO out-of-band management controller at the address of the certificate domain (which is the only sensible way - you would not want an iLO certificate for the domain if there were no iLO there).

You will want to use this at the _generate-csr_ hook stage.

### `hp-ilo-deploy`

Deploy a certificate to the HP iLO located at the domain's address.

You will want to use this at the _deploy-cert_ hook stage.
