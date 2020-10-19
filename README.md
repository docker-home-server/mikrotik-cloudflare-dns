# Mikrotik Cloudflare DNS Updater

This project generates a script for [Mikrotik](https://mikrotik.com/) routers
that updates a pair of Cloudflare DNS records
when the router's public IP address has changed.
The two records are for a domain and all it's subdomains:
for example `sub.domain.com` and `*.sub.domain.com`.
I use this to direct traffic to a [Traefik](https://traefik.io/) instance,
with subdomains routed to different services behind the Traefik proxy.

Mikrotik routers include a
[Dynamic DNS feature](https://wiki.mikrotik.com/wiki/Manual:IP/Cloud#DDNS)
that works for many purposes,
and it's possible to use `CNAME` DNS records that point to the Mikrotik
dynamic DNS domain for your router.
But you might prefer a script like this that allows your
router to update the records at Cloudflare directly.

## Setup

`gen-script` will query the Cloudflare API and generate a
script that you can add to your Mikrotik router.
To run `gen-script ` successfully you will need to set the following environment
variables:

* `DOMAIN` - the domain records at Cloudflare that you want to update. For example, `sub.domain.com`.
* `TOKEN` - an API token that you've created in the Cloudflare dashboard. See below.
* `ZONE_ID` - the Zone ID, found on the Cloudflare Overview page
for the top-level domain containing the domain records.

I recommend using [direnv](https://direnv.net/) to set these variables when you
change to this directory.

### API Token

The `TOKEN` value is created in the Cloudflare dashboard.
Follow these steps:

1. Click the profile icon in the top right of the dashboard,
and choose 'My Profile'.
2. Click on 'API Tokens', then 'Create Token'.
3. Click 'Start with a template', then choose the 'Edit zone DNS' template.
4. Under 'Zone Resources', choose your top level domain name
from the pull-down list on the right.
5. Click 'Continue to summary'.
6. Click 'Create Token'.
7. Copy the token shown on the following screen
and set the `TOKEN` environment variable to its value.

## Installation

Once you have set the environment variables listed above
you can run `gen-script`.
It will use the Cloudflare API to find the DNS record ids for the
domain and wildcard subdomain of `DOMAIN` within the zone
identified by `ZONE_ID`.
The value of `TOKEN` will be used to authenticate the API requests.

`gen-script` will output the script to be loaded on to your
router by following these steps:

1. Log in to the Mikrotik web user interface on your router,
or use WinBox if you are familiar with it.
2. Choose 'System' > 'Scripts'.
3. Click 'Add New'.
4. Set an appropriate name (e.g. `mcd`).
5. Under 'Policy', select only 'read', 'write', 'policy', and 'test'.
6. Paste the output from `gen-script` in to the 'Source' field.
7. Click 'Apply'.

You will probably want to configure the router to run the script
every few minutes:

1. Choose 'System' > 'Scheduler'.
2. Click 'Add New'.
3. Set an appropriate name (e.g. `mcd`).
4. Set an interval, such as `00:15:00` for every 15 minutes.
5. Select only the 'read', 'write', 'policy', and 'test' policies, as above.
6. Under 'On Event' enter the name you gave to the script when you created it
(step 4 above).
7. Click 'Apply'.

The script should now run every 15 minutes, updating the
Cloudflare DNS records when the router's public IP address
has changed.
