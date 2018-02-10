Ansible Development Environment
=========

An Ansible role to install and configure necessary softwares and toolings for the personal development environment in Vagrant boxes.

Currently, it supports:
- Install and configure author information and ssh key pairs for `git`
- Install Java (OpenJDK) and set `JAVA_HOME`
- Install Maven and set `M2_HOME`
- Install Docker
- Install `kubectl` and configure it to connect to remote cluster
- Install `helm`
- Install `httpie`
- Install MongoDB and configure it
- Configure kubernetes cluster to insert private docker registry secret

It plans to support in the future:
- Add tags to allow more granular control
- Install Go, set `GOROOT` and set `GOPATH`
- Install Gradle
- Install and configure MariaDB
- Install and configure Redis
- Install and configure RabbitMQ

Requirements
------------

This role is solely designed for `Ubuntu xenial` Vagrant images. User may choose
to define `ops.secretDir` variable, in which YAML files containing secret
variables like user name and password will be placed. All other variables
may be provided to the role via the `vars` key. For details, see the
**Role Variables** section.

Configuration Options
--------------

### Install Toggles

Install toggles to select which software package should be installed and
configured.

Key | Default
--- | ---
`install.git` | `true`
`install.*` | `false`

Example config to install `git`, `java`, `maven` and `httpie`:
```
install:
  git: true
  java: true
  maven: true
  gradle: false
  go: false
  kubectl: false
  helm: false
  docker: false
  mongodb: false
  mariadb: false
  redis: false
  httpie: true
```

### Configure Toggles

Configuration toggles will switch on and off which configuration actions
to perform.

Key | Default
--- | ---
`configure.k8sPrivateRegistryImagePull` | `false`

### Git Variables

When `install.git` is `true`, this role will perform its jobs related to git.
If `git.useExistingKey` is set to `true`, this role will look for
`git.existingKeyPath` and the corresponding public key (`.pub`) files and copy
them to `/home/vagrant/.ssh/` under the name `git.keyName`.

If `git.useExistingKey` is set to `false`, it will generate a new key pair
using the name specified by `git.keyName`.

In addition, `git.user.name` and `git.user.email` must be provided. Since
these are personal details are not suitable to be version track, users can
choose to put it in the `ops.secretDir` directory and ignore it with version
control.

Key | Default
--- | ---
`git.useExistingKey` | `false`
`git.existingKeyPath` | `/home/vagrant/from-host/ssh-keys/id_rsa`
`git.keyName` | `id_rsa`

Example config to use existing key pair call `my_key` and copy over as `id_rsa`:
```
git:
  useExistingKey: true
  existingKeyPath: /home/vagrant/from-host/ssh-keys/my_key
  keyName: id_rsa
```

### Java Variables

If `install.java` is `true`, this role installs OpenJDK from apt-repository and
sets up `JAVA_HOME`.

Key | Default
--- | ---
`java.version` | `8`

Example config to install OpenJDK 8:
```
java:
  version: 8
```

### Maven Variables

If `install.maven` is `true`, this role downloads Maven archive, unarchives it
and puts it to `maven.home`. It then creates a profile file to setup `M2_HOME`
environment variable. In addition, if `maven.forceReinstall` is set to `true`,
the role will attempt to uninstall existing installation first and install a
fresh copy. This is useful when we want to upgrade Maven.

Key | Default
--- | ---
`maven.forceReinstall` | `false`
`maven.version` | `3.5.2`
`maven.home` | `/usr/lib/maven`

Example config to _downgrade_ to Maven `3.3.0`:
```
maven:
  forceReinstall: true
  version: 3.3.0
  home: /usr/lib/maven
```

### Docker Variables

Key | Usage | Default
--- | --- | ---
`docker.registryUrl` | identify registry | `https://index.docker.io/v1/`
`docker.user.name` | user name | no default
`docker.user.password` | user password | no default
`docker.user.email` | user email | no default

### Kubectl Variables

If `install.kubectl` is set to `true`, this role will try to install and
configure kubectl. The script will only install kubectl when this is no existing
installation of `kubectl` binary under `kubectl.installDir`. However, this
behaviour can be overriden by `kubectl.forceReinstall`, which will perform
uninstallation first and then a fresh installation. Note the the uninstallation
process will also remove any config files at `<kubectl.config.targetDir>/<kubectl.config.name>`.

As for the configuration side, when `kubectl.config.sourceFile` is set, the script
will just copy the config file at `kubectl.config.sourceFile` to
`<kubectl.config.targetDir>/<kubectl.config.name>`. Otherwise, it will attempt
to render the config file using variables defined under `kubectl.config.props.*`.

The options provided by `kubectl.config.props.*` is slightly more restrictive
than a plain kubectl client config file. It assumes only one cluster, one context
and one user, and therefore simplifies the list entries to only one options. This
should fit most needs, however, if multiple user and multiple contexts are needed,
user can always provide a complete config at `kubectl.config.sourceFile`.

Key | Default
--- | ---
`kubectl.forceReinstall` | `false`
`kubectl.version` | `1.9.2`
`kubectl.installDir` | `/usr/local/bin`
`kubectl.config.name` | `config`
`kubectl.config.targetDir` | `/home/vagrant/.kube`
`kubectl.config.props.cluster.certificateAuthorityData` | blank
`kubectl.config.props.cluster.insecureSkipTLSVerify` | `false`
`kubectl.config.props.user.name` | `admin`

When `kubectl.config.sourceFile` is not set, the following variables must be set:
- `kubectl.config.props.cluster.name`
- `kubectl.config.props.cluster.server`
- `kubectl.config.props.context.name`
- `kubectl.config.props.user.password`

Example configuration to install kubectl version 1.9.2 and configure it to target
a home cluster.
```
kubectl:
  forceReinstall: false
  version: 1.9.2
  installDir: /usr/local/bin
  config:
    name: config
    targetDir: /home/vagrant/.kube
    props:
      cluster:
        name: juju-cluster
        certificateAuthorityData: ""
        server: https://192.168.100.199:443
        insecureSkipTLSVerify: false
      context:
        name: juju-context
      user:
        name: admin
        password: redacted
```

### Helm Variables

If `install.helm` is set to `true`, the script will attempt to install helm of
version `helm.version` to `helm.installDir`. If `helm.forceReinstall` is set,
the script will attempt to remove existing helm first.

Key | Default
--- | ---
`helm.forceReinstall` | `false`
`helm.version` | `2.6.0`
`helm.installDir` | `/usr/local/bin`

Example config to install helm:
```
helm:
  forceReinstall: false
  version: 2.6.0
  installDir: /usr/local/bin
```

### Mongo Variables

If `install.mongo` is set to `true`, the script will attempt to install mongoDB.
However, the script will skip if MongoDB is already installed. During
configuration, if `mongo.config.sourceFile` is set, the script will copy the
file to `mongo.config.targetFile`; otherwise, it will try to render the config
file using values under `mongo.config.props.*`. The script will restart MongoDB
after config file is in place.

If `mongo.auth.enabled` is set to `true`, the script will create an admin database
user under the name `mongo.auth.user` and password `mongo.auth.password` with
roles `mongo.auth.roles`. Please note that this is not the database user you
may wanna connect to. To connect to a custom database, you still have to login
to MongoDB and do something like:
```
use newDB
db.createUser({user: "tom", pwd: "s3cret", roles:[{role:"readWrite",db:"newDB"}]})
```

Key | Default
--- | ---
`mongo.version` | `3.6`
`mongo.config.targetFile` | `/etc/mongod.conf`
`mongo.config.props.port` | `27017`
`mongo.config.props.ip` | `listOf(127.0.0.1)`
`mongo.auth.enabled` | `false`
`mongo.auth.roles` | `readWriteAnyDatabase`

Example config to install MongoDB and configure it listen for requests on both
`127.0.0.1` and `192.168.35.100` interface.
```
mongo:
  version: 3.6
  config:
    targetFile: /etc/mongod.conf
    props:
      port: 27017
      ip:
        - 127.0.0.1
        - 192.168.35.100
```

### k8sPrivateRegistryImagePull Variables

`k8sPrivateRegistryImagePull` task configures a kubernetes docker registry
Secret object in the cluster with the specified parameters:

Key | Usage
--- | ---
`k8sPrivateRegistryImagePull.namespace` | The K8S namespace to create the secret, defaulted
`kubectl.config.targetDir` | The directory for the `kubectl` config file, defaulted
`kubectl.config.name` | The `kubectl` config file name, defaulted
`k8sPrivateRegistryImagePull.secretName` | The name for the registry secret, defaulted
`docker.registryUrl` | docker registry url, defaulted
`docker.user.name` | docker username, user specified
`docker.user.password` | docker password, user specified
`docker.user.email` | docker email, user specified

Here are the default values:

Key | Default
--- | ---
`k8sPrivateRegistryImagePull.namespace` | `default`
`k8sPrivateRegistryImagePull.secretName` | `regsecret`

### Other Variables

Key | Default | Use
--- | --- | ---
`ops.downloadDir` | `/tmp` | used for downloading temporary files
`ops.secretDir` | no default | directory, if defined, used to put secret YAML files
