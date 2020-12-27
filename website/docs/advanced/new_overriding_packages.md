---
id: new_overriding_packages
title: Packages
---

The package determines where the content of each input config is placed in the output config.

The default package of an input config is derived from its config group.
(e.g. the package of `db/engine/innodb.yaml` is `db.engine`). It can be overridden in 
the Defaults List or via a Package Directive at the top of the config file.

One may want to package of a config in some cases, for example, when:
 - Using a config group from another library
 - Using a config from the same config group more than once

We will use the following structure in the examples below:

```text title="Config directory structure"
├── server
│   ├── db
│   │   ├── mysql.yaml
│   │   └── sqlite.yaml
│   └── apache.yaml
└── config.yaml
```
Input configs:
<div className="row">
<div className="col col--4">

```yaml title="config.yaml"
defaults:
  - server/apache

debug: false



```
</div>

<div className="col col--4">

```yaml title="server/apache.yaml"
defaults:
  - db: mysql

name: apache



```
</div>

<div className="col col--4">

```yaml title="server/db/mysql.yaml"
name: mysql
```

```yaml title="server/db/sqlite.yaml"
name: sqlite
```
</div></div>

### An example using only default packages

The default packages of *config.yaml* and *server/apache.yaml* are *\_global\_* and *server* respectively:

<div className="row">
<div className="col col--6">

```yaml title="$ python my_app.py" {1-2}
server:
  name: apache
debug: false
```
</div></div>

### Overriding packages using the Defaults List
By default, packages specified in the Defaults List are relative to the package of containing config. 
As a consequence, overriding a package relocates the entire subtree. 

<div className="row">
<div className="col col--4">

```yaml title="config.yaml" {2}
defaults:
  - server/apache@admin

debug: false

```
</div>
<div className="col col--4">

```yaml title="server/apache.yaml" {2}
defaults:
 - db@backup: mysql

name: apache

```
</div>
<div className="col col--4">

```yaml title="Output config" {1-4}
admin:
  backup:
    name: mysql
  name: apache
debug: false
```
</div></div>

Note that *server/db/mysql.yaml* is relocated to *admin.backup*.

#### Default List package keywords
Use `_here_` to relocate a config in the Defaults List to the package of the containing config. e.g:
```yaml title="config_group/config.yaml"
defaults:
  - /engine/db@_here_: mysql         # package: config_group
```

The keyword `_name_` is being substituted by the name of the config:
```yaml title="config_group/config.yaml"
defaults:
  - /engine/db@_name_: mysql        # package: config_group.mysql
```

There are cases when the desired package is absolute.  
Use `_global_` as a prefix to relocate to an arbitrary absolute package:
```yaml title="config_group/config.yaml"
defaults:
  - /engine/db@_global_.foo   # package: foo 
```

Use `_group_` to relocate to the absolute default package of the used config: 
```yaml title="config_group/config.yaml"
defaults:
  - /engine/db@_group_        # package: engine.db 
```

### Overriding the package via the package directive

The `@package directive` can change the default package for a config file. 

```yaml title="server/db/mysql.yaml" {1}
# @package foo.bar
name: mysql
```

To change the package to the global (empty) package, use the keyword `_global_`.

### Determining the final package
The priority for determining the final package for a config is as follows:
1. The package specified in the Defaults List (relative to the package of the including config)
2. The package specified in the config header (absolute)
3. The default package

:::info
Configs with package directive are absolute unless their package is overridden directly in the including
Defaults List.
:::

### Using a config group more than once
The following example adds the `db/server/mysql` config in the packages `src` and `dst`.

<div className="row">
<div className="col col--6">

```yaml title="config.yaml"
defaults:
 - server/db@src: mysql
 - server/db@dst: mysql

```
</div><div className="col  col--6">

```yaml title="$ python my_app.py"
src:
  name: mysql
dst:
  name: mysql
```
</div></div>

When overriding config groups with a non-default package, the package must be used:
```yaml title="$ python my_app.py server/db@src=sqlite"
src:
  name: sqlite
dst:
  name: mysql
```

