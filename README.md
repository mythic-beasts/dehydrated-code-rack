# dehydrated-code-rack

Easily invoke multiple dehydrated hooks.

A strength of [dehydrated](https://github.com/lukas2511/dehydrated) is its
ability to call a hook script at various stages in the ACME process. A weakness
is that only a single script is called.

The first argument to the hook indicates the stage in question, so it's easy
enough to dispatch on that. But you can end up with a lot of boilerplate: the
sample `hook.sh` that comes with **dehydrated** is nearly 200 lines long. Of
course, it is heavily commented, but there are over 30 actual lines of code.
That's a lot when in many cases just a single command is needed to be run.

More importantly, the single hook script destroys composability. Mythic Beasts
implemented a hook script that does `dns-01` validation using our DNS API. But
there's no simple way to compose it with, say, nginx reload.

**Code-rack** is a very simple dispatcher based around `run-parts`. Each stage
of dehydrated has its own directory, and you can put any hook scripts you need
in each one.

You will need to instruct dehydrated to call **code-rack** as its hook:

    $ cat /etc/dehydrated/conf.d/hook.sh
    HOOK=/etc/dehydrated/hooks/code-rack

Then create hooks in subdirectories adjacent to **code-rack** (e.g. if `code-rack`
is in `/etc/dehydrated/hooks` then also create these subdirectories there). The
following subdirectories will be used at the appropriate stage of the the
**dehyrated** process:

    deploy-challenge
    clean-challenge
    deploy-cert
    unchanged-cert
    invalid-challenge
    request-failure
    exit-hook

Each hook needs to be a standalone executable, and follow the **run-parts**
rules:

> [T]he names must consist entirely of ASCII upper- and lower-case letters,
> ASCII digits, ASCII underscores, and ASCII minus-hyphens.

For the nginx reload case, the hook is a simple one-liner:

    $ cat /etc/dehydrated/hooks/deploy-cert/nginx
    #!/bin/sh
    /usr/sbin/nginx -s reload

The parameters that **dehydrated** sets for hooks are available in environment
variables. For example our hook for the courier mail system combines the key
and certificate chain into a single file:

    $ cat /etc/dehydrated/hooks/deploy-cert/courier
    #!/bin/sh
    combined="$(dirname "$CERTFILE")"/combined.pem
    umask 077
    cat "$FULLCHAINFILE" "$KEYFILE" > "$combined"

The complete list of variables that will be set for each hook is as follows. If
you have `HOOK_CHAIN=yes` then deploy-challenge and clean-challenge variables
will be for the first domain only and they will also recieve all arguments in the
`ARGS` variable.

    deploy-challenge and clean-challenge
        DOMAIN
        TOKEN_FILENAME
        TOKEN_VALUE
    
    deploy-cert and unchanged-cert
        DOMAIN
        KEYFILE
        CERTFILE
        FULLCHAINFILE
        CHAINFILE
        TIMESTAMP (not for unchanged-cert)
    
    invalid-challenge
        DOMAIN
        RESPONSE
    
    request-failure
        STATUSCODE
        REASON
        REQTYPE
    
    exit-hook
        (none)
