# cardboard
PD - Documentation: 
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
First, install postfix:
CMD1:
$ sudo apt install postfix postfix-mysql dovecot-core dovecot-imapd dovecot-lmtpd dovecot-mysql      // you can use "apt" or "apt-get", where apt is new version for installation.
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// now you will see a graphical choice box, asking to select a choice
Please select  >> Internet site  // then hit return (enter).
// Its time to give a system mail name
As mentioned in the requirement  type >>mails.net //then hit return (enter).
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
//now creating MySQL database along with virtual domains, and users 

CMD2:
$ mysqladmin -p create <name_as_you_like>              Note: *(<name_as_you_like> = server name)*
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
//Now login using root to MySQL

 CMD3:
$ sudo mysql -u root -p    // if it is successful you can see the below prompt

" mysql >"  // If you don't see the prompt then you have installed "MariaDB" or some other database. Please type CMD3.1> use mysql;  -------- MySQL command.
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// It's time to give SELECT permisson to add users with authentication.

mysql > GRANT SELECT ON <name_as_you_like>.* TO 'usermail'@'127.0.0.1' IDENTIFIED BY 'mailpassword';
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// reloading MySql permissions

mysql > FLUSH PRIVILEGES;
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// now use database for creating tables and introducing data (username and password)

mysql> USE servermail;
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// creating a DOMAIN table on servermail
enter the followig MySQL command:

CREATE TABLE `virtual_domains` (
`id`  INT NOT NULL AUTO_INCREMENT,
`name` VARCHAR(50) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// Creating a USER table on servermail.
enter the following MySQL command:

CREATE TABLE `virtual_users` (
`id` INT NOT NULL AUTO_INCREMENT,
`domain_id` INT NOT NULL,
`password` VARCHAR(106) NOT NULL,
`email` VARCHAR(120) NOT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `email` (`email`),
FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// creating an alias user table on servermail database
enter following MySQL command:

CREATE TABLE `virtual_aliases` (
`id` INT NOT NULL AUTO_INCREMENT,
`domain_id` INT NOT NULL,
`source` varchar(100) NOT NULL,
`destination` varchar(100) NOT NULL,
PRIMARY KEY (`id`),
FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// virtual domain name entry (adding 4 different domains)

INSERT INTO `servermail`.`virtual_domains`
(`id` ,`name`)
VALUES
('1', 'domain1.com'),
('2', 'domain2.com'),
('3', 'domain3.com'),
('4', 'domain4.com');
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// adding email and password

INSERT INTO `servermail`.`virtual_users`
(`id`, `domain_id`, `password` , `email`)
VALUES
('1', '1', ENCRYPT('password1', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'email1@domain1.com'),
('2', '2', ENCRYPT('password2', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'email2@domain2.com'),
('3', '3', ENCRYPT('password3', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'email3@domain3.com'),
('4', '4', ENCRYPT('password4', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'email4@domain4.com');
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// optional ___________ you can add alias mail to the mailserver

INSERT INTO `servermail`.`virtual_aliases`
(`id`, `domain_id`, `source`, `destination`)
VALUES
('1', '1', 'alias@domain1.com', 'email1@domain1.com'),
('2', '2', 'alias@domain2.com', 'email2@domain2.com'),
('3', '3', 'alias@domain3.com', 'email3@domain3.com'),
('4', '4', 'alias@domain4.com', 'email4@domain4.com');
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// exit from mysql

mysql > exit
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// Postfix configuration  ----  editing main.cf 
first copy a backup
CMD:

$ cp /etc/postfix/main.cf /etc/postfix/main.cf.orig

$ vi /etc/postfix/main.cf                                                                                    // main.cf  --------- edit begin

edit as shown below:
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# TLS parameters
#smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
#smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
#smtpd_use_tls=yes
#smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
#smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache 
smtpd_tls_cert_file=/etc/ssl/certs/dovecot.pem
smtpd_tls_key_file=/etc/ssl/private/dovecot.pem
smtpd_use_tls=yes
smtpd_tls_auth_only = yes
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

repeat above step:

smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions =
permit_sasl_authenticated,
permit_mynetworks,
reject_unauth_destination
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
comment following line 

#mydestination = example.com, hostname.example.com, localhost.example.com, localhost
mydestination = localhost 

now check for FQDN:

myhostname = hostname.example.com

now append following to the file 
virtual_transport = lmtp:unix:private/dovecot-lmtp
                                                                                                            // main.cf  --------- edit ends here

Note: for verification please check file data stored below:
==========================================================================================================================
# See /usr/share/postfix/main.cf.dist for a commented, more complete version


# Debian specific:  Specifying a file name will cause the first
# line of that file to be used as the name.  The Debian default
# is /etc/mailname.
#myorigin = /etc/mailname

smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# TLS parameters
#smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
#smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
#smtpd_use_tls=yes
#smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
#smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

smtpd_tls_cert_file=/etc/ssl/certs/dovecot.pem
smtpd_tls_key_file=/etc/ssl/private/dovecot.pem
smtpd_use_tls=yes
smtpd_tls_auth_only = yes

smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions =
        permit_sasl_authenticated,
        permit_mynetworks,
        reject_unauth_destination

# See /usr/share/doc/postfix/TLS_README.gz in the postfix-doc package for
# information on enabling SSL in the smtp client.

myhostname = hostname.example.com
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = /etc/mailname
#mydestination = example.com, hostname.example.com, localhost.example.com
mydestination = localhost
relayhost =
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all

## Tells Postfix to use Dovecot's LMTP instead of its own LDA to save emails to the local mailboxes.
virtual_transport = lmtp:unix:private/dovecot-lmtp

## Tells Postfix you're using MySQL to store virtual domains, and gives the paths to the database connections.
virtual_mailbox_domains = mysql:/etc/postfix/mysql-virtual-mailbox-domains.cf
virtual_mailbox_maps = mysql:/etc/postfix/mysql-virtual-mailbox-maps.cf
virtual_alias_maps = mysql:/etc/postfix/mysql-virtual-alias-maps.cf
===============================================================================================================================================
// Edit domains.cf file as shown below.

vi /etc/postfix/mysql-virtual-mailbox-domains.cf
		
user = usermail
password = mailpassword
hosts = 127.0.0.1
dbname = servermail
query = SELECT 1 FROM virtual_domains WHERE name='%s' 
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

now restart the postfix services

$ systemctl restart postfix
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
We need to ensure that Postfix finds your domain, so we need to test it with the following command. If it is successful, it should returns 1:


$ postmap -q example.com mysql:/etc/postfix/mysql-virtual-mailbox-domains.cf

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Then we need to create the mysql-virtual-mailbox-maps.cf file.


$ vi /etc/postfix/mysql-virtual-mailbox-maps.cf 
edit as below:
		
user = usermail
password = mailpassword
hosts = 127.0.0.1
dbname = servermail
query = SELECT 1 FROM virtual_users WHERE email='%s'
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

// now restart the postfix services

$ systemctl restart postfix

At this moment we are going to ensure Postfix finds your first email address with the following command. It should return 1 if it's successful:

postmap -q email1@example.com mysql:/etc/postfix/mysql-virtual-mailbox-maps.cf

 
Finally, we are going to create the last file to configure the connection between Postfix and MySQL.


nano /etc/postfix/mysql-virtual-alias-maps.cf
		
user = usermail
password = mailpassword
hosts = 127.0.0.1
dbname = servermail
query = SELECT destination FROM virtual_aliases WHERE source='%s'

// now restart the postfix services

$ systemctl restart postfix

We need to verify Postfix can find your aliases. Enter the following command and it should return the mail that's forwarded to the alias:

postmap -q alias@example.com mysql:/etc/postfix/mysql-virtual-alias-maps.cf
If you want to enable port 587 to connect securely with email clients, it is necessary to modify the /etc/postfix/master.cf file


nano /etc/postfix/master.cf
We need to uncomment these lines and append other parameters:


submission inet n       -       -       -       -       smtpd
-o syslog_name=postfix/submission
-o smtpd_tls_security_level=encrypt
-o smtpd_sasl_auth_enable=yes
-o smtpd_client_restrictions=permit_sasl_authenticated,reject


In some cases, we need to restart Postfix to ensure port 587 is open.
// now restart the postfix services

$ systemctl restart postfix

To be continued ....
