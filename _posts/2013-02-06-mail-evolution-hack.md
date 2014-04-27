---
title: hacking mail/evolution to fix the annoying dialog instead of reminders
layout: post
comments: yes
category: patches
not_pretty: yes
---

`mail/evolution` has this annoying feature, where when showing reminders from
external sources where an email reminder type is set for the appointment.  It
pops up a silent dialog that waits for you to acknowledge that `evolution`
doesn't yet support email reminders.  And, you have to acknowledge the dialog
before it sounds the alert and displays that you have an appointment.

So, I commented it out :smirk:

I see the dates have changed since I originally created this patch...but
here's the latest patch that would be submitted if I were to submit it....

{% highlight diff linenos %}
--- files/patch-calendar_gui_alarm-notify_alarm-queue.c.orig	2014-01-22 11:40:44.000000000 -0600
+++ files/patch-calendar_gui_alarm-notify_alarm-queue.c	2013-02-06 09:16:27.906137000 -0600
@@ -1,6 +1,6 @@
---- calendar/gui/alarm-notify/alarm-queue.c.orig	2011-03-07 20:53:40.000000000 +0100
-+++ calendar/gui/alarm-notify/alarm-queue.c	2011-03-07 20:53:50.000000000 +0100
-@@ -1606,7 +1606,7 @@ popup_notification (time_t trigger, Comp
+--- calendar/gui/alarm-notify/alarm-queue.c.orig	2010-11-02 00:23:52.000000000 -0500
++++ calendar/gui/alarm-notify/alarm-queue.c	2013-02-06 09:15:13.440137277 -0600
+@@ -1606,7 +1606,7 @@
  			body = g_strdup_printf ("%s %s", start_str, time_str);
  	}
  
@@ -9,3 +9,19 @@
  	if (!notify_notification_show(n, NULL))
  	    g_warning ("Could not send notification to daemon\n");
  
+@@ -1691,6 +1691,7 @@
+ 						CAL_STATIC_CAPABILITY_NO_EMAIL_ALARMS))
+ 		return;
+ 
++#if 0
+ 	dialog = gtk_dialog_new_with_buttons (_("Warning"),
+ 					      NULL, 0,
+ 					      GTK_STOCK_OK, GTK_RESPONSE_CANCEL,
+@@ -1706,6 +1707,7 @@
+ 
+ 	gtk_dialog_run (GTK_DIALOG (dialog));
+ 	gtk_widget_destroy (dialog);
++#endif
+ }
+ 
+ /* Performs notification of a procedure alarm */
{% endhighlight %}
