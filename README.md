# Shuriken

A simple `bash`-based throwing framework.

It's stupid simple, and configuration is just done by flat files.

## Adding an exploit

Each exploit belongs in its own directory under `exploits`.  Symlinks work nicely if moving the files isn't ideal.

Each exploit directory must contain a file named `exploit`.  If this file is executable (`chmod +x`), it is invoked each round.  It is not invoked with any arguments.

The arguments to the exploit are passed in the environment.  The most important ones are:

- `TARGET_HOST` - IP or hostname to exploit
- `TEAM_NAME` - Pretty name of the team being exploited
- `FLAG_HOST` - IP to send flags to
- `FLAG_PORT` - Port to send flags to
- `FLAG_FILE` - File to write flags to (instead of IP:port)

All data from each invokation of the exploit is logged into the `logs` directory inside the exploit directory.  It is automatically created if it does not exist.

### Blacklists and Whitelists

By default, an exploit is thrown against all teams every round.

To modify this behavior, create a file named `whitelist` or `blacklist` in your exploit directory.  Any IPs or team names in `blacklist` are skipped.  If `whitelist` exists, any IPs or teams not contained in the file are skipped.

## Logging

### Exploitation Attempt Logs

All of the exploitation attempts are logged to stdout, as well as syslog.

See `config/log` for an extensible location to add to the logging.

To set up the syslog endpoint:

```
cat >> /etc/syslog.conf <<EOF
# Shuriken logs
local3.* /var/log/shuriken
EOF
touch /var/log/shuriken
```

### Per-Exploit Logs

Each exploit attempt gets its own log directory, in the `logs` directory.  Each log directory is timestamped, and includes the team name and target IP.  For example, it might look something like: `exploits/example_slowpoke/logs/2015-07-01-02:37:58-samurai-104.236.67.106-94230`.

Inside of the log directory are a handful of files:

- `stdout` - All data written over stdout
- `stderr` - All data written over stderr
- `status` - How the exploit exited (normally, terminated, etc)
- `run` - Shell script used to invoke the exploit

To see the full list of environment variables, look at the `run` file generated for each execution.

## Configuration

Shuriken does not require any command line arguments, all configuration comes from the files in the `config/` directory.  By default, no configuration should be necessary.

In addition to those documented above, there are also:

- `before` - Function which is invoked before every exploit
- `after`  - Function which is invoked after every exploit
- `log`    - Function used by Shuriken to do its logging
- `get_log_dir` - Emits the location where the current exploit should log to

## Github Integration

By default, at the beginning of each round, the repository is updated from GitHub.  This means that you can develop changes on your laptop, push the changes to GitHub, and they will be automatically synced on the throwing server.

**For the scrimmage**, it is important that you [fork your own repo][fork], and clone your fork rather than the original repo.  Otherwise your changes won't show up.

To disable this behavior, delete or comment out `config/00-github`.

[fork]: https://help.github.com/articles/fork-a-repo/

## Listeners

Some example key listeners are included in `listeners/`.  These are designed as testing tools in lieu of a full-blown Nodachi key listener.

- `tcp` receives flags via TCP
- `fifo` receives flags from a FIFO

Example:

```
./listeners/tcp &
./shuriken > /dev/null
```
