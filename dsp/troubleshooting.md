---
title: DSP Troubleshooting
description: Troubleshooting common issues on the Digital Solutions Platform (DSP)
---

[&larr; back to Overview](/dsp)


# CI / CD

## The pipeline is failing with the error `/var/run/docker.sock: connect: connection refused`

This error occurs because we're using a rootless Docker setup in the GitHub Actions runner. The docker socket is not available at `/var/run/docker.sock` in this setup but is located at `/home/runner/var/run/docker.sock`.
Some tools including the [Cloud Native Buildpacks implementation of Spring Boot](Packaging OCI Images) require the docker socket to be available at `/var/run/docker.sock`. 

To work around this issue in spring boot, you can set the `bindHostToBuilder` configuration to `true`. This will bind the host's docker socket to the builder container.
