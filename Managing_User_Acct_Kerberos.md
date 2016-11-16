---
title: Managing User Accounts in Kerberos
keywords: Managing User Accounts Kerberos
last_updated: 'November, 2016'
tags:
  - Managing Accounts Kerberos
summary: How users create a Jupyter notebook instance. 
sidebar: mydoc_sidebar
permalink: managing_user_acct_kerberos.html
folder: mydoc
published: true
---

## Managing User Accounts in Kerberos
To manage user accounts in Kerberos, you will need access to the _kadmin.local_ tool or _kadmin_ tool.

## The kadmin.local tool

To use the _kadmin.local_ tool, you must either log onto the manager server as root or use _sudo kadmin.local_.
```
[ec2-user@ip-10-10-10-105 ~]$ sudo kadmin.local
Authenticating as principal cf/admin@CLOUDERA with password.
kadmin.local:
```

## The kadmin tool

1. To use the _kadmin_ tool, first run the _klist_ command to make sure that you don't have any active principals in Kerberos:
    ```
    [ec2-user@ip-10-10-10-105 ~]$ klist
klist: No credentials cache found (ticket cache FILE:/tmp/krb5cc_500)
    ```

9. To initialize a new client, use the _kinit <principal name>_ command and enter the password.
    ```
    [ec2-user@ip-10-10-10-105 ~]$ kinit cf
    Password for cf@CLOUDERA:
    ```
9. If you don't specify a principal name, the _kinit_ command will use your current username. The principal must have access to the admin console, and the principal's name should end with _/admin_. Otherwise, check your principal name. If something is wrong, use _kdestroy_ to destroy the session and use the _kinit <principal name>_ command again to create a new one.

9. After this step, the _klist_ command should show a result like this:
    ```
[ec2-user@ip-10-10-10-105 ~]$ klist
Ticket cache: FILE:/tmp/krb5cc_500
Default principal: cf@CLOUDERA

Valid starting     Expires            Service principal
07/22/15 07:29:48  07/23/15 07:29:48  krbtgt/CLOUDERA@CLOUDERA
        renew until 07/29/15 07:29:48
    ```
9. Use the _kadmin_ command and enter the password.

# Managing accounts in Kerberos
  
The question mark command (?) shows you a list of all commands available. 

## Principal privileges

Each principal's access is described in its name.
 - Access to admin console `<name>/admin@<domain>`, `cf/admin@CLOUDERA`
 - Access to specified host `<name>/<host>@<domain>`, `HTTP/ip-10-10-10-195.eu-west-1.compute.internal`
 - Access to all hosts `<name>@<domain>`, `cf@CLOUDERA`

## Creating a principal (user)
    
1. To add a principal to Kerberos, use the _addprinc_ command:
    ```
kadmin.local:  addprinc
usage: add_principal [options] principal
        options are:
                [-x db_princ_args]* [-expire expdate] [-pwexpire pwexpdate] [-maxlife maxtixlife]
                [-kvno kvno] [-policy policy] [-clearpolicy] [-randkey]
                [-pw password] [-maxrenewlife maxrenewlife]
                [-e keysaltlist]
                [{+|-}attribute]
        attributes are:
                allow_postdated allow_forwardable allow_tgs_req allow_renewable
                allow_proxiable allow_dup_skey allow_tix requires_preauth
                requires_hwauth needchange allow_svr password_changing_service
                ok_as_delegate ok_to_auth_as_delegate no_auth_data_required

...where:
        [-x db_princ_args]* - any number of database specific arguments.
                        Look at each database documentation for supported arguments
    ```

1. To create a user with a specified password, use the _addprinc <name>_ command and enter the password, or use the following: 

    `addprinc -pw <password> <name>`

You can also create a user with a random password for services with the _addprinc -randkey <name>_ command.

## Token lifetime

To change token lifetime, use the _-maxlife_ parameter with the _addprinc_ or _modprinc_ command. The token in the first example below expires after 30 days; the token in the second expires after 12 hours:

```
 modprinc -maxlife 30d cf 
 addprinc -maxlife 12h -randkey cf 
```

**Note:** The value you enter cannot exceed the _ticket_lifetime_ limit specified in the Kerberos configuration file, by default in the _/etc/krb5.conf_ file. 

## Creating a keytab file
  
To create keytab files, use the _ktadd_ command:
```
kadmin.local:  ktadd
Usage: ktadd [-k[eytab] keytab] [-q] [-e keysaltlist] [-norandkey] [principal | -glob princ-exp] [...]
```

Example: <br />
```ktadd -norandkey -k <file> <name1> <name2> ... <nameN>```

## Other useful commands

 * <b>modprinc:</b> Modifies Kerberos principal.
 * <b>listprincs:</b> Shows all Kerberos principals (users).
 * <b>delprinc <i>name</i>:</b> Deletes principal. 
 * <b>cwp <i>name</i>:</b> Changes password for user <name>. You must enter the current password and confirm it.
