# SSH UFW Bootstrap Manager

An interactive Bash utility that scans an IPv4 range for SSH services and runs
[ufw-bootstrap-kit](https://github.com/Michdo93/ufw-bootstrap-kit) on the selected devices: on each
device the ports of the running services are allowed (SSH first) and UFW is enabled afterwards.
It works like [ssh-dns-manager](https://github.com/Michdo93/ssh-dns-manager): range → scan →
confirm each device → user name → up to three login attempts.

## Requirements

- Bash 4 or newer, OpenSSH client (`ssh`), `timeout`, `xargs`, `base64` (Linux, WSL).
- Targets: Debian, Ubuntu, Raspberry Pi OS or DietPi with an SSH server and Internet access.
- A remote account that is root or may use `sudo` (password typed into the remote prompt).
- **All services of a device must be running** when it is processed – their ports are what stays open.

## Run

```bash
chmod +x deploy-ufw.sh
./deploy-ufw.sh
```

1. **Settings (once):**
   - additional ports for all devices, e.g. `80/tcp,443,7400:7649/udp` (Enter for none)
   - allow all detected ports without asking (default **no**: you see the port list of every
     device and confirm it or decide per port)
   - dry run (only show the rules)
2. **Range** and **scan** as in the other SSH tools (32 parallel probes, no ping needed).
3. **Per device:** confirm, SSH user name, up to three login attempts. The kit shows the detected
   ports, e.g.

   ```
   PORT           PROTO  LISTENING ON               PROCESS
   22             tcp    *                          ssh  (always allowed)
   7400:7649      udp    *                          DDS/ROS 2 domain 0
   8080           tcp    0.0.0.0                    python3
   ```

   and enables UFW. SSH is always allowed before UFW is enabled, so the session is not cut off.

At the end the tool lists the successful and the failed devices.

## Options (environment)

| Variable | Default | Meaning |
|---|---|---|
| `SSH_PORT` | `22` | SSH port for scan and login |
| `CONNECT_TIMEOUT` | `4` | seconds per connection attempt |
| `SCAN_PARALLEL` | `32` | parallel probes while scanning |
| `INSTALLER_URL` | `install.sh` of ufw-bootstrap-kit (`main`) | other installer, e.g. a fork |
| `INSTALLER_ENV` | – | `VAR=value` words passed to the installer |

## How it works

The remote part is a short POSIX-sh script sent base64-encoded as the remote command of `ssh -tt`
(not on stdin): the remote terminal stays connected to your keyboard for the sudo password and the
kit's questions, and the script text is never echoed back. It downloads the kit with `wget` or
`curl` and runs it as the logged-in user; the kit uses sudo itself.

## License

MIT License. Feel free to use and modify.
