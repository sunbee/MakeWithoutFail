# Make Without Fail
Build resilience in Raspberry Pi-based compute infrastructure for makers. Start with setting up Pi to operate off an SSD instead of the conventional SD card. The build resilience by containerizing apps with Docker, automated backups of application data to Cloud and disk imaging.

This note covers automated backups of application data to Cloud with **rclone** and **rsync**.

## Checklist

Step | Instruction
---- | -----------
YAML file for IOT stack | See Andreas Speiss and Graham Garner [here](https://sensorsiot.github.io/IOTstack/Getting-Started.html). The application data is mapped to a volumes folder on host machine.
Dropbox mount | Use rclone as shown [here](https://rclone.org/dropbox/).
Dropbox mount-point service | Use systemd as explained [here](https://www.jamescoyle.net/how-to/3116-rclone-systemd-startup-mount-script) and [here](https://wildestpixel.co.uk/systemctl-running-tasks-at-boot-on-a-raspberry-pi) and shown in **Exhibit A**.
Backup command | Use rsync as explained [here](https://www.linuxscrew.com/rsync) and shown in **Exhibit B**.
Backup shell script | Use bash script for backup job as explained [here](https://www.raspberrypi.org/forums/viewtopic.php?p=1710222) and shown in **Exhibit C**.
Backup automation on task scheduler | Use crontab as explained [here](https://www.factoryforward.com/autorun-python-on-raspberry-pi-code-using-crontab/) and shown in **Exhibit D**.

## Exhibit A: Create the `rclone.service` file in the home directory `/home/pi`.

```
# /etc/systemd/system/rclone.service
[Unit]
Description=Dropbox (rclone)
AssertPathIsDirectory=/mnt/dropbox
[Service]
Type=notify
ExecStart=/usr/bin/rclone mount \
    --config /home/pi/.config/rclone/rclone.conf \
    --allow-other \
    --vfs-cache-mode writes \
    --vfs-cache-max-size 100M \
    dropbox: /mnt/dropbox
ExecStop=/bin/fusermount -u /mnt/dropbox
```
1. Create in Nano using arrow-keys to navigate, Ctrl-K to cut lines, Ctrl-U to paste, Ctrl-O to save and Ctrl-X to quit.
2. The important specifications are: `--config` to point to the `rclone.conf` file and the mapping from `dropbox:` to the local `/mnt/dropbox`.
3. Place the file in the location: /etc/systemd/system. Use sudo cp to write to this destination.
4. Start with the following commands: 
	1. `systemctl start rclone`
	2. `systemctl enable rclone`
5. Use `ls -l` to verify that the Dropbox folders show up at the mount location. Note that the files may not show up in the File Explorer (GUI).
6. In case of any errors, proceed as follows:
	- Grab the error logs with the following command: `systemctl status rclone.service > err.txt`.
	- Fix the file in the home directory using Nano, then `sudo cp` to `/etc/systemd/system` as before.
	- Refresh: `systemctl daemon-reload`.

## Exhibit B: Backup with rsync.  

To backup, use the command as follows:
```
sudo rsync -avz --delete /home/pi/IOTstack/volumes /mnt/dropbox
```
To test, execute dry-run as follows:
```
sudo rsync -avz --delete --dry-run /home/pi/IOTstack/volumes /mnt/dropbox
```

## Exhibit C: Write bash (shell) script 'backup2dropbox.sh' in `/home/pi` for backup with rsync. Use Nano following guide [here](https://www.raspberrypi-spy.co.uk/2013/11/quick-guide-to-nano-text-editor-on-the-raspberry-pi/).
```
NOW="$(date)"
echo "Started backup at $NOW"
# sudo rsync -avz --delete --dry-run /home/pi/IOTstack/volumes /mnt/dropbox
sudo rsync -avz --delete /home/pi/IOTstack/volumes /mnt/dropbox
NOW="$(date)"
echo "Ended backup at $NOW"
```
- The flags are as follows:
	- `-a`: save metadata (e.g. ownership) while taking back-up.
	- `-v`: verbose
	- `-z`: compress
- Use `--delete` to clean up files at the destination that have been removed at the source. 
- Make the shell script executable with chmod as follows:
```
 chmod 755 /home/pi/backup2dropbox.sh
```
- Then test it as follows:
```
sh /home/pi/backup2dropbox.sh
```

## Exhibit D: Open up crontab as root. 
```
sudo crontab -e
```
Select Nano as the editor from options presented by the interactive shell. Then add the following line to run the shell script at **4:30 am** on the 1st and 15th day of every month and every Friday.
```
30 4 1,15 * 5 /home/pi/backup2dropbox.sh >> /home/pi/logs/cron.log 2>&1
```
Make sure to have a logs folder /home/pi/logs. The output will be appended to the log file instead of being sent to standard output. Errors will be sent to standard output. To backup daily, modify as follows:
```
30 4 * * * /home/pi/backup2dropbox.sh >> /home/pi/logs/cron.log 2>&1
```
