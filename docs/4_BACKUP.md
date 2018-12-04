# Backup

I use a shell script to create a .tar file. I'm using a slightly modified verison of this guide [here](https://help.ubuntu.com/lts/serverguide/backup-shellscripts.html)

I'm backing it up a NAS on my local network. 


`backup.sh`:
```
#!/bin/bash
####################################
#
# Backup to NFS mount script.
#
####################################

# What to backup. 
backup_files="/home /var/spool/mail /etc /root /boot /opt"

# Where to backup to.
dest="/mnt/backup"

# Create archive filename.
day=$(date -I)
hostname=$(hostname -s)
archive_file="$hostname-$day.tgz"

# Print start status message.
echo "Backing up $backup_files to $dest/$archive_file"
date
echo

# Backup the files using tar.
tar czf $dest/$archive_file $backup_files

# Print end status message.
echo
echo "Backup finished"
date

# Long listing of files in $dest to check file sizes.
ls -lh $dest
```

- `$backup_files`: a variable listing which directories you would like to backup. The list should be customized to fit your needs.
- `$day`: a variable holding the day of the week (Monday, Tuesday, Wednesday, etc). This is used to create an archive file for each day of the week, giving a backup history of seven days. There are other ways to accomplish this including using the date utility.
- `$hostname`: variable containing the short hostname of the system. Using the hostname in the archive filename gives you the option of placing daily archive files from multiple systems in the same directory.
- `$archive_file`: the full archive filename.
- `$dest`: destination of the archive file. The directory needs to be created and in this case mounted before executing the backup script. See Network File System (NFS) for details of using NFS.
- `status messages`: optional messages printed to the console using the echo utility.
- `tar czf $dest/$archive_file $backup_files`: the tar command used to create the archive file.
	- `c`: creates an archive.
	- `z`: filter the archive through the gzip utility compressing the archive.
	- `f`: output to an archive file. Otherwise the tar output will be sent to STDOUT.
- `ls -lh $dest`: optional statement prints a `-l` long listing in `-h` human readable format of the destination directory. This is useful for a quick file size check of the archive file. This check should not replace testing the archive file.


After you have created your `backup.sh` file, you'll need to make it executable by running

```
chmod u+x backup.sh
```

`+x` makes it executable, while `u+x` will only grant owner of that file  execution premissions.


## Backup frequently
I'm using crontab to execute the above script every day at 00:00

Run `sudo crontab -e` and add the following line to the bottom


Note: Using `sudo` with the `crontab -e` command edits the root user's crontab. This is necessary if you are backing up directories only the root user has access to.

```
0 0 * * * bash /backup.sh
```

Change out `/backup.sh` to wherever you saved that file.

Those zero's and asteriks dictates when that shell script should run. Use [crontab guru](https://crontab.guru/#0_0_*_*_*) if you want it to run at a different time.

## External backup
It's always good practise to save your backups several places. I'm using [rclone](https://rclone.org/) to create an encrypted version of my backup.tar file and sync it to Google Drive. I wont explain in detail, but I created a new shell script file:

```
LOGFILE="/opt/logs/rclone-upload.log"
##!/bin/bash
if pidof -o %PPID -x “rclone-cron.sh”; then
   exit 1
fi

start=$(date +'%s')
echo "$(date "+%d.%m.%Y %T") RCLONE UPLOAD STARTED" | tee -a $LOGFILE

rclone copy rclone copy source:sourcepath dest:destpath --log-file=$LOGFILE

echo "$(date "+%d.%m.%Y %T") RCLONE UPLOAD FINISHED IN $(($(date +'%s') - $start)) SECONDS" | tee -a $LOGFILE
exit

```
Change `source:sourcepat` and `dest:destpath`, then added it to crontab. 
