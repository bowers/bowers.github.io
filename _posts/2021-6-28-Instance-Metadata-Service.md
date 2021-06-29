---
layout: post
title: Script for Retriving Instance Metadata Service values in Public Clouds
---

AWS, Azure, Google Cloud, and Oracle Cloud each provide an interface for virtual machines to read metadata information about themselves.  All of these clouds call this the Instance Metadata Service (IMDS). 

The cloud IMDS includes information like the instance type, the region housing it, the image used, and the VMs private IP address(es). While each of these clouds provide the metadata via HTTP access to an endpoint visibile only from within the instance, the format of the request and of the data differs between the clouds.

[Here is a bash script for Linux called get-imds](https://github.com/bowers/get-imds) that reads commonly-required metadata fields, regardless of which cloud you're on.

Below are links to each cloud's documents on their IMDS, and one example of using each one.

### [Amazon Web Services IMDS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)
AWS offers 2 versions of the IMDS, the second being more secure. You can force instances at launch to use only the second (IMDSv2).

Example: ```wget http://169.254.169.254/latest/meta-data/instance-id```

### [Microsoft Azure IMDS](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/instance-metadata-service?tabs=linux)
With Azure, you can retrive a signature as a metadata field to verify it's authenticity. Microsoft has a repo holding lots of sample requests (here.)[https://github.com/microsoft/azureimds]

Example: ```curl -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance/compute/subscriptionId?api-version=2019-06-01&format=text"```

### [Oracle Cloud IMDS](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/gettingmetadata.htm)
Like AWS, Oracle Cloud offers an older IMDSv1 and newer (more secure) IMDSv2.

Example: ```curl http://169.254.169.254/opc/v1/instance/region```

### [Google Cloud IMDS](https://cloud.google.com/compute/docs/storing-retrieving-metadata)
Google provides a domain name (metadata.google.internal) to access the interface, instead of just an IP address. You must provide a specific header in the HTTP request for Google's IMDS to respond.

Example: ```curl -H "Metadata-Flavor: Google" "http://metadata.google.internal/computeMetadata/v1/instance/disks/"```


