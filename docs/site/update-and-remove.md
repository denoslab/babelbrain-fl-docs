# Update and Remove

## Update

The site runs the code of its checkout, and a pull brings new code that the site runs from its next start.

```bash
starfish-site stop
cd starfish-fl
git pull
starfish-site/install.sh
starfish-site start
```

The installer matters only when an update changes the dependencies, which the [Release Notes](../references/release-notes.md) mention. Running it again keeps the site's settings and run files.

## Remove

```bash
starfish-site stop
rm -rf ~/.starfish-site ~/.local/bin/starfish-site
```

This deletes the site's settings, logs and run files together with the installed software. uv keeps its download cache in `~/.cache/uv`, which can go as well. The sample store and the checkout stay where they are.

The site's token stays valid on the router until the admin revokes it, as [Enrolment Codes](../main-device/enrolment-codes.md#revoke-a-sites-token) shows. A site that leaves the testbed asks the admin to do so.
