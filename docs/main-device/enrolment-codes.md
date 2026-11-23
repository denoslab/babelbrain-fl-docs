# Enrolment Codes

A site enrols with a single-use code that the admin creates on the router. The router stores only a hash of each code, and the code appears once, in the answer that creates it. The commands below use the router's API with the admin account, and curl asks for the password.

## Create a code

```bash
curl -u admin --cacert router-ca.pem -H 'Content-Type: application/json' -d '{"note": "Lab B"}' \
    https://192.168.1.20:8443/starfish/api/v1/enrolment-codes/
```

The answer holds the code and its expiry.

```json
{"id":1,"code":"v9Rfrj536Eb9dg2ztnq8_HWixqvL1xgTbriabkhhWLY","project":null,"expires_at":"2026-11-26T17:56:45.518324Z"}
```

| Field | Meaning |
| --- | --- |
| `note` | Who the code is for |
| `valid_hours` | Hours until the code expires, 72 by default |
| `project` | The ID of a project the site joins when it enrols with this code |

The admin hands the code to the site's practitioner, together with the router's URL and the CA file. A code works once, and the router refuses a used or expired code.

## List codes

```bash
curl -u admin --cacert router-ca.pem https://192.168.1.20:8443/starfish/api/v1/enrolment-codes/
```

The list shows each code's note, expiry and the site that used it, without the codes themselves.

## Revoke a site's token

Enrolment gives each site a token, and the site sends it with every request. A token that leaked, or the token of a site that left the testbed, gets revoked. The first command lists the tokens with their sites and when each was last used, and the second revokes one by its `id`.

```bash
curl -u admin --cacert router-ca.pem https://192.168.1.20:8443/starfish/api/v1/site-tokens/
curl -u admin --cacert router-ca.pem -X POST https://192.168.1.20:8443/starfish/api/v1/site-tokens/<id>/revoke/
```

The router refuses the revoked token from then on, and the site's check reports that the router refused its credentials. A new code and `starfish-site enrol --force` give the site a new token and keep its ID.
