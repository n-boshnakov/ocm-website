---
title: artifacts
name: describe artifacts
url: /docs/cli/cli-reference/ocm/describe/artifacts/
date: 2024-04-16T15:24:23+02:00
draft: false
images: []
toc: true
sidebar:
  collapsed: true
---
### Usage

```
ocm describe artifacts [<options>] {<artifact-reference>}
```

### Options

```
  -h, --help            help for artifacts
      --layerfiles      list layer files
  -o, --output string   output mode (JSON, json, yaml)
      --repo string     repository name or spec
```

### Description


Describe lists all artifact versions specified, if only a repository is specified
all tagged artifacts are listed.
Per version a detailed, potentially recursive description is printed.



If the repository/registry option is specified, the given names are interpreted
relative to the specified registry using the syntax

<center>
    <pre>&lt;OCI repository name>[:&lt;tag>][@&lt;digest>]</pre>
</center>

If no <code>--repo</code> option is specified the given names are interpreted 
as extended OCI artifact references.

<center>
    <pre>[&lt;repo type>::]&lt;host>[:&lt;port>]/&lt;OCI repository name>[:&lt;tag>][@&lt;digest>]</pre>
</center>

The <code>--repo</code> option takes a repository/OCI registry specification:

<center>
    <pre>[&lt;repo type>::]&lt;configured name>|&lt;file path>|&lt;spec json></pre>
</center>

For the *Common Transport Format* the types <code>directory</code>,
<code>tar</code> or <code>tgz</code> are possible.

Using the JSON variant any repository types supported by the 
linked library can be used:
  - <code>ArtifactSet</code>: v1
  - <code>CommonTransportFormat</code>: v1
  - <code>DockerDaemon</code>: v1
  - <code>Empty</code>: v1
  - <code>OCIRegistry</code>: v1
  - <code>oci</code>: v1
  - <code>ociRegistry</code>


With the option <code>--output</code> the output mode can be selected.
The following modes are supported:
  - <code></code> (default)
  - <code>JSON</code>
  - <code>json</code>
  - <code>yaml</code>


### Examples

```

$ ocm describe artifact ghcr.io/mandelsoft/kubelink
$ ocm describe artifact --repo OCIRegistry::ghcr.io mandelsoft/kubelink

```

### See Also

* [ocm describe](/docs/cli/cli-reference/ocm/describe)	 &mdash; Describe various elements by using appropriate sub commands.
