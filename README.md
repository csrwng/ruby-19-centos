Ruby 1.9 - CentOS Docker image
========================================

This repository contains the sources and
[Dockerfile](https://github.com/openshift/ruby-19-centos/blob/master/Dockerfile)
of the base image for deploying Ruby 1.9 applications as reproducible Docker
images. The resulting images can be run either by [Docker](http://docker.io)
or using [geard](https://github.com/openshift/geard/).

Installation
---------------

This image is available as trusted build in [Docker Index](https://index.docker.io):

[index.docker.io/u/openshift/ruby-19-centos](https://index.docker.io/u/openshift/ruby-19-centos/)

You can install it using:

```
$ docker pull openshift/ruby-19-centos
```

Repository organization
------------------------

* **`.sti/bin/`**

  This folder contains scripts that are run by [STI](https://github.com/openshift/geard/tree/master/sti):

  *   **assemble**

      Is used to restore the build artifacts from the previous built (in case of
      'incremental build'), to install the sources into location from where the
      application will be run and prepare the application for deployment (eg.
      installing rubygems using bundler, compiling Rails assets, etc..)

  *   **run**

      This script is responsible for running the application, by using the
      application web server. In case of Ruby, the [puma](http://puma.io/)
      server is used, if users put the `gem "puma"` into `Gemfile`. If they do not,
      then the application is run by using the `rackup` command, which should
      honour the application server users provided.

  *   **save-artifacts**

      In order to do an *incremental build* (iow. re-use the build artifacts
      from an already built image in a new image), this script is responsible for
      archiving those. In this image, this script will archive the
      `/opt/src/ruby/bundle` and `Gemfile.lock`.


  *   **test**

      After the application image is built, we want to check that when we create a new
      container based on that image, this container will respond to HTTP calls.
      This image contains a sample Ruby application in the **test-app** folder. This
      sample application is run when a test is performed.

* **`ruby/`**

  This folder is a skeleton of the `$HOME` folder, which is set to `/opt/ruby`.
  It provides a Ruby shebang helper (`ruby` script) and the `usage` script,
  which STI uses to print a usage message when you run this image outside STI.
  This folder also provides minimal, production ready configuration for *puma*
  and also tweaked *.bashrc* and *.gemrc* to optimize the installation of ruby
  gems.


Environment variables
---------------------

*  **APP_ROOT** (default: '.')

    This variable specifies a relative location to your application inside the
    application GIT repository. In case your application is located in a
    sub-folder, you can set this variable to a *./myapplication*.

*  **STI_SCRIPTS_URL** (default: '[.sti/bin](https://raw.githubusercontent.com/openshift/ruby-19-centos/master/.sti/bin)')

    This variable specifies the location of directory, where *assemble*, *run* and
    *save-artifacts* scripts are downloaded/copied from. By default the scripts
    in this repository will be used, but users can provide an alternative
    location and run their own scripts.

Usage
---------------------

**Building the [sinatra-app-example](https://github.com/mfojtik/sinatra-app-example) Ruby 1.9 application..**

1. **using standalone [STI](https://github.com/openshift/geard/tree/master/sti) and running the resulting image by [Docker](http://docker.io):**
    
    ```
$ sti build git://github.com/mfojtik/sinatra-app-example openshift/ruby-19-centos sinatra-app
$ docker run -p 9292:9292 sinatra-app
```
    
2. **using `gear build` and running the resulting image as a systemd unit via [geard](https://github.com/openshift/geard/):**
    
    ```
$ gear build git://github.com/mfojtik/sinatra-app-example openshift/ruby-19-centos sinatra-app
$ gear install sinatra-app sinatra-app-1 -p 9292:9292
$ gear start sinatra-app-1
$ gear list-units
```

**Accessing the application:**
```
$ curl 127.0.0.1:9292
```

Copyright
--------------------

Released under the Apache License 2.0. See the [LICENSE](https://github.com/openshift/ruby-19-centos/blob/master/LICENSE) file.
