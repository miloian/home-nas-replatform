# Home NAS replatform Project

## Objective

This project is to replaform my home UnRAID server to a Debian 10 based server with Docker running any apps as needed in order to keep the main OS as clean as possible.

### Disk Partition layout

#### BTRFS Partition Setup:

| Disk | Partition | Array | Partition Size  | Mountpoint      | Mode  |
|------|-----------|-------|-----------------|-----------------|-------|
| 1    | 1         | 1     | 80GB            | /var/lib/docker | RAID1 |
| 2    | 1         | 1     | 80GB            | /var/lib/docker | RAID1 |
| 1    | 2         | 2     | 420GB           | /tank/ssd       | RAID1 |
| 2    | 2         | 2     | 420GB           | /tank/ssd       | RAID1 |
| 3    | 1         | 3     | 6TB             | /tank/rust      | JBOD  |
| 4    | 1         | 3     | 6TB             | /tank/rust      | JBOD  |
| 5    | 1         | 3     | 6TB             | /tank/rust      | JBOD  |
| 6    | 1         | 3     | 6TB             | /tank/rust      | JBOD  |

#### Subvolumes:

| Array | Subvolume            | Mountpoint      |
|-------|----------------------|-----------------|
| 2     | /tank/ssd/appdata    | /var/docker     |
| 3     | /tank/rust/movies    | /data/movies    |
| 3     | /tank/rust/tv        | /data/tv        |
| 3     | /tank/rust/misc      | /data/misc      |
| 3     | /tank/rust/downloads | /data/downloads |

## Initial setup

### Install

	apt-get install git sudo btrfs-progs vim -y

### Add user to sudo using root

	/sbin/adduser username sudo

## Volume prep

### Create Mount directory

	sudo mkdir -p /tank/{ssd,rust}

### Create BTRFS Arrays

	# Create Docker array with smaller partions from the SSD
	sudo mkfs.btrfs -L docker -m raid1 -d raid1 /dev/{diskX,diskY}
	
	# Create the general use Array using the larger SSD partitons
	sudo mkfs.btrfs -L ssd -m raid1 -d raid1 /dev/{diskX,diskY}
	
	# Create the JBOD Sotre
	mkfs.btrfs -L rust -d single /dev/mkfs.btrfs -d single /dev/{diskW,diskX,diskY,diskZ}

### Get BTRFS UUID

	sudo btrfs fi show

### Fstab Options for arrays

	sudo vi /etc/fstab

	UUID=UUID_FROM_ABOVE_COMMAND	/tank/ssd		btrfs	ssd,noatime,autodefrag 0 0
	#UUID=UUID_FROM_ABOVE_COMMAND	/var/lib/docker 	btrfs 	ssd,noatime,autodefrag 0 0
	UUID=UUID_FROM_ABOVE_COMMAND 	/tank/rust 		btrfs 	noatime,autodefrag	0 0

Note: Leave the docker entry commented out for now

Run:

	sudo mount -a

### Create subvolumes

	sudo btrfs subvolume create /tank/ssd/appdata
	sudo btrfs subvolume create /tank/rust/{movies,tv,misc,downloads}

### Create the Subvolume mountpoints

	sudo mkdir -p /data/{movies,tv,misc,downloads,appdata}

### Add Subvolume mounts to fstab

	sudo vi /etc/fstab


	LABEL=ssd 		/data/appdata 		subvol=/appdata,defaults,noatime 		0  0
	LABEL=rust 		/data/movies 		subvol=/movies,defaults,noatime 		0  0
	LABEL=rust 		/data/tv 		subvol=/tv,defaults,noatime 			0  0
	LABEL=rust 		/data/misc 		subvol=/misc,defaults,noatime 			0  0
	LABEL=rust 		/data/downloads 	subvol=/downloads,defaults,noatime 		0  0

## Software Install

### Install Docker

[Instructions here](https://phoenixnap.com/kb/how-to-install-docker-on-debian-10)

### Configure docker to use BTRFS

[Using BTRFS with Docker](https://docs.docker.com/storage/storagedriver/btrfs-driver/)

Note: For the mounting step in this article, remove the # from the docker mount we created earlier and run

	sudo mount -a

