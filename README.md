`quotapolicy` : Postfix SMTP access policy for Unix filesystem quotas
======================================================================

This program is a dæmon that makes Postfix reject emails if:
 - The recipient is an Unix user, and
 - Their filesystem quota is full.

The quota check is run right during the SMTP transaction, so that Postfix won't
have to run further processing, test for spam, call maildrop/procmail, etc. (It
will also prevent a misconfigured procmail from silently storing messages at
`/var/mail`.)  The sender receives a failure message.

The program uses Postfix access policy delegation; for more info, see
http://www.postfix.org/SMTPD_POLICY_README.html .

Requirements
============

 - Python 2.x
 - python-daemon (available via apt-get or pip)
 - quota (the binary)
 - sudo

Mini-guide
==========

1. Copy `bin/quotapolicy` to `/usr/local/bin` (or elsewhere)

2. Create user `quotapolicy`

3. Add to `/etc/sudoers`:

        quotapolicy myhost=NOPASSWD: /usr/bin/quota

   Make sure it works without passwords:

        myuser$ sudo -u quotapolicy sudo /usr/bin/quota
   
4. `mkdir /var/spool/postfix/quotapolicy` (or whatever)

5. `chown quotapolicy: /var/spool/postfix/quotapolicy`

6. Setup your system to run `quotapolicy` on startup.  An `init.d` script is
   provided for Debian-style systems:

        quotapolicy$ sudo cp init.d/quotapolicy /etc/init.d/
        quotapolicy$ sudo update-rc.d quotapolicy defaults
        quotapolicy$ sudo /etc/init.d/quotapolicy restart

8. Add to main.cf (assuming chrooted postfix):
        smtpd_recipient_restrictions =
          permit_mynetworks
          check_policy_service unix:quotapolicy/quotapolicy.sock
          [other restrictions...]

9. Restart postfix.

