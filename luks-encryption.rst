Creating the encrypted device
=============================

This can be done on any linux based machine, but the commands below were initially run on Debian based distros. This assumes a blank storage device, rendering the need for a secure wipe unnecessary. Use ``lsblk`` to find the storage device (``/dev/mmcblk1`` is the location for example, an SD card). The device size listed in ``lsblk`` should be the expected size of the device. This also assumes there is only one existing partition on the device (the default partition, which is the case for SD cards).

Remove the partition::

 sudo parted -s /dev/mmcblk1 rm 1

Build the new partition
~~~~~~~~~~~~~~~~~~~~~~~~

This creates the primary partition for all available space on the device. For example ``/home/user``::

 sudo parted --align optimal /dev/mmcblk1 mkpart primary ext4 0% 100%

Encrypt the partitions
~~~~~~~~~~~~~~~~~~~~~~

The following will require user interaction with the command. Store passwords somewhere safe (a password manager)::

 sudo cryptsetup -v -y -c aes-xts-plain64 -s 512 -h sha512 -i 5000 --use-random luksFormat /dev/mmcblk1p1

Backup the header
~~~~~~~~~~~~~~~~~

This is so that the data can be recovered in case of corruption of the header due to hardware issues, or meddling with the formatting of the device::

 sudo cryptsetup luksHeaderBackup --header-backup-file /tmp/user_home_header.$(hostname).img /dev/mmcblk1p1

It's best to copy these headers to a separate machine.

Decrypt the drives to put a file system on it
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

 sudo cryptsetup open --type luks /dev/mmcblk1p1 user_home

Format the drive
~~~~~~~~~~~~~~~~

Note that the decrypted drive mapping is available in ``/dev/mapper/``::

 sudo mkfs.ext4 /dev/mapper/user_home

Close the encryption containers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

 sudo cryptsetup close user_home

The device is now ready to be used.

Migrate existing data
=====================

This assumes that you have another user to use besides the user who's home directory is being encrypted.

Kill all processes for the user
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

 sudo pkill -u user

Move the data that is to be encrypted
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

 cd /
 sudo mv /home/user /tmp/user.bkp

Recreate the mount points and add a notice file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
::

 sudo mkdir /home/user
 sudo touch /home/user/DECRYPT_DEVICE_BEFORE_CONTINUING

Add the decrypting commands to the notice file for ease of use::

 echo "sudo cryptsetup open --type luks /dev/mmcblk1p1 user_home" \
   > /home/user/DECRYPT_DEVICE_BEFORE_CONTINUING
 echo "sudo mount /dev/mapper/user_home /home/user" \
   >> /home/user/DECRYPT_DEVICE_BEFORE_CONTINUING

Unlock the encrypted device and mount it
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This assumes the parameters used above in "Creating the encrypted the device". Note that unlocking the home directory while currently visiting the home directory will require refreshing the home directory (just use `cd`)::

 sudo cryptsetup open --type luks /dev/mmcblk1p1 user_home
 sudo mount /dev/mapper/user_home /home/user

Copy data back
~~~~~~~~~~~~~~
::

 sudo rsync -au /tmp/user.bkp/ /home/user/

Ensure the data copy was correct
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once it is ensured that the data copy was done correctly, remove the backups::

 sudo rm -rf /tmp/user.bkp/

The device has now been migrated.
