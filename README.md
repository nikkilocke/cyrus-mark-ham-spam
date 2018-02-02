# cyrus-mark-ham-spam
Bash script to trawl through user's mailboxes and SPAM folders, marking ham in the spam folder as spam, and vice versa

Requires Cyrus and SpamAssassin to be installed.

Tested on ClearOs 7.

You must also have a filter which filters mail with `X-Spam-Flag: YES` headers into a separate SPAM folder. Instructions for setting up such a filter are below.

With that filter set up for a user, all mail SpamAssassin thinks is spam will automatically be put in your SPAM folder. Mail SpamAssassin thinks is not spam will go in your INBOX.

If you find spam in your INBOX, merely move it to the SPAM folder. Likewise, if you find ham (i.e. mail which is not spam) in your SPAM folder, merely move it to your INBOX.

The script looks through Cyrus IMAP mailboxes, searching for mail which has `X-Spam-Flag: YES` in the INBOX - this is assumed to be mail marked as spam by SpamAssassin, but which you have moved to your INBOX. It will take the mail, strip all the SpamAssassin headers out, and pass it to sa-learn to tell the SpamAssassin engine it is ham.

It also looks through the SPAM folder, looking for mail which has `X-Spam-Flag: NO` - this is assumed to be mail which SpamAssassin marked as ham, but which you have moved to your SPAM folder. It will take the mail, strip all the SpamAssassin headers out, and pass it to sa-learn to tell the SpamAssassin engine it is spam.

## Installation

Copy the script anywhere you like (in the examples below we assume it is in `/usr/local/bin`), and make it executable using `chmod +x cyrus-mark-ham-spam`.

### Setting up the SPAM filter

Each user must create a Cyrus Sieve script that looks like this:

	require ["fileinto"];
	if header :contains "x-spam-flag" "YES" {
	  fileinto :create "INBOX/SPAM";
	  stop;
	}

Note: The above assumes you have `unixhierarchysep: yes` in your `/etc/imapd.comf`. If you do not, the `INBOX/SPAM` should be `INBOX.SPAM`.

They must then load it into Cyrus using the `sieveshell` command, like as follows (the `>` is the sieveshell prompt, and the file above is called `spamrule`):

	sieveshell -u (your user name) localhost:2000
	Please enter your password:
	> put spamrule
	> activate spamrule
	> list
	spamrule  <- active script
	> quit

### Running the script

The script *must* be run as user `cyrus` (or whatever user runs cyrus and owns the mail files on your system). If you are root, you can run it using the command:

	su -s /bin/bash -c '/usr/local/bin/cyrus-mark-ham-spam' cyrus

I suggest you run it with the `-t` and `-v` parameters to test it first, then with `-v` until you are happy it is doing the right thing.

### Setting the script up to run automatically

You can add a file to `/etc/cron/d` to run the script, as follows:

	SHELL=/bin/bash
	PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
	MAILTO=nikki
	#min	hr	day	month	weekday	user	command
	01		2	*	*		*		cyrus	/usr/local/bin/cyrus-mark-ham-spam -v

This runs it at 02:00 am every night.

