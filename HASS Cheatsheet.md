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
I am using the default AIO username/password, replace them with yours

1. You can remove a topic from Mosquitto using `mosquitto_pub -r -n -u 'pi' -P 'raspberry' -t 'owntracks/arsaboo/mqttrpi'`
2. To subscribe to all the topics use `mosquitto_sub -h 192.168.2.199 -u 'pi' -P 'raspberry' -v -t '#'` (replace the IP address)
3. To publish use `mosquitto_pub -u 'pi' -P 'raspberry' -t 'smartthings/Driveway/switch'  -m 'on'` (use the relevant topic)

# HASS operations
1. To check realtime logs `sudo journalctl -fu home-assistant.service`
2. To restart HA `sudo systemctl restart home-assistant.service`
3. To check logs `sudo systemctl status -l home-assistant.service`
4. To stop HA `sudo systemctl stop home-assistant.service`
5. To start HA `sudo systemctl start home-assistant.service`

# Backing up Configurations on Github
Thanks to @dale3h for assistance with these instructions.

1. Install `git` using `sudo apt-get install git`
2. Go to https://github.com/new and create a new repository. I named mine `homeassistant-config`. Initialize with `readme: no` and `.gitignore: none`.
3. Navigate to your `.homeassistant` directory. For AIO, it should be `/home/hass/.homeassistant`, and for HASSbian, it is `/home/homeassistant/.homeassistant`.
4. Run `sudo su -s /bin/bash hass` for AIO and `sudo su -s /bin/bash homeassistant` for HASSbian.
5. Run `wget https://raw.githubusercontent.com/arsaboo/homeassistant-config/master/.gitignore` to get the `.gitignore` file from your repo (replace the link to match your repository). You can add things to your `.gitignore` file that you do not want to be uploaded.
6. Next, we need to add SSH keys to your Github account.
    * Navigate to `cd /home/hass/.ssh` (for AIO). If you don't have `.ssh` directory, create one and change the permission `chmod 700 ~/.ssh`.
    * Run `ssh-keygen -t rsa -b 4096 -C "homeassistant@pi"`. If you want to enter a passphrase, that's up to you. If you do, you'll have to enter that passphrase any time you want to update your changes to github. If you do not want a passphrase, leave it blank and just hit `Enter`.
    * Save the key in the default location (press `Enter` when it prompts for location).
    * When you're finished, run `ls -al ~/.ssh` to confirm that you have both `id_rsa` and `id_rsa.pub` files.
    * Go to https://github.com/settings/keys and click `New SSH key` button at top right. Title: `homeassistant@pi` (or whatever you want, really...it's just for you to know which key it is)
    * Run `cat id_rsa.pub` in the SSH session and copy/paste the output to that github page.
    * Then click `Add SSH key` button.
7. Go back to your repo page on GitHub. It'll be something like https://github.com/yourusernamehere/homeassistant-config. Click the green `Clone or download` button, and then click `Use SSH`.
8. You should see something like this in the textbox: `git@github.com:yourusername/homeassistant-config.git`. Copy that to your clipboard.
9. Now you are ready to upload the files to GitHub.
    * Navigate to `cd ~/.homeassistant`
    * `git init`
    * `git add .`
    * `git commit -m 'initial commit'`
    * If you get an error about `*** Please tell me who you are.`, run `git config --global user.email "your@email.here"` and `git config --global user.name "Your Name"`
    * After that commit succeeds, run: `git remote add origin git@github.com:yourusername/homeassistant-config.git` (make sure you enter the correct repo URL here)
    * Just to confirm everything is right, run `git remote -v` and you should see:
```
hass@raspberrypi:~/.homeassistant$ git remote -v
origin  git@github.com:arsaboo/homeassistant-config.git (fetch)
origin  git@github.com:arsaboo/homeassistant-config.git (push)
```
    * Finally, run `git push origin master`.

10. For subsequent updates:
    * `cd /home/hass/.homeassistant`
    * `sudo su -s /bin/bash hass`
    * `git add .`
    * `git commit -m 'your commit message'`
    * `git push origin master`

# Integrating HASS (AIO) with Smartthings using Mosquitto
If you are using AIO (which has Mosquitto pre-installed), you can use the following to integrate Smartthings and HA.

1. Install node.js, and pm2
```
    sudo apt-get update
    sudo apt-get upgrade
    curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
    sudo apt-get install -y nodejs
    sudo npm install -g pm2
    sudo su -c "env PATH=$PATH:/usr/local/bin pm2 startup systemd -u pi --hp /user/pi"
```
2. Install the [SmartThings MQTT Bridge](https://github.com/stjohnjohnson/smartthings-mqtt-bridge)
    ```
    $ sudo npm install -g smartthings-mqtt-bridge
    ```
3. Add details of your Mosquitto to `config.yml`. For the default AIO username password, your file should look something like:

    ```
    ---
    mqtt:
        # Specify your MQTT Broker's hostname or IP address here
        host: mqtt://192.168.2.199
        # Preface for the topics $PREFACE/$DEVICE_NAME/$PROPERTY
        preface: smartthings

        # Suffix for the state topics $PREFACE/$DEVICE_NAME/$PROPERTY/$STATE_SUFFIX
        # state_suffix: state
        # Suffix for the command topics $PREFACE/$DEVICE_NAME/$PROPERTY/$COMMAND_SUFFIX
        # command_suffix: cmd

        # Other optional settings from https://www.npmjs.com/package/mqtt#mqttclientstreambuilder-options
        username: pi
        password: raspberry

    # Port number to listen on
    port: 8080
    ```
4. Start ST-MQTT bridge `pm2 start smartthings-mqtt-bridge`
5. Follow the rest of the instructions (from step 2) listed [here](https://github.com/stjohnjohnson/smartthings-mqtt-bridge#usage).

# To upgrade the All-In-One setup manually (using this as I am using the older version of AIO):

*  Login to Raspberry Pi `ssh pi@your_raspberry_pi_ip`
*  Change to homeassistant user `sudo su -s /bin/bash hass`
*  Change to virtual enviroment `source /srv/hass/hass_venv/bin/activate`
*  Update HA `pip3 install --upgrade homeassistant`
*  Type `exit` to logout the hass user and return to the `pi` user.
*  Restart the Home-Assistant Service `sudo systemctl restart home-assistant.service`
