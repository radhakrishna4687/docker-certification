# Set up Security Scanning in DTR
This page explains how to set up and enable Docker Security Scanning on an existing installation of Docker Trusted Registry.
# Prerequisites
- These instructions assume that you have already installed Docker Trusted Registry
- have access to an account on the DTR instance with administrator access.
- make sure that you or your organization has purchased a DTR license that includes Docker Security Scanning, and that your Docker ID can access and download this license from the Docker Hub.
- If you will be allowing the Security Scanning database to update itself automatically, make sure that the server hosting your DTR instance can access
# Get the security scanning license.
# Set repository scanning mode
Two modes are available when Security Scanning is enabled:
1. Scan on push & Scan manually
2. Scan manually
# Update the CVE scanning database
# Update CVE database - online mode
