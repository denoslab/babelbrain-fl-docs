# Main Device

The main device is one workstation on the lab network that runs the Starfish-FL router. Every site talks to the router only. The router keeps the projects, runs, models and the model registry in PostgreSQL and moves the weight updates between the sites.

## What it needs

- A fixed address on the lab network that every site reaches, over the LAN or the university wifi.
- Docker, which runs the router and PostgreSQL.
- An HTTPS certificate for the router, signed by the lab's own certificate authority. Each site receives the authority's CA file at enrolment.
- An admin account on the router.

## What the admin does

| Task | How |
| --- | --- |
| Give each site an enrolment code | [Enrolment Codes](enrolment-codes.md) |
| Revoke the token of a site that leaked it or left | [Enrolment Codes](enrolment-codes.md#revoke-a-sites-token) |
| Create a project for the enrolled sites and start its runs | A command for the admin is planned, and the [synthetic tutorial](../tutorials/synthetic-run.md) does this with a test script |

## Status

!!! warning "In progress"
    A setup script for the router behind HTTPS with the lab's certificate authority is in the works, together with the admin command for projects and runs. Until they arrive, the [synthetic tutorial](../tutorials/synthetic-run.md) runs the whole flow on one computer with a test router that only that computer reaches.
