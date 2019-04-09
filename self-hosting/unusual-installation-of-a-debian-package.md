## Unusual installation of a Debian package

## Installing specific version of the package

* Check available versions either with `apt-cache madison package-name` or `apt-cache policy package-name`
* Install specific version: `apt-get install package-name=1.2.3-3`

## Unattended installation of a Debian package

* Set `DEBIAN_FRONTEND=noninteractive`
* Add option `-q` to `apt-get` to suppress progress bars

### Passing variables

If some package options are necessary during non-interactive install, pass them via `debconf-set-selections`.
This command is provided by `debconf-utils` package.

First check which options package provides (for example for `mysql-server`):

```command
debconf-get-selections | grep mysql-server
```

Then set proper options:

```command
echo mysql-server-5.5 mysql-server/root_password password xyzzy | debconf-set-selections
```

## Installing package for sid

### Create preferences fields

First, create the following files in `/etc/apt/preferences.d`:

* `stable.pref`
    ```ini
    # 500 <= P < 990: causes a version to be installed unless there is a
    # version available belonging to the target release or the installed
    # version is more recent

    Package: *
    Pin: release a=stable
    Pin-Priority: 900
    ```

* `testing.pref`
    ```ini
    # 100 <= P < 500: causes a version to be installed unless there is a
    # version available belonging to some other distribution or the installed
    # version is more recent

    Package: *
    Pin: release a=testing
    Pin-Priority: 400
    ```

* `unstable.pref`
    ```ini
    # 0 < P < 100: causes a version to be installed only if there is no
    # installed version of the package

    Package: *
    Pin: release a=unstable
    Pin-Priority: 50
    ```

* `experimental.pref`
    ```ini
    # 0 < P < 100: causes a version to be installed only if there is no
    # installed version of the package

    Package: *
    Pin: release a=experimental
    Pin-Priority: 1
    ```

Don't be afraid of the unstable/experimental stuff here. The priorities are
low enough that it's never going to automatically install any of that stuff.
Even the testing branch will behave, as it's only going to install
the packages you want to be in testing.

### Create sources lists

Now, creating a matching set for `/etc/apt/sources.list.d`:

* `stable.list`
    ```
    deb http://deb.debian.org/debian/ stable main
    deb-src http://deb.debian.org/debian/ stable main

    deb http://security.debian.org/debian-security stable/updates main
    deb-src http://security.debian.org/debian-security stable/updates main
    ```


* `testing.list`
    ```
    deb http://deb.debian.org/debian/ testing main
    deb-src http://deb.debian.org/debian/ testing main

    deb http://security.debian.org/debian-security testing/updates main
    deb-src http://security.debian.org/debian-security testing/updates main
    ```

* `unstable.list`
    ```
    deb http://deb.debian.org/debian/ unstable main
    deb-src http://deb.debian.org/debian/ unstable main

    deb http://security.debian.org/debian-security unstable/updates main
    deb-src http://security.debian.org/debian-security unstable/updates main
    ```

* `experimental.list`
    ```
    deb http://deb.debian.org/debian/ experimental main
    deb-src http://deb.debian.org/debian/ experimental main

    deb http://security.debian.org/debian-security experimental/updates main
    deb-src http://security.debian.org/debian-security experimental/updates main
    ```

### Set stable as default and refresh packages

```command
echo 'APT::Default-Release "stable";' > /etc/apt/apt.conf.d/99defaultrelease
apt-get update
```

### Install package from desired repository

```command
apt-get -t testing install package
```
