---
title: using the F5configbackup appliance and selinux
-date:
-description:
layout: post
comments: yes
category: other
---

At work we got a pair of BIGip 7200v units, with the notion that we will be migrating from our BIGip 6400 pair in the
near future.  In the meantime, having a means to backup their configs is highly desireable.  Especially after a close call
to losing everything on our 7200v's due to bad directions given by an outside consultant (he had directed onsite consultant
to turn on GTM & LTM on the host which was vCMP dedicated.)  Fortunately, config backups of the mgmt existed as a manual
backup done prior to making those changes along with our legacy backup process.

On the 6400s, which started out live as 9.3.1, later upgraded to 10.2.3 and finally 10.2.4.  I had written a script named
`/config/backup`, so that it is sync'd and preserved across upgrades.  That is called by cron from root, which I have to
remember to re-add after any application of HF or upgrades.

{% highlight bash %}
#!/bin/bash

# backup the configuration
b config save /config.ucs
b export /config.scf

# Copy the configuration to somewhere
scp /config.ucs someuser@somewhere.foo.ksu.edu:/f5-backup/$HOSTNAME/config-`date +%Y%m%d`.ucs
scp /config.scf someuser@somewhere.foo.ksu.edu:/f5-backup/$HOSTNAME/config-`date +%Y%m%d`.scf
scp /config.scf someuser@somewhere.foo.ksu.edu:/f5-backup/$HOSTNAME.scf
{% endhighlight %}

A cronjob 'somewhere' does an `rcs` checkin attempt on the second scf, in an attempt to provide some way to track changes
to that F5.

Other scripts also exist on our 6400 that are stored in the local `cronuser` user's home directory.  All these scripts
make use of bigpipe.  And, will need to get done some other way entirely.

When we got our 7200v, it initially came with 11.4.1, but it was our desire to run 11.5.1 in the vCMP guests.  And, it
had apparently been decided that the host should get upgraded to 11.5.1 first (I missed the first day with
_(different)_&nbsp;outside consultant, as I was at The University of Kansas Hospital that day.  Being there on a different
day was also the reason I missed out on a chance to register for the recently Gallifrey One.  I haven't decided if 2016 is
is something I will try to register for this year, as priorities have shifted to being able to regulary attend other
conferences.

I had initially converted the backup script to use tmsh and had it working on the vCMP host, but getting it to work on the
vCMP guest has been problematic.  Especially when appliance mode is active.

Either there was something different about adding files to /config & ucs in 11.5.1, or I just opted to put them under
`cronuser`'s home directory, as: `/home/cronuser/scripts/backup`.

{% highlight bash %}
#!/bin/bash

rm -f /root/config-*
# backup the configuration
tmsh save /sys ucs /root/config-`date +%Y%m%d`.ucs
tmsh save /sys config file /root/config-`date +%Y%m%d`.scf

# Copy the configuration to somewhere
scp /root/config-`date +%Y%m%d`.* someuser@somewhere.foo.ksu.edu:/f5-backup/$HOSTNAME
scp /root/config-`date +%Y%m%d`.scf someuser@somewhere.foo.ksu.edu:/f5-backup/$HOSTNAME.scf

# cleanup
rm -f /root/config-*
{% endhighlight %}

In addition to using tmsh, this one eliminates the first scf, its redundant with the version in RCS.  And, it cleans up for
itself.

There have also been problems with `somewhere` that have resulted in missed backups.

So, it occurs to me to search on devcentral to see how other people are backing up their F5's, and perhaps they have shared
the solution.  Better yet, they offer it in the form of an appliance.

In the first search, I found an iApp to do backups.  It didn't work if the vCMP was in appliance mode though.  But, then
found [F5configbackup](http://nerdof.technology/project/config-backup-for-f5).  This seemed to hold great promise, as it
used the F5's iControl API to get device information and download files.  Only downloads when UCS archives have changed, and
of great interest, collects certificate information, and emails reports about upcoming certificate expriration.

The latter was something I had somehow done to the 6400, that likely won't get converted to work on 11.5.1.  As, have jobs
that send emails is much more difficult, with the removal of postfix, and perhaps might be something I'll have to explore as added functionality to the _F5configbackup_ appliance.

Due to an incident where all the 'admin'/'root' passwords are the same among different classification of F5 pairs.  We made
each pair have different 'admin'/'root' passwords from each other, in one case the 'admin' and 'root' passwords differ.
The 'admin'&'root' password being the same is less of an issue with the vCMP guests running in appliance mode, as root is
disabled. (along with bash, which is a double whammy to root cronjobs.  Wonder what the implication is for the lines for
`/usr/bin/diskmonitor` and `/usr/bin/copy_rrd save` ?

What was done, manual edits to bigip_base.conf, under 11.5.1 with continous sync enabled caused lots of problems.  Ended
up having to break the device group and rebuild it and sort out the configuration differences/expected, and then decide on
whether to do continuous syncs.  Eventually, the answer was yes.  As Firewall Managers would very much like to have their configuration changes sync'd to the other unit, but lack the ability to do a manual config sync.  They also for some unknown reason lack the ability to view ARP via the CU, so have to resort to using tmsh.

Things are also different on these new F5's....historically, it (prefered) to have on EST admin using the admin account at
one time.  Which was easy when there were only two admins fluent in F5, one more so than the other as primary.  Later when
there were two more, where neither seemed to know the issue of simultaneous changes being a problem....caused a few issues
now and then.  Especially as I was trying to make more drastic configuration changes to the F5, to reparition for
administrative groups.  Since it had come up from 9.3.1, this concept didn't exist, but as we added remote users (all
as 'Operators') so that they could bring up extra nodes for capacity or take nodes out for maintenance.  I had suggested that
this could be done through special monitors, but in the end it was give them control of the nodes, since the regular flood
of tickets to turn disable or enable nodes was getting somewhat annoying (at one time I included a note about using a special
monitor to handle the problem in the future when closing every such ticket, but evidently the completion notice doesn't get
read.)

With the new F5's, the plan is to allow greater access to certain individuals within an administrative parititon, especially
on the pair for development systems ('Manager' or 'Application Editor').  In this new scheme, nobody should use the 'admin'
account, nor have the 'Administrator' role.

As a result, EST admins only have 'Resource Administrator' roles, IT Security have 'Firewall Manager' roles, and 4 roles
have been defined for 3 of 4 administrative partitions. ('Manager', 'Application Editor', 'Operator', 'Guest')  The new F5
uses our central LDAP for authentication and authorization (group == role).  While with the old 6400, which had been doing
local passwords it had been updated to use LDAP for authentication (retaining local authorization settings), as there are
users that forget that when they ahve to change their password (every 180 days or 90 days, depending on PCI status) with
central IDM that the local password for their F5 access doesn't change.  And, they've promptly forgotten their old password.
So, having worked out the issues in getting the new F5's to use LDAP, and finally got LDAP working on the old F5.  The
'Auditor' role is also defined.  I have reserved remote role values for possible addition of the other defined roles.

FWIW, doing the manual repartitioning on under 11.5.1 would be much more difficult has instead of sections in the single
bigip.conf, they switch to using sub-folders with additional conf files, etc.  Though due to significant changes in how
things are being done going forward, all migrations will generally take the form of build a new system to replace the old
one and put that into its place in behind the new F5. or some variation of that order...

So, anyways, back to the issue of doing config backups of our F5's.  This application showed great promise, I configured it
with names and IPs of all current and future F5 units, and discovered what appeared to be the first shortcoming.  The
backup password is entered separately, and it is only one username/password used to backup all units.  Which won't work
in our environment.  And, I didn't what to explore getting a service account with 'Administrator' role just yet.

But, curious about the certificate feature, I gave it the 'admin' password for our 6400s.  Which to much joy it was able to
backup.  And, provide nice report of certificates, including on that had expired over a year ago, which was getting regular
and heavy use, but apparently its users don't care that it has expired.  Perhaps its expired status is a required part of
ensuring they've connected to their server.  The server is currently down due to a hardware issue where they systme is still located in corner of a network closet, but the personnel of the unit have been relocated to another state and for me to 
get physical access would require them to escort/surpervise.  Though in the past, they would do the physcial access
themselves and coordinate by phone for the what can be managed through the ALOM.  If they had some soft of remotely
accessible PDU, this would solve the physical access required part (in theory).  It is an external disk array which when
the controllers get out of sync, requires a complete power off to reset correctly.  There is supposed to be a web interface
to shutdown, but it doesn't remain available to power the system back up.  And, its reset doesn't remove power so doesn't
fix the issue.

If only we had continued the work to migrate the Solaris Zone to a FreeBSD jail.  The Solaris server is more than 4 years
past EOL.  (it did get a borrowed powesupply a couple years ago...a left over from when we tried to keep spare parts on
hand for our servers.)

I then changed the password to our first (at the time only) vCMP guest pair, where much to my dismay all the certificate
information that it had collected from our 6400s was purged.  Could be a problem in the future should there be some kind
of disruption causing some units to be inaccessible during a daily backup.  Though haven't decided if the acknowledge
option is relevant to my needs yet.

I left the _F5configbackup_ appliance running as I continued to look for other solutions or get pulled away to other
projects.  At the time of the recent crisis, and wondering if there were configuration backups for everything.  A new pair
of vCMPs had come into being and no known config backup existed for them.  Fortunately, after re-reading the documentation
numerous times, I read that we should be able to reattach the image and not lose its configuration.  And, everything seemed
to come back (with assistance from F5 support.)  The only thing broken was client certificate for configuration utility on
the primary host.  It still works on the secondary host.  Though doing client certificate was mainly a proof of concept, so
currently inorder for the _F5configbackup_ appliance to work it isn't being used.

Well, now that having config backups has become a priority, I turned my attention to looking at it again.  But, first
I couldn't recall what passwords I had used to.  I was pretty sure I had used a variation of our 'admin' passwords, just
without special characters (a limitation of the app.)   Eventually, I had to go for the reset web 'admin' script, but first
I had to recall how to log into the appliance by ssh.  I knew of the port, but had forgotten that the user name was
'console'.  So, ended up looking for how to reset passwords when I don't know what the root password is (the appliance makes
it that way.)  Found the procedure [here](https://wiki.centos.org/TipsAndTricks/ResetRootPassword), wasn't after the root
pasword in this case.  There is a note for CentOS-6, which I needed.  Except I stopped at booting with "selinux=0", instead
of reading on to "setenforce 0".  The latter may have caused me less grief.  Since apparently selinux detects if it had been
previously booted by a non-selinux kernel, and does an 'autorelabel'.

Guessing that original the db files had `chcon` done to them so that the UI/webserver was able to make updates to them, but
the auto-relabel changed it so that they were read-only.  Which made it hard to make any configuration changes, namely when to do backups and the 'admin' password.  But the python side wasn't having any problems, as I found that they are started in a context with great power.

Only it took me over a week, to finally look to see if selinux might be the culprit.  Though during this time, I had made a
small change to the python side to support device specific passwords.  I found that the info is stored in BACKUP_USER table
as ID,NAME,PASS  The PASS is encrypted, so I would need to figure that out later.  But, the UI always stores the general
NAME,PASS info with ID=0.  And, the ID column in the DEVICE table is a unique autoincrement primary key, which I guess had
started from 1.  Given that the first DEVICE ID was 11.  It displays devices on the UI sorted by NAME, which looks strange
given that we named our units with pri- and sec- prefixes.  So, I had deleted all 10 units and added 10 new ones moving the
prefix to end.  Though may revert now that I've addressed it (more later.)

But, since getting backups was high priority, I decided to go ahead and make a python side change.  And, wrote a small php
script that performed the encryption of passwords that produced the SQL statements to feed into sqlite3 to get the into the
database. 

Here's the hack I made to the f5backup service side:

{% highlight udiff linenos %}
diff --git a/src/lib/f5backup_lib.py b/src/lib/f5backup_lib.py
index 898fb8c..31d704a 100644
--- a/src/lib/f5backup_lib.py
+++ b/src/lib/f5backup_lib.py
@@ -101,14 +101,15 @@ and inserts into DB.
          e = sys.exc_info()[1]
          self.log.error('Can\'t update DB: add_error - %s' % e)
    
-   def getcreds(self,key):
+   def getcreds(self,key,id=0):
       '''
-getcreds(key) - Gets user credentials from DB 
+getcreds(key, id=0) - Gets user credentials from DB 
 args - key: encryption key used by m2secret 
+id - device id of creds to fetch, default is for default creds
 Return - dict of creds
       '''
       # Get user and encypted pass from DB, convert to dict of str types
-      self.dbc.execute("SELECT NAME,PASS FROM BACKUP_USER")
+      self.dbc.execute("SELECT NAME,PASS FROM BACKUP_USER WHERE ID = %d" % id)
       raw_creds = dict(zip( ['name','passwd'], [str(i) for i in self.dbc.fetchone()] ))
       
       # Decrypt pass and return dict of creds
@@ -299,7 +300,7 @@ def main():
    try:
       with open(sys.path[0] + '/.keystore/backup.key','r') as psfile:
          cryptokey =  psfile.readline().rstrip()
-      creds = job.getcreds(cryptokey)
+      creds0 = job.getcreds(cryptokey)
    except:
       e = sys.exc_info()[1]
       log.critical('Can\'t get credentials from DB - %s' % e)
@@ -359,6 +360,15 @@ def main():
       else:
          log.debug('Created device directory %s/devices/%s' % (sys.path[0],dev['name']))
       
+      # Get credentials for device from DB, if failure use global
+      log.debug('Retrieving credentials from DB.')
+      try:
+         with open(sys.path[0] + '/.keystore/backup.key','r') as psfile:
+            cryptokey =  psfile.readline().rstrip()
+         creds = job.getcreds(cryptokey,dev['id'])
+      except:
+	 creds = creds0
+
       # Get IP for device or keep hostname if NULL
       ip = dev['name'] if dev['ip'] == 'NULL' else dev['ip']
       
@@ -437,6 +447,7 @@ def main():
    
    # Clear creds & key
    creds = None
+   creds0 = None
    cryptokey = None
    
    #  Add deletion note to log file
@@ -470,3 +481,4 @@ def main():
    # Clear objects
    log.close()
    del log,job,date,dbc,db
+
{% endhighlight %}

The other direct SQL change I made was to change the backup time, from 13:50 to various other times in the afternoon
until 19:50, after that I called it a success and changed it to 23:50 to run a few days to make sure it appeared to do
the right thing.

Meanwhile, its time to find out why the UI isn't working.  But, first I wanted somewhere to _publish_ my change, and include
any other changes I might make along the way (mostly in the UI side.)  I'm not familiar with how things work with git on
sourceforge, so I opt to clone into a repository under my [github](https://github.com/TheDreamer) account.

https://github.com/TheDreamer/f5configbackup

With this in place, I start looking at how the PHP code works.  And, I can't help to make some tweaks along the way.  So
far the cosmetic changes have been to sort the devices table by IP instead of NAME, but not to work, I made the table
sortable.  While at it, I also made the Certificates and Users table sortable.  Eventually, I got putting a lot of debug
into `user_add.php` in trying to figure out why it wasn't making any changes.  Well, turns out its being silent about
database errors, so it will always report success when nothing happened.  But the debug code reveals that the database is
read-only to it.

In my search on selinux, I found this page: https://wiki.centos.org/HowTos/SELinux  Though it took reading other pages to
get to the point of understanding selinux and other things to figure out what I needed to be looking for, understand what
I was looking at, and to start attempting to fix things.

What it came down to is the webserver is running as `unconfined_u:system_r:httpd_t:s0`, while all the files/dirs under
`/opt/f5backup` are relabeled as `system_u:object_r:usr_t:s0`.  Files that I've added during troubleshooting are of
`unconfined_u:object_r:usr_t:s0`.  And, in the end the key detail was what `httpd_t` is allowed to do to `usr_t`, likely
the quick fix would be to chcon the files into `httpd_t`...though I was having trouble getting that to work, or how to make
the context change permanent.  So, I continued on to creating a custom selinux policy.

Along the way I had figure out what packages I needed to install inorder to run the various selinux utilities.

At first doing:

    # grep httpd_t /var/log/audit/audit.log | audit2allow -m f5backupui > f5backupui.te

resulted in:

```
module f5backupui 1.0;

require {
	type httpd_t;
	type usr_t;
	class file write;
}

#============= httpd_t ==============
allow httpd_t usr_t:file write;
```

This change the error from the database being read-only to being unable to open it.  Searches on sqlite and selinux pointed
to issues with the journal file.  But, no new denials showed up in the audit log.  But, I took a guess that it also need
write to the directory.  So, the `f5backupui.te` file became:

```
module f5backupui 1.0.1;

require {
	type httpd_t;
	type usr_t;
	class file write;
	class dir write;
}

#============= httpd_t ==============
allow httpd_t usr_t:file write;
allow httpd_t usr_t:dir write;
```

This didn't work any better. So, I then read elsewhere about doing:

    # setenforce 0

With this the user add worked.  And, there were additional 'AVC' messages in the audit log.  So, I generated a `third.te`,
good thing I didn't name it `charm.te` :blush:

```
module f5backupui 1.0.2;

require {
	type httpd_t;
	type usr_t;
	class file { write create unlink };
}

#============= httpd_t ==============

#!!!! This avc is allowed in the current policy
allow httpd_t usr_t:file write;
allow httpd_t usr_t:file { create unlink };
```

Still no joy.  decided that its still missing write to the directory, so I created `forth.te`:

```
module f5backupui 1.0.3;

require {
	type httpd_t;
	type usr_t;
	class file { write create unlink };
	class dir write;
}

#============= httpd_t ==============

allow httpd_t usr_t:file { write create unlink };
allow httpd_t usr_t:dir { write };
```

This still didn't work.  The site that I was reading this on, says maybe denial of the action is the correct thing, but
not cluttering up the log file is desired, in that case replace 'allow' with 'dontaudit'.  So, I wondered how do I determine
what the silent denials are, a search brought me to this [page](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/sect-Security-Enhanced_Linux-Fixing_Problems-Possible_Causes_of_Silent_Denials.html).

Doing the:

    # semodule -DB

And, trying things agin, yielded a `fifth.te`

```
module f5backupui 1.0.4;

require {
	type httpd_t;
	type usr_t;
	class file { write create unlink };
	class dir { remove_name add_name };
}

#============= httpd_t ==============
allow httpd_t usr_t:dir { remove_name add_name };
allow httpd_t usr_t:file { write create unlink };
```

And, still no luck.  Well, its still missing write to the directory.  So I tweak `fifth.te` to become:

```
module f5backupui 1.0.5;

require {
	type httpd_t;
	type usr_t;
	class file { write create unlink };
	class dir { write remove_name add_name };
}

#============= httpd_t ==============
allow httpd_t usr_t:dir { write remove_name add_name };
allow httpd_t usr_t:file { write create unlink };
```

Success! :+1:

Though this probably opens up too much to httpd_t now, but its nearly midnight on an EOF Friday...and I'm starving and ready
for bed.

-----

Next will be to get figure out how to implement the desired changes of device specific backup user/password info, along with
some of the other wishlist items.  As per usual, the thought of how does the _F5configbackup_ appliance get backed up. But,
the direction our environment is headed is that our backup is (_asynchronous_) **replication** to Lawrence.
