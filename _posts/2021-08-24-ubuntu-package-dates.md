---
layout: post
title: Find the Date that a Package in the Ubuntu Repository was Created
---
This blog shows how to use the API on Ubuntu's "Launchpad" package creation system to retrieve metadata, like the date a patch was created

### Fetching Metadata from Launchpad
Ubuntu has [four main, official repositories](https://help.ubuntu.com/community/Repositories/Ubuntu) where packages are stored. You typically run [apt](http://manpages.ubuntu.com/manpages/bionic/man8/apt.8.html) or [apt-get](http://manpages.ubuntu.com/manpages/cosmic/man8/apt-get.8.html) to find and download the latest packages.  You can look through these repositories and search for packages though a web browser at [https://packages.ubuntu.com/](https://packages.ubuntu.com/).

[Canonical](https://canonical.com/), the makers of Ubuntu, use a system called [Launchpad](https://launchpad.net/) to create the packages and store them in the repositories. You can search through Launchpad with a browser and find information like a package's detailed history.  Launchpad also has a [poorly documented API](https://help.launchpad.net/API) that lets you retrieve this same information.

This [example shell program](https://raw.githubusercontent.com/bowers/util-disk/main/package-date-launchpad)example shell program uses HTTP requests to the API to retrieve package creation dates.  This specific script reports dates for packages in the [updates or security pockets](https://docs.ubuntu.com/landscape/en/repositories) of Ubuntu 20.04 ("Focal") that the apt utility says have available updates.

The key line showing this fetch from the API is:
```
sources_json=$(curl -s "https://api.launchpad.net/1.0/ubuntu/+archive/primary?ws.op=getPublishedBinaries&exact_match=true&$pocket=Security&status=Publi
shed&distro_arch_series=https://api.launchpad.net/1.0/ubuntu/focal/$architecture&binary_name=$package")

```
Running this script (```./package-date-launchpad```) results in output like this:

```
Package: openssh-client
Pocket: focal-updates
Architecture: amd64
New Version Available: 1:8.2p1-4ubuntu0.3
Dates for all versions:
"openssh-client Published 1:8.2p1-4ubuntu0.3 2021-08-13T07:23:55.517155+00:00"
"openssh-client Published 1:8.2p1-4ubuntu0.2 2021-03-10T14:33:26.528429+00:00"
"openssh-client Published 1:8.2p1-4 2020-03-30T13:43:28.232861+00:00"

Package: openssl
Pocket: focal-updates
Architecture: amd64
New Version Available: 1.1.1f-1ubuntu2.5
Dates for all versions:
"openssl Published 1.1.1f-1ubuntu2.5 2021-08-13T07:23:55.517155+00:00"
"openssl Published 1.1.1f-1ubuntu2.3 2021-03-25T14:33:39.771591+00:00"
"openssl Published 1.1.1f-1ubuntu2 2020-04-21T22:13:20.156919+00:00"

```
