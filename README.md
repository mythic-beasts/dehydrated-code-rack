# dehydrated-code-rack

Easily invoke multiple dehydrated hooks.

A strength of [dehydrated](https://github.com/lukas2511/dehydrated) is its
ability to call a hook script at various stages in the ACME process. A weakness
is that only a single script is called.

Which stage the hook is being called for is available in the first argument, so
it's easy enough to dispatch on that, but you can end up with a lot of
boilerplate. (The sample `hook.sh` that comes with **dehydrated** is nearly 200
lines long. Of course, it is heavily commented, but there are over 30 actual
lines of code. That's a lot when in many cases just a single command is needed
to be run, such as `nginx -s reload`.)

More importantly, the single hook script destroys composability. Mythic Beasts
implemented a hook script that does `dns-01` validation using our DNS API. But
there's no simple way to compose it with, say, nginx reload.

**Code-rack** is a very simple dispatcher based around `run-parts`. Each stage
of dehydrated has its own directory, and you can put any hook scripts you need
in each one.

For the nginx reload case, the hook is a simple one-liner:

    $ cat /etc/dehydrated/hooks/deploy-cert.d/nginx
    #!/bin/sh
    /usr/sbin/nginx -s reload

The parameters that **dehydrated** sets for hooks are available in environment
variables. For example our hook for the courier mail system combines the key
and certificate chain into a single file:

    $ cat /etc/dehydrated/hooks/deploy-cert.d/courier
    #!/bin/sh
    combined="$(dirname "$CERTFILE")"/combined.pem
    umask 077
    cat "$FULLCHAINFILE" "$KEYFILE" > "$combined"
