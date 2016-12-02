# If RPi is not syncing the time
Use the `systemd-timesyncd` service as explained [here](https://wiki.archlinux.org/index.php/systemd-timesyncd)
Here are the [time servers](https://wiki.archlinux.org/index.php/Network_Time_Protocol_daemon#Connection_to_NTP_servers) that I used.
```
NTP=0.north-america.pool.ntp.org 1.north-america.pool.ntp.org 2.north-america.pool.ntp.org 3.north-america.pool.ntp.org
FallbackNTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
```

# Configuring WinSCP to edit HASS files
Default AIO installation does not allow editing the configuration files in WinSCP. To enable the same, you will need to change the SFTP server (in Advanced settings -> Environment -> SFTP).

1. Obtain the SFTP by running `grep sftp /etc/ssh/sshd_config` at the `pi` shell. You will get something like, `Subsystem sftp /usr/lib/openssh/sftp-server`.
2. Now, set the sftp server to: `sudo su -c /usr/lib/openssh/sftp-server` (note that the `/usr/lib/openssh/sftp-server` is the same path obtained from the previous step) and set Shell to `sudo -s`.

# Mosquitto operations
I am using the default AIO username/password, replace them with yours.
1. You can remove a topic from Mosquitto using `mosquitto_pub -r -n -u 'pi' -P 'raspberry' -t 'owntracks/arsaboo/mqttrpi'`
2. To subscribe to all the topics use `mosquitto_sub -h 192.168.2.199 -u 'pi' -P 'raspberry' -v -t '#'`
3. To publish use `mosquitto_pub -u 'pi' -P 'raspberry' -t 'smartthings/Driveway/switch'  -m 'on'`

# HASS operations
1. To check realtime logs `sudo journalctl -fu home-assistant.service`
2. To restart HA `sudo systemctl restart home-assistant.service`
3. To check logs `sudo systemctl status -l home-assistant.service`
4. To stop HA `sudo systemctl stop home-assistant.service`
5. To start HA `sudo systemctl start home-assistant.service`

# Backing up Configurations on Github
TODO - Add instructions for linking HASS and Github.

1. `cd /home/hass/.homeassistant`
2. `sudo su -s /bin/bash hass`
3. `git add .`
4. `git commit -m 'your commit message'`
5. `git push origin master`
