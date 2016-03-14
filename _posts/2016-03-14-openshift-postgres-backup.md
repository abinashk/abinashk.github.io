---
layout: post
title: Backup Postgresql database to your local machine from Openshift server
excerpt: "Simple way to create a backup of postgresql database in your Openshift server and download it to your local machine when the database gear is different from the head app gear."
tags: [openshift, postgresql, backup]
modified: 2016-02-16
comments: true
---

'pg_dump' is a very useful tool to create backups of your postgresql database. It is very straightforward to use in normal linux servers. But in case of Openshift, especially if your app gear and database gear are different, it can be a little bit tricky. I have stumbled around many times to figure this out. Following approach has worked for me every time.

First we should get the ssh address of the postgress gear. For this we can use the 'rhc app show' command which lists the gears in your application and their ssh addresses. 

{% highlight shell %}
rhc app show <app_name> --gears
{% endhighlight %}

This will give you a list of gears in your app along with their ssh address. Copy the ssh address for your postgres gear. 

Now you need to 'ssh' into that gear using 'rhc ssh' command.

{% highlight shell %}
rhc ssh <ssh_address_of_postgres_gear>
{% endhighlight %}

You'll be in the root directory of your gear. It is not a good idea to make a backup file in that directory. Every Openshift gears have a data directory
which doesn't get wiped out in every deployment(all other directories get wiped clean in each deployment). So we'll use this data directory to create the backup file. First cd to the data dir.

{% highlight shell %}
cd /app-root/data
{% endhighlight %}

Now you can use 'pg_dump' command to create a backup.

{% highlight shell %}
pg_dump <database_name> > <backup_file_name>
{% endhighlight %}

After this if you want to download the backup to your local machine then you can do 'scp' using the ssh address that you got in the first step. First you need to logout from the server 'Ctrl + c' or 'Ctrl + z' will log you out from the ssh session. Before logging out from the server you need to copy the full path of the backup file. Usually 'pwd' command can give you the full path of the folder and after adding the filename to that path it will be the full path of the file. Now, you need to log out of the server and use scp from the terminal in your machine.

{% highlight shell %}
scp <ssh_address_of_postgres_gear>:<full_path_of_backup_file> <destination_path_in_local_machine>
{% endhighlight %}

This will download the backup file to the path that you entered as the destination.
