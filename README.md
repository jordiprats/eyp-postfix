# postfix ![status ready](https://img.shields.io/badge/status-ready-brightgreen.svg)

#### Table of Contents

1. [Overview](#overview)
2. [Module Description](#module-description)
3. [Setup](#setup)
    * [What postfix affects](#what-postfix-affects)
    * [Setup requirements](#setup-requirements)
    * [Beginning with postfix](#beginning-with-postfix)
4. [Usage](#usage)
5. [Reference](#reference)
5. [Limitations](#limitations)
6. [Development](#development)
    * [Contributing](#contributing)

## Overview

postfix management

## Module Description

postfix setup and configuration, can be configured to act like a simple mail relay or a multidomain mailserver

## Setup

### What postfix affects

* /etc/postfix/main.cf
* package management
* service management
* purges packages for other MTA on CentOS and switches to postfix on Ubunut 14.04

### Setup Requirements

This module **requires pluginsync** enabled and **eyp-dovecot** (it is required to be able to setup IMAP for the mailserver: **postfix::vmail**)

### Beginning with postfix

basic setup:

```puppet
class { 'postfix':
  inetinterfaces => 'localhost',
}
```

mail relay:

```puppet
class { 'postfix':
  relayhost => '1.2.3.4',
}
```

multidomain mail server

```
class { 'postfix': }

class { 'postfix::vmail': }

postfix::vmail::alias { 'example@systemadmin.es':
  aliasto => [ 'exemple@systemadmin.es' ],
}

postfix::vmail::account { 'example@systemadmin.es':
  accountname => 'example',
  domain => 'systemadmin.es',
  password => 'secretpassw0rd',
}

postfix::vmail::account { 'silvia@systemadmin.es':
  accountname => 'silvia',
  domain => 'systemadmin.es',
  password => 'secretpassw0rd2',
}

postfix::vmail::account { 'marc@systemadmin.es':
  accountname => 'marc',
  domain => 'systemadmin.es',
  password => 'secretpassw0rd3',
}
```

To setup **opportunistic TLS with custom certificates**:

```puppet
	class { 'postfix':
		opportunistictls => true,
		tlscert => 'puppet:///openldap/masterauth/ldap-master-01.crt',
		tlspk => 'puppet:///openldap/masterauth/ldap-master-01.key.pem',
	}
```

To setup **opportunistic TLS with selfsigned certificate**:

```puppet
	class { 'postfix':
		opportunistictls => true,
		subjectselfsigned => '/C=ES/ST=Barcelona/L=Barcelona/O=systemadmin.es/CN=systemadmin.es',
		generatecert => true,
	}
```

## Usage

This module can be used to configure postfix to relay mails to another server or to have virtual mailboxes (multidomain/multiaccount).

multiple smtp outbound instances:

```
postfix::masterservice { 'out_domain1':
  type => 'unix',
  chroot => 'n',
  command => 'smtp',
  args => { 'smtp_bind_address' => '1.1.1.1',
            'smtp_helo_name' => 'systemadmin.es',
            'syslog_name' => 'postfix-systemadmin.es',
          }
}

postfix::masterservice { 'out_domain2':
  type => 'unix',
  chroot => 'n',
  command => 'smtp',
  args => { 'smtp_bind_address' => '1.2.2.2',
            'smtp_helo_name' => 'sysadmins.es',
            'syslog_name' => 'postfix-sysadmins.es',
          }
}
```

## Reference

### postfix

Most variables are standard postfix variables, please refer to postfix documentation:
 * append_dot_mydomain
 * biff
 * inetinterfaces
 * ipv6
 * mail_spool_directory
 * mydestination
 * mydomain
 * myhostname
 * mynetworks
 * myorigin
 * readme_directory
 * recipient_delimiter
 * relayhost
 * smtp_fallback_relay
 * smtpdbanner
 * install_mailclient
 * default_process_limit
 * smtpd_client_connection_count_limit
 * smtpd_client_connection_rate_limit
 * in_flow_delay
 * setgid_group
 * (...)

* **install_mailclient**: controls if a mail client should be installed (default: true)

#### SSL certificates:
* **opportunistictls**: controls Opportunistic TLS (default: false)
* **generatecert**: controls if a selfsigned certificate is generated for this postfix instance (default: true)
* **tlscert**: source cert file - **generatecert** must be false
* **tlspk**: source private key - **generatecert** must be false
* **subjectselfsigned** subject for a selfsigned certificate - **generatecert** must be true. example: '/C=RC/ST=Barcelona/L=Barcelona/O=systemadmin.es/CN=systemadmin.es',

### postfix::transport

```puppet
postfix::transport { 'example.com':
  error => 'email to this domain is not allowed',
}
```

```puppet
postfix::transport { 'example.com':
  nexthop => '1.1.1.1',
}
```

### postfix::vmail

* **mailbox_base**: (default: /var/vmail)
* **setup_dovecot**: (default: true)
* **smtpd_recipient_restrictions** (default: permit_inet_interfaces,permit_mynetworks,permit_sasl_authenticated,reject_unauth_destination)
* **smtpd_relay_restrictions** (default: permit_inet_interfaces,permit_mynetworks,permit_sasl_authenticated,reject_unauth_destination)

### postfix::vmail::acount

```puppet
postfix::vmail::account { 'silvia@systemadmin.es':
  accountname => 'silvia',
  domain => 'systemadmin.es',
  password => 'secretpassw0rd2',
}
```

### postfix::vmail::alias

```puppet
postfix::vmail::alias { 'example@systemadmin.es':
  aliasto => [ 'exemple@systemadmin.es' ],
}
```

## Limitations

Tested on:
* CentOS 5
* CentOS 6
* CentOS 7
* Ubuntu 14.04
* SLES 11 SP3

## Development

We are pushing to have acceptance testing in place, so any new feature should
have some test to check both presence and absence of any feature

### TODO

* improve documentation (multidoamin mailserver is not yet covered)

### Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
