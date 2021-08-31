Title: Access PostgreSQL via SSH tunnel
URL: access-postgresql-via-ssh-tunnel
Blurb: How I access PostgreSQL when it only listens to the server it is running on.
Type: blog
Tags: PostgreSQL,SSH

As part of my security policy, PostgreSQL (psql) doesn't allow outside connections. While this provides a lesser attack surface area, it does make running queries much more convoluted. Typical work flow uses something like this:

1. Write article, workshop entry, change in schema etc
2. Write new entry SQL command around said article
3. Run the command using the local installation of psql
4. Identify and make changes
5. GOTO step 1 until all changes needed are done
6. SCP the file to the server
7. SSH to the server (probably already have open in another window)
8. Run the command using the server installation of psql
9. Go to the local installation of psgq and reset the db ready for the next article

Now if I discover that I need to make a change (spelling usually or not closing tags) on the server version, it is usually easier to `UPDATE` than `INSERT` new changes.

1. Make updates by manually adding the 'contenteditable' attribute to the whole article; this shows what changes will look like in real time
2. Write changes to a new file
3. Write new entry SQL `UPDATE` command around said article the file to the server
4. SSH to the server (probably already have open in another window)
5. Run the command using the server installation of psql

Another option is to:

1. Make updates by manually adding the 'contenteditable' attribute to the whole article
2. Write changes to a new file
3. Write new entry SQL `UPDATE` command around said article
4. SSH to the server (probably already have open in another window)
5. Connect to the psql install and then to the db `psql -U username dbname`
6. Run the command by copying the local file contents into the server window psql prompt

Either way, not very efficient. What is more efficient is being able to use an SSH connetion to transparently open a path the server psql that the local one can use with SSH protection around the whole thing. An SSH tunnel. The easiest way to do this is: '`ssh -N -L 5555:localhost:5432 server`'.

Buuuut...couldn't you just whitelist your ip adress on the server? What if you don't have permission to do so, or you move around a lot or just don't want any external interface to your database/whatever.

The trick to understanding these tunnels is to read from the outside in. Let's do that now:

1. '`ssh ... server`' Connect to server
2. '`-N`' Don't connect a command prompt: we don't want to do anything directly
3. '`-L 555:localhost:5432`' This is the meat of the operation and is discussed below

The general idea is to use the established connection to transparently forward ports from one computer, computer through the ssh connection to the port on the other computer. It can get (much) more complicated than that but for our purposes today let's leave it at that.

Specifically in our connection the following happens: forward any connection to port 5555 through the ssh tunnel to (and this is the trick) what is now (since we are now on the server) the (remote remember) severs version of localhost port 5432. Once you understand that, most tunnels make much more sense. That idea is so important I will say it again: that 'localhost' is the remote connection's version of localhost not the local version. Renaming might clear up any stragglers: '`ssh -N -L 5555:remoteserversversionoflocalhost:5432 remoteserver`'

Check `man ssh` for better technical description as to what is happening.

In practice this is a great way to send commands to pg (5432 is the default port) on a server that does not allow remote connection directly to pg, but you do have SSH access to.

N.B. There is one more thing to watch out for, PostgreSQL specific: by default psql tries to connect over a Unix socket which tunneling (AFAIK) can't handle. To get around this you need to tell psql to use a TCP connection by adding in the '`-h localhost`' flag which defaults to TCP connections. That `-h` is the `remoteserversversionoflocalhost` remember.

In action: '`ssh -N -L 5555:localhost:5432 server`'. You will end up (after authenticating) with a terminal window that looks like it is waiting to return; it will not. You could run it in the background if you wanted to establish a more permanent, out of the way connection by ameding '`-f`' and '`&`' ('`ssh -f -N -L 5555:localhost:5432 server &`')d. In another window send the command you want (or connect via pgAdmin3 etc): '`psql -h localhost -U username -p 5555 < commandtorun.sql`'. If everything has gone well, your command will automagically just run like the machine was sitting next to you.