---
id: new_overriding_packages
title: Packages
---

The package determines where the content of each input config is placed in the output config.

The default package of an input config is derived from its config group.
(e.g. the package of `db/engine/innodb.yaml` is `db.engine`). It can be overridden in 
the Defaults List or via a Package Directive at the top of the config file.

The priority for determining the final package for a config is as follows:
1. The package specified in the Defaults List (relative to the package of the including config)
2. The package specified in the config header (absolute)
3. The default package


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



#### An example using only default packages

The default packages of *config.yaml* and *server/apache.yaml* are *\_global\_* and *server* respectively:

<div className="row">
<div className="col col--6">

```yaml title="$ python my_app.py" {1-2}
server:
  name: apache
debug: false
```
</div></div>

#### Overriding packages using the Defaults List

Packages specified in the Defaults List are relative to the package of containing config. 
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


:::note
TODO
There are some keyword for convenience.
- \_name\_
- \_here\_

To relocate a package absolutely:
- \_group\_
- \_global\_
:::

### Package specification

``` text title="Definition of a package"
PACKAGE      : _global_ | COMPONENT[.COMPONENT]*
COMPONENT    : _group_ | _name_ | \w+

_global_     : the top level package
_group_      : the config group in dot notation: foo/bar/zoo.yaml -> foo.bar
_name_       : the config file name: foo/bar/zoo.yaml -> zoo
```

:::note
TODO: document `_here_`.
It is only supported in Default Lists and is equivalent to the empty string, making the package of the entry
the same as of the containing config.
:::


---
---
---


### Overriding the package in a file via a package directive

A `@package directive` specifies a common [package](/terminology.md#package) for all nodes in the config file.
It must be placed at the top of each `config group file`.

```text title="Package directive examples"
# @package foo.bar
# @package _global_
# @package _group_
# @package _group_._name_
# @package foo._group_._name_
```
#### Examples
##### A package directive with a literal
<div className="row">
<div className="col col--6">

```yaml title="mysql.yaml" {1-2}
# @package foo.bar

db:
  host: localhost
  port: 3306
```

</div>

<div className="col  col--6">

```yaml title="Interpretation" {1-2}
foo:
  bar:
    db:
      host: localhost
      port: 3306
``` 

</div>
</div>


##### A package directive with `_group_` and `_name_`

<div className="row">
<div className="col col--6">

```yaml title="db/mysql.yaml" {1-2}
# @package _group_._name_

host: localhost
port: 3306
```
</div><div className="col  col--6">

```yaml title="Interpretation" {1-2}
db:
  mysql:
    host: localhost
    port: 3306
``` 
</div></div>

### Overriding the package via the defaults list
The following example adds the `mysql` config in the packages `db.src` and `db.dst`.

:::note
Example of creating two copies of the same config group
:::


<div className="row">
<div className="col col--6">

```yaml title="config.yaml"
defaults:
 - db@db.src: mysql
 - db@db.dst: mysql




```
</div><div className="col  col--6">

```yaml title="Interpretation"
db:
  src:
    host: localhost
    port: 3306
  dst:
    host: localhost
    port: 3306
```
</div></div>

## Content from Defaults List

## Package overrides : take 1
To use a config from outside of the subtree of the current config, one can prefix the entry with `/`, for example:
```yaml title="webserver/apache.yaml"
defaults:
 - /db: mysql   # package: webserver.db
```
In the above example, we are using `db/mysql.yaml` from `webserver/apache.yaml`.
Note that the resulting package for `/db: mysql` is relative to the package of `webserver/apache.yaml`, and is thus `webserver.db`.  

You can override the package of `/db: mysql` to be `db` in the global package using the `_group_` keyword:

```yaml title="webserver/apache.yaml"
defaults:
 - /db@_group_: mysql   # package: db
```

In some scenarios you may want to place the used config in the same package as the containing config. 
Do this by overriding the package to `_here_` or to the empty string (`/db/@_here_`,`/db@`):
```yaml title="webserver/apache.yaml"
defaults:
 - /db@_here_: mysql   # package: webserver
```


:::note
TOOD, Talk about:
- how webserver and engine are relative and db is absolute
- The package of the used config is relative to that of the containing config in all cases  
- The fact the keywords like _group_ and _name_ in the defaults list are interpreted based 
  on the config group + option in that entry
:::


## Relocating config content in the composed config
:::note
TODO: Does this section deserves its own page?
should be merged into Overriding Packages?
:::

The Default Package of a config is derived from the config group it is in
(e.g. the package of `db/engine/innodb.yaml` is `db.engine`).

One may want to place the content into a config package other than the default in some cases, for example, when:
 - Using a config group from another library
 - Using a config from the same config group more than once

Overriding the package of a config relocates the entire subtree.

