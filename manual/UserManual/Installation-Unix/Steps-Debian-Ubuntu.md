# Installation on Debian and Ubuntu Linux Distributions

Below are the steps for installing ClamAV from source on Debian and Ubuntu Linux.

---

## Install prerequisites

1. Install ClamAV dependencies
    1. Install the developer tools
        <pre>
            sudo apt-get install build-essential
        </pre>
    2. Install library dependencies
        <pre>
            sudo apt-get install openssl libssl-dev libcurl4-openssl-dev zlib1g-dev libpng-dev libxml2-dev libjson-c-dev libbz2-dev libpcre2-dev ncurses-dev
        </pre>
    3. (very optional) Those wishing to use clamav-milter may wish to install the following
        <pre>
            sudo apt-get install libmilter1.0.1 libmilter-dev
        </pre>

2. Install the unit testing dependencies
    <pre>
        sudo apt-get install valgrind check check-devel
    </pre>

_Note_: LLVM is also an optional dependency. LLVM will not provide any additional features, but is an alternative method for executing bytecode signatures versus using the built-in bytecode interpreter. Limited performance testing between LLVM and the bytecode interpreter did not yield conclusive evidence that one is "better" than the other. For the sake of simplicity, it is not recommended to install LLVM.

---

## Download the latest stable release

1. Open a browser and navigate to [the ClamAV downloads page](https://www.clamav.net/downloads)
2. Click `clamav-<version>.tar.gz` link to download the latest stable release.

---

## Extract the source archive

<pre>
    cd ~/Downloads
    tar xzf clamav-[ver].tar.gz
    cd clamav-[ver]
</pre>

---

## Configure the build

ClamAV's configure script should detect each of the above dependencies automatically.

### Typical `./configure` usage

<pre>
    ./configure --enable-check
</pre>

Once `./configure` completes, it will print a summary. Verify that the packages you installed are in fact being detected.

Example configure summary output:

<pre>
    configure: Summary of detected features follows
                OS          : linux-gnu
                pthreads    : yes (-lpthread)
    configure: Summary of miscellaneous features
                check       : -lcheck_pic -pthread -lrt -lm -lsubunit
                fanotify    : yes
                fdpassing   : 1
                IPv6        : yes
    configure: Summary of optional tools
                clamdtop    : -lncurses (auto)
                milter      : yes (disabled)
                clamsubmit  : yes (libjson-c-dev found at /usr), libcurl-devel found at /usr)
    configure: Summary of engine performance features
                release mode: yes
                llvm        : no (disabled)
                mempool     : yes
    configure: Summary of engine detection features
                bzip2       : ok
                zlib        : /usr
                unrar       : yes
                preclass    : yes (libjson-c-dev found at /usr)
                pcre        : /usr
                libmspack   : yes (Internal)
                libxml2     : yes, from /usr
                yara        : yes
                fts         : yes (libc)
</pre>

### Additional popular `./configure` options

* `--with-systemdsystemunitdir` - Do not install `systemd` socket files. This option disables systemd support, but will allow you to `make install` to a user-owned directory without requiring `sudo`/root privileges:
    <pre>
        ./configure --with-systemdsystemunitdir=no
    </pre>
* `--sysconfdir` - Install the configuration files to `/etc` instead of `/usr/local/etc`:
    <pre>
        ./configure --sysconfdir=/etc
    </pre>
* `--prefix` - Install ClamAV to a directory other than `/usr/local/`:
    * Example 1: Install to a local `./install` directory.
        <pre>
            ./configure --prefix=`pwd`/install
        </pre>
    * Example 2: Install ClamAV locally on an unprivileged shell account.
        <pre>
            ./configure --prefix=$HOME/clamav --disable-clamav --with-systemdsystemunitdir=no
        </pre>
* `--disable-clamav` - _Don't_ drop super-user priveleges to run `freshclam` or `clamd` as the `clamav`* user.
    <pre>
        ./configure --disable-clamav
    </pre>
    *_Tip_: Using this `--disable-clamav` means that `freshclam` and `clamd` will run with _root privleges_ if invoked using `sudo`. Running `clamd` or `clamscan` as root is **not recommended**. Instead of using this option, you can configure `freshclam` or `clamd` to drop to any other user by:
    * setting the `DatabaseOwner` option in `freshclam.conf` and
    * setting the `User` option in `clamd.conf`.

Please see the `./configure --help` for additional options.

---

## Compile ClamAV

Compile ClamAV with:
<pre>
    make -j2
</pre>

---

## Run ClamAV Unit Tests (Optional)

For peace of mind, it can be helpful to run a small suite of unit and system tests.

Run:
<pre>
    make check
</pre>

All tests should pass.* Output will look something like this:

<pre>
        ...
    PASS: check_clamav
    PASS: check_freshclam.sh
    PASS: check_sigtool.sh
    PASS: check_unit_vg.sh
    PASS: check1_clamscan.sh
    PASS: check2_clamd.sh
    PASS: check3_clamd.sh
    PASS: check4_clamd.sh
    PASS: check5_clamd_vg.sh
    PASS: check6_clamd_vg.sh
    SKIP: check7_clamd_hg.sh
    PASS: check8_clamd_hg.sh
    PASS: check9_clamscan_vg.sh
        ...
    ============================================================================
    Testsuite summary for ClamAV 0.100.2
    ============================================================================
    # TOTAL: 13
    # PASS:  12
    # SKIP:  1
    # XFAIL: 0
    # FAIL:  0
    # XPASS: 0
    # ERROR: 0
</pre>

_Notes_:

* The `*.vg.sh` tests will be skipped unless you run `make check VG=1`.
* The `check7_clamd.hg.sh` (helgrind) is presently disabled and will be skipped.
  * For details, see: [the Git commit](https://github.com/Cisco-Talos/clamav-devel/commit/2a5d51809a56be9a777ded02969a7427a3c26713)

If you have a failure or an error in the unit tests, it could be that you are missing one or more of the prerequisites.

If you are investigating a failure, please do the following:

`cd unit_tests`

Use `less` to read the log for the failed test.
Example:

<pre>
    less check4_clamd.sh.log`
</pre>

To submit a bug report regarding unit text failures, please follow these [bug reporting steps](https://www.clamav.net/documents/installing-clamav-on-unix-linux-macos-from-source#Reporting-a-unit-test-failure-bug).

---

## Install ClamAV

Install ClamAV with:
<pre>
    make install
</pre>

_Tip_: If installing to the default or other system-owned directory, you may need to use `sudo`.

---

## First time set-up

_Note_: The following instructions assume you used the default install paths (i.e. `/usr/local`). If you modified the install locations using `--prefix` or `--sysconfdir` options, replace `/usr/local` with your chosen install path.

---

### `freshclam` config

Before you can use `freshclam` to download updates, you need to create a `freshclam` config. A sample config is provided for you.

1. Copy the sample config. You may need to use `sudo`:
    <pre>
        cp /usr/local/etc/freshclam.conf.sample /usr/local/etc/freshclam.conf
    </pre>
2. Modify the config file using your favourite text editor. Again, you may need to use `sudo`.
    * At a minimum, remove the `Example` line so `freshclam` can use the config.

    Take the time to look through the options. You can enable the sample options by deleting the `#` comment characters.

    Some popular options to enable include:

    * `LogTime`
    * `LogRotate`
    * `NotifyClamd`
    * `DatabaseOwner`

3. Create the database directory. *Tip: _You may need to use `sudo`._
    <pre>
        mkdir /usr/local/share/clamav
    </pre>

---

### `clamd` config (optional)

You can run `clamscan` without setting the config options for `clamd`. However, the `clamd` scanning daemon allows you to use `clamdscan` to perform faster a-la-carte scans, allows you to run multi-threaded scans, and allows you to use `clamav-milter` if you want to use ClamAV as a mail filter if you host an email server.

Additionally, if you are a running modern versions of Linux where the FANOTIFY kernel feature is enabled, `clamd` has a feature run with On-Access Scanning*. *When properly configured*, On-Access Scanning can scan files as they are accessed and optionally block access to the file in the event that a signature alerted.

  _Note_: At this time, for On-Access Scanning to work, `clamd` must run with `sudo`/root privileges. For more details, please see our documentation on On-Access Scanning.

1. Copy the sample config. You may need to use `sudo`:
    <pre>
        cp /usr/local/etc/clamd.conf.sample /usr/local/etc/clamd.conf
    </pre>
2. Modify the config file using your favourite text editor. Again, you may need to use `sudo`.
    * At a minimum, remove the `Example` line so `freshclam` can use the config.
    * You also _need_ to select a Socket option for `clamd` so `clamdscan` and other utilities can communicate with `clamd`. You must enable _one_ of the following.
        * `LocalSocket`
        * `TCPSocket`

    Take the time to look through the options. You can enable the sample options by deleting the `#` comment characters.

    Some popular options to enable include:

    * `LogTime`
    * `LogClean`
    * `LogRotate`
    * `User`
    * `ScanOnAccess`
        * `OnAccessIncludePath`
        * `OnAccessExcludePath`
        * `OnAccessPrevention`

---

### Configure SELinux for ClamAV

Certain distributions (notably RedHat variants) when operating with SELinux enabled use the non-standard `antivirus_can_scan_system` SELinux option instead of `clamd_can_scan_system`.

At this time, libclamav only sets the `clamd_can_scan_system` option, so you may need to manually enable `antivirus_can_scan_system`. If you don't perform this step, freshclam will log something like this when it tests the newly downloaded signature databases:

<pre>
    During database load : LibClamAV Warning: RWX mapping denied: Can't allocate RWX Memory: Permission denied
</pre>

To allow ClamAV to operate under SELinux, run the following:
<pre>
    setsebool -P antivirus_can_scan_system 1
</pre>

---

### Download / Update the signature database

Before you can run a scan, you'll need to download the signature databases. Once again, you may need to run with `sudo`/root privileges.

If you installed to a location in your system PATH:
<pre>
    freshclam
</pre>

If you installed to another location:
<pre>
    /{path}/{to}/{clamav}/bin/freshclam
</pre>

  _Important_: It is common on Ubuntu after a fresh install to see the following error the first time you use ClamAV:
  <pre>
    $ freshclam
    freshclam: error while loading shared libraries: libclamav.so.7: cannot open shared object   file: No such file or directory
  </pre>

  You can fix this error by using ldconfig to rebuild the library search path.
  <pre>
    sudo ldconfig
  </pre>

---

### Users and on user privileges

If you are running `freshclam` and `clamd` as root or with `sudo`, and you did not explicitely configure with `--disable-clamav`, you will want to ensure that the `DatabaseOwner` user specified in `freshclam.conf` owns the database directory so it can download signature udpates.

The user that `clamd`, `clamdscan`, and `clamscan` run as may be the same user, but if it isn't -- it merely needs _read_ access to the database directory.

If you choose to use the default `clamav` user to run `freshclam` and `clamd`, you'll need to create the clamav group and the clamav user account the first time you install ClamAV.

<pre>
    groupadd clamav
    useradd -g clamav -s /bin/false -c "Clam Antivirus" clamav
</pre>

Finally, you will want to set user ownership of the database directory.
For example:
<pre>
    sudo chown -R clamav:clamav /usr/local/share/clamav
</pre>

---

## Usage

You should be all set up to run scans.

Take a look at our [usage documentation](https://www.clamav.net/documents/usage) to learn about how to use ClamAV each of the utilities.
