---
title: componentversions
name: check componentversions
url: /docs/cli/check/componentversions/
date: 2024-04-04T14:09:23+02:00
draft: false
images: []
menu:
  docs:
    parent: check
toc: true
isCommand: true
---
### Usage

```
ocm check componentversions [<options>] {<component-reference>}
```

### Options

```
      --fail-on-error      fail on validation error
  -h, --help               help for componentversions
  -R, --local-resources    check also for describing resources with local access method, only
  -S, --local-sources      check also for describing sources with local access method, only
  -o, --output string      output mode (JSON, json, wide, yaml)
      --repo string        repository name or spec
  -s, --sort stringArray   sort fields
```

### Description


This command checks, whether component versions are completely contained
in an OCM repository with all its dependent component references.


If the <code>--repo</code> option is specified, the given names are interpreted
relative to the specified repository using the syntax

<center>
    <pre>&lt;component>[:&lt;version>]</pre>
</center>

If no <code>--repo</code> option is specified the given names are interpreted 
as located OCM component version references:

<center>
    <pre>[&lt;repo type>::]&lt;host>[:&lt;port>][/&lt;base path>]//&lt;component>[:&lt;version>]</pre>
</center>

Additionally there is a variant to denote common transport archives
and general repository specifications

<center>
    <pre>[&lt;repo type>::]&lt;filepath>|&lt;spec json>[//&lt;component>[:&lt;version>]]</pre>
</center>

The <code>--repo</code> option takes an OCM repository specification:

<center>
    <pre>[&lt;repo type>::]&lt;configured name>|&lt;file path>|&lt;spec json></pre>
</center>

For the *Common Transport Format* the types <code>directory</code>,
<code>tar</code> or <code>tgz</code> is possible.

Using the JSON variant any repository types supported by the 
linked library can be used:

Dedicated OCM repository types:
  - <code>ComponentArchive</code>: v1

OCI Repository types (using standard component repository to OCI mapping):
  - <code>CommonTransportFormat</code>: v1
  - <code>OCIRegistry</code>: v1
  - <code>oci</code>: v1
  - <code>ociRegistry</code>



If the options <code>--local-resources</code> and/or <code>--local-sources</code> are given the
the check additionally assures that all resources or sources are included into the component version.
This means that they are using local access methods, only.

With the option <code>--output</code> the output mode can be selected.
The following modes are supported:
  - <code></code> (default)
  - <code>JSON</code>
  - <code>json</code>
  - <code>wide</code>
  - <code>yaml</code>


### Examples

```

$ ocm check componentversion ghcr.io/mandelsoft/kubelink
$ ocm get componentversion --repo OCIRegistry::ghcr.io mandelsoft/kubelink

```

### See Also

* [ocm check](/docs/cli/check)	 &mdash; check components in OCM repository
