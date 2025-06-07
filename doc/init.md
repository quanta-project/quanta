Sample init scripts and service configuration for quantad
==========================================================

Sample scripts and configuration files for systemd, Upstart and OpenRC
can be found in the contrib/init folder.

    contrib/init/bitcoind.service:    systemd service unit configuration
    contrib/init/bitcoind.openrc:     OpenRC compatible SysV style init script
    contrib/init/bitcoind.openrcconf: OpenRC conf.d file
    contrib/init/bitcoind.conf:       Upstart service configuration file
    contrib/init/bitcoind.init:       CentOS compatible SysV style init script

Service User
---------------------------------

All three Linux startup configurations assume the existence of a "quanta" user
and group.  They must be created before attempting to use these scripts.
The macOS configuration assumes quanta will be set up for the current user.

Configuration
---------------------------------

Running quantad as a daemon does not require any manual configuration. You may
set the `rpcauth` setting in the `quanta.conf` configuration file to override
the default behaviour of using a special cookie for authentication.

This password does not have to be remembered or typed as it is mostly used
as a fixed token that quantad and client programs read from the configuration
file, however it is recommended that a strong and secure password be used
as this password is security critical to securing the wallet should the
wallet be enabled.

If quantad is run with the "-server" flag (set by default), and no rpcpassword is set,
it will use a special cookie file for authentication. The cookie is generated with random
content when the daemon starts, and deleted when it exits. Read access to this file
controls who can access it through RPC.

By default the cookie is stored in the data directory, but its location can be
overridden with the option `-rpccookiefile`. Default file permissions for the
cookie are "owner" (i.e. user read/writeable) via default application-wide file
umask of `0077`, but these can be overridden with the `-rpccookieperms` option.

This allows for running quantad without having to do any manual configuration.

`conf`, `pid`, and `wallet` accept relative paths which are interpreted as
relative to the data directory. `wallet` *only* supports relative paths.

To generate an example configuration file that describes the configuration settings,
see [contrib/devtools/README.md](../contrib/devtools/README.md#gen-bitcoin-confsh).

Paths
---------------------------------

### Linux

All three configurations assume several paths that might need to be adjusted.

    Binary:              /usr/bin/quantad
    Configuration file:  /etc/quanta/quanta.conf
    Data directory:      /var/lib/quantad
    PID file:            /var/run/quantad/quantad.pid (OpenRC and Upstart) or
                         /run/quantad/quantad.pid (systemd)
    Lock file:           /var/lock/subsys/quantad (CentOS)

The PID directory (if applicable) and data directory should both be owned by the
quanta user and group. It is advised for security reasons to make the
configuration file and data directory only readable by the quanta user and
group. Access to quanta-cli and other quantad rpc clients can then be
controlled by group membership.

NOTE: When using the systemd .service file, the creation of the aforementioned
directories and the setting of their permissions is automatically handled by
systemd. Directories are given a permission of 710, giving the quanta group
access to files under it _if_ the files themselves give permission to the
quanta group to do so. This does not allow
for the listing of files under the directory.

NOTE: It is not currently possible to override `datadir` in
`/etc/quanta/quanta.conf` with the current systemd, OpenRC, and Upstart init
files out-of-the-box. This is because the command line options specified in the
init files take precedence over the configurations in
`/etc/quanta/quanta.conf`. However, some init systems have their own
configuration mechanisms that would allow for overriding the command line
options specified in the init files (e.g. setting `QUANTAD_DATADIR` for
OpenRC).

### macOS

    Binary:              /usr/local/bin/quantad
    Configuration file:  ~/Library/Application Support/Quanta/quanta.conf
    Data directory:      ~/Library/Application Support/Quanta
    Lock file:           ~/Library/Application Support/Quanta/.lock

Installing Service Configuration
-----------------------------------

### systemd

Installing this .service file consists of just copying it to
/usr/lib/systemd/system directory, followed by the command
`systemctl daemon-reload` in order to update running systemd configuration.

To test, run `systemctl start quantad` and to enable for system startup run
`systemctl enable quantad`

NOTE: When installing for systemd in Debian/Ubuntu the .service file needs to be copied to the /lib/systemd/system directory instead.

### OpenRC

Rename quantad.openrc to quantad and drop it in /etc/init.d.  Double
check ownership and permissions and make it executable.  Test it with
`/etc/init.d/quantad start` and configure it to run on startup with
`rc-update add quantad`

### Upstart (for Debian/Ubuntu based distributions)

Upstart is the default init system for Debian/Ubuntu versions older than 15.04. If you are using version 15.04 or newer and haven't manually configured upstart you should follow the systemd instructions instead.

Drop quantad.conf in /etc/init.  Test by running `service quantad start`
it will automatically start on reboot.

NOTE: This script is incompatible with CentOS 5 and Amazon Linux 2014 as they
use old versions of Upstart and do not supply the start-stop-daemon utility.

### CentOS

Copy quantad.init to /etc/init.d/quantad. Test by running `service quantad start`.

Using this script, you can adjust the path and flags to the quantad program by
setting the BITCOIND and FLAGS environment variables in the file
/etc/sysconfig/quantad. You can also use the DAEMONOPTS environment variable here.

### macOS

Copy org.quanta.quantad.plist into ~/Library/LaunchAgents. Load the launch agent by
running `launchctl load ~/Library/LaunchAgents/org.quanta.quantad.plist`.

This Launch Agent will cause quantad to start whenever the user logs in.

NOTE: This approach is intended for those wanting to run quantad as the current user.
You will need to modify org.quanta.quantad.plist if you intend to use it as a
Launch Daemon with a dedicated quanta user.

Auto-respawn
-----------------------------------

Auto respawning is currently only configured for Upstart and systemd.
Reasonable defaults have been chosen but YMMV.
