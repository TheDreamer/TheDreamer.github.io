---
title: ZFS and tuning Postgresql
layout: post
comments: yes
category: other
---

For the umpteenth time, I found myself looking over mysql parameters, reading documentation and fiddling with more settings.
Whereas my PostgreSQL setup, I have done very little in the way of fiddling with its configuration.  Not sure if that says
anything about one versus the other.

With mysql, I've been using a combination of `mysqltuner`, `tuning-primer.sh`, `phpMyAdmin`, and Percona monitoring
plugins for cacti, along with various online sites and the online manuals to decide what to fiddle with.  I currently have
3 mysql-server instances:

| Host 	| Processor    	| Clock   | Cores | Threads     | RAM  	| Disk 	| Databases                                    	|
|------	|--------------	|-----:   |-----: |-------:     |-----:	|-----:	|----------------------------------------------	|
| cbox 	| Atom D2700   	| 2.13GHz | 2     | 4           |  4GB 	|  SSD 	| cacti                                        	|
| zen  	| Core i7-870  	| 2.93GHz | 4     | 8           | 16GB 	|  HDD 	| phlymail, roundcube, owncloud, phpMyAdmin    	|
| mew  	| Core i7-2600 	| 3.40GHz | 4     | 8           | 16GB 	|  HDD 	| cacti, phlymail, phpMyAdmin, pydio, roundcube	|

Not sure which mysql-server instance draws the most attention.  At first it was cbox, since its maxed on memory and does
lots of services for my network, with cacti as a secondary service on this headless server.  Which is also busy with other
secondary services since I made it my primary (dynamic) domain, since its associated with my cable modem service.  While
its counterpart handles my secondary (dynamic) domain, since that is my dsl modem service.  Both machines are dual homed,
thought the primary gateway is the associated service, and just about everything can be accessed through either domain.

Though it wasn't always the case, as I'm not running a hacked router with cable modem service at the moment.  So, it has
a rather limited number of port forwards.  Though due to attrition, I don't need as many forwards as I used to...

Anyways, originally, cbox, when it was first installed as FreeBSD 9.1, it was running a fairly constant load of ~15.  There
would be higher periods and lots of problems at midnight due to log rotations, and other periodic tasks.  Later the load
became a fairly constant ~11.5....  I largely attribute this to [upgrading to FreeBSD 9.2, and] switching to `lz4`
compression.

There have been other tweaks here and there to try to have cacti keep up while giving the system a serious workout.
Eventually, I caved and reset my cacti install with a 5 minute poll time instead of 1 minute.  This puts the load at ~2
most of the time.  dbox's load is ~1.5.  dbox is my nagios server, and it is where I leave irssi running.  Someday I hope
to figure out CARP and HAST, and get to setting my my own git repo and migrate my CFEngine configuration from SVN to GIT.
Though the nested GIT directories in SVN might pose a challenge.

So, after reconciling the latest set of recommendations from the various mysql tuning tools, some contradictory, others
that seem to make no sense.  Though tuning-primer warns it advise increasing something, even though the nature of the
table doesn't use the buffers.  Where as `mysqltuner` will always advise making certain parameters bigger than current,
on the basis that its utilization is low.  Not that its low, because that's all there is for it.  Like `key_bufffer_size`,
my MyISAM index space is 115K, `key_buffer_size` defaults to 8MB.  Because its utilization is so low, `mysqltuner` will
want me to increase this.  Same with `table_open_cache`, table cache hit rate is low, so must increase.  `tuning-primer.sh`,
notes that I have a total of X tables, and currently have Y tables open...making the parameter fine.

Meanwhile, I'm also contending against constraints, none of these systems are dedicated for mysql.  Far from it, so I need
to keep a reign on memory.  For the filesystems, they are currently have local settings for `recordsize = 16K`,
`compression = lz4`, `logbias = throughput` and depending on system either local or inherited setting of
`primarycache = metadata` and an inherited setting of `atime = off`.  cbox also has a quota/reservation since space is a
premium with it only having a 120GB SSD.

So, today I turned my attention to PostgreSQL, I only have one instance (currently), and it is on zen.  It currently only
has one active database, `delta_reporting`, and am not aware of any performance issues.  Though I was looking at some other
open source products, and I'm considering one likely very intensive app use PostgreSQL as its back-end rather than mysql.

Well, `pgtune` just picks a set of numbers based on what it would get used for and available memory.  The initial result was
based on it being able to use all my memory.  It didn't work, because my shared memory limit setting wasn't high enough for
it.  I raised the limit, but left it for a future reboot, because I decided that it was rather excessive given all the other
things on my system.  So, I reran it for a lower amount of memory.  Its possibly still too large as I keep doing more and
more on zen.

Given that, there didn't seem any point to running `pgtune` today, since its not going to give me any different answers.
But, I got curious about other parameters in the configuration file.  Since there are a number of recommended configuration
changes associated with ZFS, like `innodb_doublewrite=FALSE` and `innodb_flush_method=O_DIRECT`.  Or SSD, for 5.6.3+,
`innodb_flush_neighbors=0`  And, then I spotted `innodb_io_capacity`.  Used to be fixed at 100 IOPS, but now that it is
configurable it defaults 200, with 100 being the minimum.

Since I added collection of data from gstat and zpool iostat for cacti, I have graphs that give me an idea of IOPS I've
been getting with the various zpools.  For the HDD based ones, the numbers seem in line with what I found online.  So, I
tweaked this parameter to see what kind of trouble I might get into.  For both mew and zen, I set down to 150.  On zen,
the average for write ops/s is 135 and read ops/s is 196.  Though there are peaks over 600.  Should probably go 100 with
zen, since I had gone from 7200rpm disks to 5400rpm disks recently, though the new drives feel faster.  Probably being of
higher density, or having more cache.  Now mew, the average write ops/s is currently 156, with read ops/s of 122.  I had
felt that these drives were slower on read performance.  And, I guess I have some data to prove it now.  This is probably
due to mew using a pair of WD Purple...which are advertised as being write optimized, among other things.  Versus the WD
Red's that are in zen.  Had gone with Purple with mew, at work, because at the time Purple was slightly cheaper than Red.

And, there hadn't been any reviews of these drives in non-video settings.  Where in the past there had been recommendation
to use other video type drives (since they're also designed to be on 24x7, and have TLER.)  Though I suppose the distinction
is Purple is geared towards video surveillance systems as opposed to DVRs.

So, getting Purple drives at home is now off my wishlist.... So, now its like between 5 WD Red 3TB drives or 5 HGST Deskstar
NAS 4TB drives.  Neither are in the sub-$100 threshold I used to use (now its more like ~$600 total -- to tap into a 12-month
special financing offer, or sub-$200...), and then its the question of whether raidz or raidz2 with these drives?  With 3TB
drives, raidz is ~10.5TB (~9.5TB @ 90%) vs raidz2 of ~8TB (~7.25TB @ 90%)....as upgrade to my current array (6 2TB HGST
raidz2) of ~7.15TB (~6.4TB @ 90%)....the 90% mark of a zpool is kind of a serious limitation.  The zpool is pretty much
unusable when this point is crossed.  Though with my BackupPC pool, it seems to be worse....might need to keep more than 10%
free to keep it usable.  Its probably gotten very badly fragmented given the nature of its use.

The current array is 6 drives, putting one drive in another enclosure, because originally the 6 drives were set up in
RAID 10 -- mirrored across SATA channels.  There were 8 drives, had thought I could add two more later when needed, though
it would be concat a RAID 1.  So, I used it as a separate RAID 1 until I couldn't....

But, on the zen, it seems the hold times of the power supplies aren't quite matched so if there's a dropout, they might get
into a state where ZFS will stop the zpool.  Things usually fine when I reboot, though sometimes it takes a really long
time to import....hours.  Though this has also happened with smaller mirrored pools.  Just depends on what was happening
when things crashed.  In addition to wanting somebody to write a defrag for ZFS, an undedup tool would be next.  Dedup was
interesting, but dedup with snapshots was a nightmare.  Especially, when trying to destroy....

So back to PostgreSQL tuning...  in looking at the config file, I find `#wal_sync_method = fsync`, notes that default is
first option available on the OS in 'list'.  Which as I found out, are `'fsync'` or `'open_sync'`.  In reading the
documentation, I find that `'O_DIRECT'` applies to the `'open_*'` methods.  Wonder if that's something I want to set with
PostgreSQL?

Meanwhile, I spot a line about turning `full_page_writes` off, where on helps recover from partial page writes, for
filesystems that prevent this from happening, such as ZFS.  So, I did that...not sure if it buys me anything.  Later I
read about `pg_test_fsync`, so I played around with it a little bit, which then turned into a lot.  As I'm thinking of
moving it from being HDD based on zen to being SSD based.  And, I was curious about the impact of the ZFS settings.
Namely `recordsize` and `compression`.

So, I run a test, with 30 seconds per test, on my current directory (8k, no compression).... ~7 ops/sec !!!  Tried a few
more times, ~5.7 ops/sec.... While, non-sync'd is pretty snappy... I decided to compare it against where mysql is.  Its ~39
ops/sec, an improvement, but still pretty bad.  Expected way more, but then I realized its on the same disk as PostgreSQL.
I hadn't gotten around to move it to SSD yet.  But, that filesystem is set to 16K and lz4 compression.  Which one of those
helps PostgreSQL?

But, I decided I want to see how its going to behave on SSD, so I create some test filesystems and give it a whirl.
The first I ran it, I had accidentally run it from root's home directory so its default 128K recordsize and no compression,
etc.

|                       | 128k - no    | 4k - no       | 4k - lz4      | 8k - no       | 8k - lz4      | 16k - no      | 16k - lz4     | HDD 16k - lz4         | HDD 8k - no   |
|:--------------------  |----------:   |--------:      |---------:     |--------:      |---------:     |---------:     |----------:    |--------------:        |------------:  |
| fsync - 1-8k          |    194.464   |   997.751     |   965.816     |  1032.509     |  1010.735     | 889.180       | 862.768       | 39.597                | 7.382         |
| o_sync - 1-8k         |    245.277   |  1004.549     |   982.133     |  1038.875     |  1020.275     | 895.012       | 852.499       | 34.937                | 7.852         |
| fsync - 2-8k          |    247.556   |   838.560     |   811.667     |   860.002     |   832.615     | 857.939       | 901.253       | 31.858                | 5.563         |
| o_sync - 2-8k         |    123.775   |   498.438     |   484.799     |   512.284     |   489.692     | 407.187       | 469.617       | 11.724                | 3.897         |
| o_sync - 1 x 16k      |    248.956   |   824.287     |   807.955     |   829.164     |   832.371     | 809.979       | 858.716       | 41.391                | 10.195        |
| o_sync - 2 x 8k       |    117.872   |   497.765     |   484.520     |   520.424     |   501.406     | 436.367       | 473.452       | 13.625                | 5.291         |
| o_sync - 4 x 4k       |     53.413   |   290.826     |   281.401     |   256.518     |   250.409     | 196.668       | 232.047       | 5.468                 | 1.913         |
| o_sync - 8 x 2k       |     29.901   |   146.819     |   140.398     |   128.882     |   143.607     | 93.979        | 138.307       | 2.983                 | 0.762         |
| o_sync - 16 x 1k      |     15.361   |    73.878     |    68.127     |    64.528     |    71.880     | 50.755        | 70.569        | 1.885                 | 0.550         |
| write, fsync, close   |    175.054   |   981.875     |   949.500     |  1011.344     |   973.813     | 848.228       | 915.618       | 19.034                | 5.608         |
| write, close, fsync   |    249.723   |   973.339     |   945.572     |   957.554     |   983.589     | 836.668       | 812.607       | 7.391                 | 4.152         |
| non-sync'd 8k writes  |  80076.085   | 56245.367     | 56170.767     | 82207.834     | 81772.752     | 77844.273     | 81344.533     | 67339.440             | 61768.941     |

Wonder if the data used by `pg_test_fsync` is compressible, guessing no.

But, looks like "8k - no" on SSD is the way to go with PostgreSQL.  Wonder if there's a similar test for mysql?  The HDD
stats might be distorted due to having previously done dedup on the volume or that there were backups in progress against
it.  Wonder how would I test to see if compression is of benefit on SSD or not, for real data?  Or call the differences
insignificant and go with compression anyways.

Wonder if the HDD case, if logbias is working against me.  Since my HDD pools have SLOGs, or if I should change
secondarycache setting also?  The HDD pools have 72G of L2ARC.

Now to find time to make the migration of some filesystems to the SSD.  This has come about, because I have upgraded zen
from 120GB SSDs to 256GB SSDs.  Though wonder if I'll outgrow the SSD due to moving these filesystems soon afterwards....

_this post seems to be marking the end for my main blogging platform, wonder how I can integrate or migrate?_
