# Release Notes

## starfish-site 0.1.0

The first version of the native site, built on 2026-11-23.

- `install.sh` installs a site on Ubuntu or macOS without admin rights, with torch for CUDA 12.8 on Ubuntu and with MPS on a Mac.
- `enrol`, `check`, `start` and `stop` join the router with an enrolment code, test the computer, run the site and stop it.
- One process does the work of the controller's Celery workers and Redis and runs the controller's own task code.
- `check` also runs before enrolment and then tests torch and training on the computer.

On an Ubuntu workstation with an RTX 4070 Ti, three native sites ran 3 rounds of 4 local epochs in 90 seconds, two of them on the GPU and one on the CPU. Enrolment and a run also worked over HTTPS through a proxy with a test certificate authority.

### Known issues

- It has not run on macOS or on a Blackwell card such as the RTX PRO 6000 so far.
- It has no background service, and a site runs as long as its terminal stays open.
- The setup of the main device with HTTPS is missing from the repository.
