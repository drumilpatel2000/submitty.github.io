---
title: Email Configuration
category: System Administrator
---


Submitty can be configured to send instructor announcements by email
and send email (customizable -- *work in progress*) notifications to
users.


1. Obtain an email address that will be the sender for all email.

   This should probably not be an actual user nor a mailing list used
   for any other purpose.  It is probable that students who receive
   these emails may `reply-to` the sender without realizing they are
   sending it to a Submitty user rather than their instructor or
   teaching assistant.  Decide what you will do with these mistakenly
   sent emails.

   Learn what rate limits are configured for this email address.
   E.g., number of total emails sent per minute and/or number of
   emails sent per hour to external (non-university) email addresses.
   These limits may require adjustment of the `send_email.py` script.
   

2. Edit this configuration file:  `/usr/local/submitty/config/email.json`


   Add the following fields to the file (edit as appropriate):

   ```
   {
       "email_user": "<SPECIAL USER NAME>",
       "email_password": "<PASSWORD FOR SPECIAL USER NAME>",
       "email_sender": "submitty@myuniversity.edu"
       "email_reply_to": "submitty_do_not_reply@myuniversity.edu"
       "email_server_hostname": "mail.myuniversity.edu",
       "email_server_port": 587,
       "email_logs_path": "/var/local/submitty/logs/emails"
   }
   ```

   *NOTE:*  The user (login name) and sender (appears on the
    `From:` line of the emails) might not be exactly the same.


3. Ensure the permissions of this file allow read access by the
`submitty_daemon` user:

    ```
    -r--r----- 1 root submitty_daemonphp     email.json
    ```


## Troubleshooting


If you've done an action that should result in an email being sent and
it's not working:


1. Check the `emails` table in the master database.  First connect to the database:

   ```
   sudo su postgres
   psql
   \c submitty
   ```

   Then search for a specific target recipient that hasn't received email.  For example:

   ```
   select id,recipient,subject,created,sent from emails where recipient='myuser@university.edu' order by created;
   ```

   Verify that recent emails have been inserted into the table for the
   user (created column should have today's date & time).  Once the email
   is successfully sent, the sent column will be filled with that date
   & time.


2. Check the Submitty email error log:
   
   ```
   /var/local/submitty/logs/emails/<CURRENT_DATE>.txt
   ```


3. Attempt to manually run the script to send emails from the database:

   ```
   sudo su
   su submitty_daemon
   /usr/local/submitty/sbin/send_email.py
   ```

   *NOTE:* This script is configured to run once per minute by the
    `submitty_daemon` user.  Ensure that `submitty_daemon`'s crontab
    file is in order, for example:

   ```
   * * * * python3 /usr/local/submitty/sbin/send_email.py
   ```

