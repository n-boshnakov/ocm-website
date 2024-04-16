---
title: logging
name: ocm logging
url: /docs/cli/cli-reference/ocm/ocm/logging/
date: 2024-04-16T15:24:23+02:00
draft: false
images: []
toc: true
sidebar:
  collapsed: true
---
### Description


Logging can be configured as part of the ocm config file ([ocm configfile](/docs/cli/cli-reference/ocm/configfile))
or by command line options of the [ocm](/docs/cli/cli-reference/ocm) command. Details about
the YAML structure of a logging settings can be found on https://github.com/mandelsoft/logging.

The command line also supports some quick-config options for enabling log levels
for dedicated tags and realms or realm prefixes (logging keys).

The following *tags* are used by the command line tool:
  - <code>blobhandler</code>: execution of blob handler used to upload resource blobs to an ocm repository.



The following *realms* are used by the command line tool:
  - <code>ocm</code>: general realm used for the ocm go library.
  - <code>ocm/NPM</code>: NPM registry
  - <code>ocm/accessmethod/ociartifact</code>: access method ociArtifact
  - <code>ocm/accessmethod/wget</code>: access method for wget
  - <code>ocm/blobaccess/wget</code>: blob access for wget
  - <code>ocm/compdesc</code>: component descriptor handling
  - <code>ocm/config</code>: configuration management
  - <code>ocm/context</code>: context lifecycle
  - <code>ocm/credentials/dockerconfig</code>: docker config handling as credential repository
  - <code>ocm/credentials/vault</code>: HashiCorp Vault Access
  - <code>ocm/downloader</code>: Downloaders
  - <code>ocm/oci/mapping</code>: OCM to OCI Registry Mapping
  - <code>ocm/oci/ocireg</code>: OCI repository handling
  - <code>ocm/plugins</code>: OCM plugin handling
  - <code>ocm/processing</code>: output processing chains
  - <code>ocm/refcnt</code>: reference counting
  - <code>ocm/toi</code>: TOI logging
  - <code>ocm/transfer</code>: OCM transfer handling
  - <code>ocm/valuemerge</code>: value marge handling



### Examples

```

    type: logging.config.ocm.software
    contextType: attributes.context.ocm.software
    settings:
      defaultLevel: Info
      rules:
        - ...

```

### See Also

