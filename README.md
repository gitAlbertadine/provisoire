
postmaster info@xmailing.me
#smtp-port 25
smtp-listener 54.218.71.176:25
<source 0/0>
	log-connections yes
	log-commands    yes      # WARNING: verbose!
  allow-unencrypted-plain-auth yes
</source>
sync-msg-create false
sync-msg-update false
run-as-root no
log-file /var/log/pmta/log        # logrotate is used for rotation

<acct-file /var/log/pmta/acct.csv>
#    move-to /opt/myapp/pmta-acct   # configure as fit for your application
#    move-interval 5m

    max-size 50M
</acct-file>

# transient errors (soft bounces)
<acct-file /var/log/pmta/diag.csv>
  move-interval 1d
  delete-after never
  records t
</acct-file>

#
# spool directories
#

spool /var/spool/pmta

http-mgmt-port 1983
http-access 127.0.0.1 admin
http-access 0/0 monitor
http-access 54.218.71.176 admin




# BEGIN: USERS/VIRTUAL-MTA / VIRTUAL-MTA-POOL /  VIRTUAL-PMTA-PATTERN

#<spool /var/spool/pmta>
#</spool>
<smtp-user user1>
	password pass123
	source {smtpuser-auth}
</smtp-user>
<source {smtpuser-auth}>
	smtp-service yes
	always-allow-relaying yes
	require-auth true
	process-x-virtual-mta yes
	default-virtual-mta pmta-pool
	remove-received-headers true
	add-received-header false
	hide-message-source true
</source>
#BEGIN VIRTUAL MTAS 
<virtual-mta pmta-vmta1>
smtp-source-host 54.218.71.176 server.xmailing.me
#domain-key dkim,*,/home/admin/conf/mail/example.com/dkim.pem
#domain-key default,*,/var/cpanel/domain_keys/private/example.com 
<domain *>
max-msg-rate 400/h
</domain>
</virtual-mta> <domain xmailing.me>
route [127.0.0.1]:25
</domain>
#END VIRTUAL MTAS


<virtual-mta-pool pmta-pool>
virtual-mta pmta-vmta1
</virtual-mta-pool>

# END: USERS/VIRTUAL-MTA / VIRTUAL-MTA-POOL /  VIRTUAL-PMTA-PATTERN

<source 127.0.0.1>
    always-allow-api-submission yes
    add-message-id-header yes
    retain-x-job yes
    retain-x-virtual-mta yes
    verp-default yes
    process-x-envid yes
    process-x-job yes
    jobid-header X-Mailer-RecptId
    process-x-virtual-mta yes
</source>

<domain xmailing.me>
route [127.0.0.1]:25
</domain>
