Backup with bittorrent sync using Resilio Sync
==============================================

Sync skips the cloud and finds the shortest path between devices when transferring data. No Cloud. No uploading to third party servers. Just fast, simple file syncing.

Based on my alpine micro container with glibc [nimmis/alpine-glibc](https://registry.hub.docker.com/u/nimmis/alpine-glibc/), size just under 26 Mb


## run container on first sync node

The first nod creates a uniq secret used to sync all nodes, map the directory you wan't to be syncronized to /backup in the container.

Example synk the directory /home/me/data on first nod giving it the name syncnode

	docker run -d -v /home/me/data:/backup --name syncnode nimmis/backup-btsync


to see the secret code to use on the other nodes, look at the log-output from the container

	> docker logs -f syncnode
	Run scripts in /etc/run_once
	add -e BTSYNC_SECRET=AF2INNKYP672IGIIDTDWWVUBGP2AQRFKX to your other nodes to sync
	Run scripts is /etc/run_always
	Started runsvdir, PID is 14
	wait for processes to start....
	rsyslogd: imklog: cannot open kernel log (/proc/kmsg): Operation not permitted.
	rsyslogd: activation of module imklog.so failed [v8.18.0 try http://www.rsyslog.com/e/2145 ]
	run: btsync: (pid 19) 5s
	run: crond: (pid 20) 5s
	run: rsyslogd: (pid 21) 5s

press CTRL-C to exit log

## run container on more sync nodes

To get the other nodes to sync with the first, they have to have the same secret key. 
You add the key with -e BTSYNC_SECRET=<secret key>, start with the secret code created 
by the first sync container. The local director does not have to be the same as on the 
first container but all sub-folders will be the same. So starting a second sync container
on another docker machine using local directory /home/you/sync-backup and using the secret
key obtained from the first sync node example above 

	docker run -d -v /home/you/sync-backup:/backup --name syncnode2 -e BTSYNC_SECRET=AF2INNKYP672IGIIDTDWWVUBGP2AQRFKX nimmis/backup-btsync

NOTE!!!!! DO NOT USE the secret key in this example, use the one you got from the first sync node run.

## nodes don't sync

If the nimmis/backup-btsync is started with another secret than the syncdirectory was first started with it will not start syncing,
execute the following command on the sync container to check

	docker exec  <syncname> grep MUTEX  /var/log/btsync.log

if you get a hit on something that look like

	[20160710 13:38:31.611] MUTEX[/data]: share id mismatch, ARGCQRK35Z3DH5DPNATQOCXREYS6ZYYJV != ATFQAUDKIQKHE4IHHHBV2EZFN4HNILPRK

The sync data is not valid for the current key, just stop the container and remove old sync information

	docker stop <syncname>
	sudo rm -Rf <sync folder>/.sync
        docker start <syncname>
