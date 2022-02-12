
# Radius Server Documentation for mySql 

After finishing to install radius server we will get this table in radius database

`$ sudo mysql -u root -p`

`mariadb[(none)]> use radius;`

`mariadb[radius]> show tables;`

| Tables_in_radius |
|------------------|
| nas              |
| radacct          |
| radcheck         |
| radgroupcheck    |
| radgroupreply    |
| radpostauth      |
| radreply         |
| radusergroup     |

### NAS(Network Access Server)
nas table is used for connecting network device in radius server, different kind of service can be added in radius,
Ex: Cisco, Huawei, Juniper , MikroTik and others..
Here we used added MikroTik as a NAS device in this radius server.
Definitions of RADIUS clients. These are the IP addresses and shared secrets of NAS (Network Access Servers) 
that will be talking to FreeRADIUS. This contains the same basic information as the clients.conf file. 
These are read in from the SQL database on startup, and provide a convenient way of sharing client definitions across a cluster of FreeRADIUS instances.

`Radius Server IP` : 192.168.68.124

`MikroTik Router IP` : 192.168.68.117

`secret`: 123456

- Make Sure radius server's ip and this secret is added in Mikrotik and AAA authenthication is enable for your user

`MariaDB [radius]> insert into nas (nasname,shortname,type,ports,secret,server,community,description) values('192.168.68.117', 'mikrotik-client', 'other', NULL,'123456′,NULL,NULL,'MikroTik Client Router');`

`MariaDB [radius]> SELECT * FROM nas;`

| id | nasname        | shortname       | type  | ports | secret | server | community | description            |
|----|----------------|-----------------|-------|-------|--------|--------|-----------|------------------------|
|  1 | 192.168.68.117 | mikrotik-client | other |  NULL | 123456 | NULL   | NULL      | MikroTik Client Router |

So, here we can add as many nas device as we want, by giving this sql insert command we can add any nas device.

### radcheck

This has identical logic to the first line of a user's file entry. It has check and control pairs which are differentiated by the operator used. 
If the check pairs match then radreply is queried for reply pairs to add to the reply list. 
Both radcheck and radreply are indexed by user by default (it can be altered). 
Only one set of radcheck/radreply items is permitted per index value.

`MariaDB [radius]> SELECT * FROM radcheck;`

| id | username | attribute          | op | value        |
|----|----------|--------------------|----|--------------|
|  3 | bob      | Cleartext-Password | := | passme       |
|  4 | alice    | Cleartext-Password | := | passme       |
|  5 | tom      | Cleartext-Password | := | passme       |

`MariaDB [radius]> insert into radcheck (username,attribute,op,value) values("bappe", "Cleartext-Password", ":=", "bappe");`

`MariaDB [radius]> select * from radcheck;`

| id | username | attribute          | op | value        |
|----|----------|--------------------|----|--------------|
|  3 | bob      | Cleartext-Password | := | passme       |
|  4 | alice    | Cleartext-Password | := | passme       |
|  5 | tom      | Cleartext-Password | := | passme       |
|  6 | bappe    | Cleartext-Password | := | bappe        |

In radcheck many user info we can as user attribute prespective like we can defined user-profile for b/w allocation.
here 3 user's profile added.

`MariaDB [radius]> insert into radcheck (username,attribute,op,value) values("bob", "User-Profile", ":=", "512k_Profile");`

`MariaDB [radius]> insert into radcheck (username,attribute,op,value) values("alice", "User-Profile", ":=", "1M_Profile");`

`MariaDB [radius]> insert into radcheck (username,attribute,op,value) values("tom", "User-Profile", ":=", "2M_Profile");`

`MariaDB [radius]> select * from radcheck;`

| id | username | attribute          | op | value        |
|----|----------|--------------------|----|--------------|
|  3 | bob      | Cleartext-Password | := | passme       |
|  4 | alice    | Cleartext-Password | := | passme       |
|  5 | tom      | Cleartext-Password | := | passme       |
|  6 | bappe    | Cleartext-Password | := | bappe        |
|  7 | bob      | User-Profile       | := | 512k_Profile |
|  8 | alice    | User-Profile       | := | 1M_Profile   |
|  9 | tom      | User-Profile       | := | 2M_Profile   |

### radgroupcheck
Identical to radcheck but indexed by group instead of user.

`MariaDB [radius]> SELECT * FROM radgroupcheck;`

| id | groupname | attribute       | op | value |
|----|-----------|-----------------|----|-------|
|  1 | 512k      | Framed-Protocol | == | PPP   |
|  2 | 1M        | Framed-Protocol | == | PPP   |
|  3 | 2M        | Framed-Protocol | == | PPP   |

we just pointed every groupname with their ppp Protocol.

### radgroupreply
Identical to radreply but indexed by group instead of user.
In previous table we decleared the group name here we also Framed-pool to get ip from MikroTik pool.

`MariaDB [radius]> SELECT * FROM radgroupreply;`

| id | groupname   | attribute           | op | value                           |
|----|-------------|---------------------|----|---------------------------------|
|  4 | 512k        | Framed-Pool         | =  | 512k_pool                       |
|  5 | 1M          | Framed-Pool         | =  | 1M_pool                         |
|  6 | 2M          | Framed-Pool         | =  | 2M_pool                         |

we can also limit by defining rate limit of b/w using another attribute named Mikrotik-Rate-Limit

`MariaDB [radius]> insert into radgroupreply (groupname,attribute,op,value) values ("512k","Mikrotik-Rate-Limit","=","512k/512k 1M/1M 512k/512k 40/40");`

`MariaDB [radius]> insert into radgroupreply (groupname,attribute,op,value) values ("1M","Mikrotik-Rate-Limit","=","1M/1M 2M/2M 1M/1M 40/40");`

`MariaDB [radius]> insert into radgroupreply (groupname,attribute,op,value) values ("2M","Mikrotik-Rate-Limit","=","2M/2M 4M/4M 2M/2M 40/40");`

In MikroTik, that values will be found.

`MariaDB [radius]> SELECT * FROM radgroupreply;`

| id | groupname   | attribute           | op | value                           |
|----|-------------|---------------------|----|---------------------------------|
|  4 | 512k        | Framed-Pool         | =  | 512k_pool                       |
|  5 | 1M          | Framed-Pool         | =  | 1M_pool                         |
|  6 | 2M          | Framed-Pool         | =  | 2M_pool                         |
|  7 | 512k        | Mikrotik-Rate-Limit | =  | 512k/512k 1M/1M 512k/512k 40/40 |
|  8 | 1M          | Mikrotik-Rate-Limit | =  | 1M/1M 2M/2M 1M/1M 40/40         |
|  9 | 2M          | Mikrotik-Rate-Limit | =  | 2M/2M 4M/4M 2M/2M 40/40         |

### radpostauth 
This contains a record of the result of an authentication attempt. Usually it will hold the username, 
when the authentication attempt was made, and whether authentication was successful or not. 
Many people customize this to record extra information. 
Some even record incorrect passwords to help helpdesk users (though this is not recommended).

| id | username | pass     | reply         | authdate            |
|----|----------|----------|---------------|---------------------|
|  1 | bob      | passme   | Access-Accept | 2022-02-07 15:38:40 |
|  2 | alice    | passme   | Access-Accept | 2022-02-07 15:46:28 |
|  3 | tom      | passme   | Access-Accept | 2022-02-07 16:10:22 |
|  4 | bappe    | bappe    | Access-Accept | 2022-02-07 17:07:28 |

### radreply
Contains reply pairs to add if the radcheck check items matched.

`MariaDB [radius]> SELECT * FROM radreply;`

| id | username | attribute     | op            | value               |
|----|----------|---------------|---------------|---------------------|
|    |          |               |               |                     |

no data is matched that's why row is empty

### radusergroup
Contains mappings between users and groups. If a mapping exists for a user, the radgroupcheck 
and radgroupreply tables will be queried with the same logic as radcheck and radreply, the main difference being that check 
and reply items are being looked up for the group and not the user. This is also used when the SQL-Group attribute is used in a condition. 
SQL-Group is a magic attribute which results in radusergroup being queried, evaluating to true if the user is a member of a group, 
and false if they are not. groupname column is related with other table like radgroupreply and radcheck.

`MariaDB [radius]> insert into radusergroup (username,groupname,priority) values ("512k_Profile","512k",10);`

`MariaDB [radius]> insert into radusergroup (username,groupname,priority) values ("1M_Profile","1M",10);`

`MariaDB [radius]> insert into radusergroup (username,groupname,priority) values ("2M_Profile","2M",10);`

`MariaDB [radius]> SELECT * FROM radreply;`

| username     | groupname   | priority |
|--------------|-------------|----------|
| 512k_Profile | 512k        |       10 |
| 1M_Profile   | 1M          |       10 |
| 2M_Profile   | 2M          |       10 |


### radacct

A radacct row is inserted automatically by radius foreach user session.
The session information is sent by the NAS when the user logs out.
This stores RADIUS accounting data. RADIUS accounting data describes a service session. 
The session usually starts immediately after authentication, and ends when the user either disconnects 
from the network service (disassociates from the wireless access point, pulls out the ethernet cable etc...),
or is administratively terminated via a PoD Packet of Disconnect, or a session timer (see the Session-Timeout attribute). 
This table stores the username, connection point, and traffic stats for the session, as well as many other fields. 
Not all columns will be useful for all deployments, so its a good idea to remove ones that you're not using, 
and update the queries appropriately. Only one row is created per session, and is updated over the lifetime of that session.

`MariaDB [radius]> SELECT * FROM radacct;`

| radacctid | acctsessionid | acctuniqueid                     | username | realm | nasipaddress   | nasportid | nasporttype | acctstarttime       | acctupdatetime      | acctstoptime        | acctinterval | acctsessiontime | acctauthentic | connectinfo_start | connectinfo_stop | acctinputoctets | acctoutputoctets | calledstationid | callingstationid  | acctterminatecause | servicetype | framedprotocol | framedipaddress |
|-----------|---------------|----------------------------------|----------|-------|----------------|-----------|-------------|---------------------|---------------------|---------------------|--------------|-----------------|---------------|-------------------|------------------|-----------------|------------------|-----------------|-------------------|--------------------|-------------|----------------|-----------------|
|         1 | 84000002      | eb38aadd264e1badd1c248f5c76ea6aa | bob      |       | 192.168.68.117 |           |             | 2022-02-09 12:53:09 | 2022-02-09 12:53:09 | 2022-02-09 12:53:43 |         NULL |              34 |               |                   |                  |               0 |                0 |                 | 50:E0:85:E6:52:4E | User-Request       | Login-User  |                |                 |
|         2 | 84000003      | 7a2089bc9e0c2b1127e7e88789c2e755 | bob      |       | 192.168.68.117 |           |             | 2022-02-08 18:18:13 | 2022-02-08 18:18:13 | 2022-02-08 18:33:52 |         NULL |             938 |               |                   |                  |               0 |                0 |                 | F8:B4:6A:C1:71:78 | User-Request       | Login-User  |                |                 |
|         3 | 81000004      | 1ff3443d272d276937ce49e1212c0503 | bob      |       | 192.168.68.117 | bridge1   | Ethernet    | 2022-02-08 14:42:57 | 2022-02-08 14:42:57 | 2022-02-08 14:44:07 |         NULL |              70 | RADIUS        |                   |                  |            2958 |              144 | service1        | F8:B4:6A:C1:71:78 | User-Request       | Framed-User | PPP            | 0.0.0.0         |
|         4 | 81000005      | 40e3e8c2bf90fdf9240175370ce34ef7 | bob      |       | 192.168.68.117 | bridge1   | Ethernet    | 2022-02-08 15:13:48 | 2022-02-08 15:13:48 | 2022-02-08 15:14:56 |         NULL |              69 | RADIUS        |                   |                  |            7002 |              102 | service1        | 18:31:BF:B9:4F:D0 | User-Request       | Framed-User | PPP            | 0.0.0.0         |
|         5 | 81000006      | fe8b55cb555fac9ebdf75749ff6ac7c7 | bob      |       | 192.168.68.117 | bridge1   | Ethernet    | 2022-02-08 15:15:22 | 2022-02-08 15:15:22 | 2022-02-08 15:16:29 |         NULL |              67 | RADIUS        |                   |                  |            7122 |              102 | service1        | 18:31:BF:B9:4F:D0 | User-Request       | Framed-User | PPP            | 0.0.0.0         |
|         6 | 81000008      | 80a04ca88ebe9ea95793117f71e9800a | bob      |       | 192.168.68.117 | bridge1   | Ethernet    | 2022-02-08 15:36:15 | 2022-02-08 15:36:15 | 2022-02-08 15:36:20 |         NULL |               4 | RADIUS        |                   |                  |            1449 |              102 | service1        | 18:31:BF:B9:4F:D0 | User-Request       | Framed-User | PPP            | 0.0.0.0         |
|         7 | 84000001      | 2cb3282e5c967d38aedf6351795b66b3 | imran    |       | 192.168.68.117 |           |             | 2022-02-08 16:17:45 | 2022-02-08 16:17:45 | 2022-02-08 16:18:05 |         NULL |              15 |               |                   |                  |               0 |                0 |                 | 192.168.68.133    | User-Request       | Login-User  |                |                 |
|         9 | 8100000a      | 6f9ba70ccd13120b90cc4fe844f16b07 | bob      |       | 192.168.68.117 | bridge1   | Ethernet    | 2022-02-08 16:54:30 | 2022-02-08 16:54:30 | 2022-02-08 18:15:40 |         NULL |            4869 | RADIUS        |                   |                  |         4164618 |          7308659 | service1        | 18:31:BF:B9:4F:D0 | Lost-Carrier       | Framed-User | PPP            | 10.10.10.251    |
|        11 | 8100000b      | a906179bf7b0c050f7700ee74422cd5d | bob      |       | 192.168.68.117 | bridge1   | Ethernet    | 2022-02-08 18:24:14 | 2022-02-08 18:24:14 | 2022-02-08 18:24:39 |         NULL |              26 | RADIUS        |                   |                  |             415 |              210 | service1        | 18:31:BF:B9:4F:D0 | Lost-Carrier       | Framed-User | PPP            | 10.10.10.251    |
|        12 | 8100000c      | c5612b5f14e2fca83df2a01a25e9a15b | tom      |       | 192.168.68.117 | bridge1   | Ethernet    | 2022-02-08 18:26:00 | 2022-02-08 18:26:00 | 2022-02-08 18:26:56 |         NULL |              56 | RADIUS        |                   |                  |           23071 |            56427 | service1        | 18:31:BF:B9:4F:D0 | Lost-Carrier       | Framed-User | PPP            | 30.30.30.252    |
|        13 | 8100000d      | 4f25b18be3bc344cf76190240567509a | tom      |       | 192.168.68.117 | bridge1   | Ethernet    | 2022-02-08 18:27:51 | 2022-02-08 18:27:51 | 2022-02-08 19:05:28 |         NULL |            2257 | RADIUS        |                   |                  |        18376613 |         17268626 | service1        | 18:31:BF:B9:4F:D0 | Lost-Carrier       | Framed-User | PPP            | 30.30.30.252    |
|        15 | 84000001      | 89a6b67e533fad6158a0d4a83a187d1c | bob      |       | 192.168.68.117 |           |             | 2022-02-09 13:03:26 | 2022-02-09 13:03:26 | NULL                |         NULL |               0 |               |                   |                  |               0 |                0 |                 | 50:E0:85:E6:52:4E |                    | Login-User  |                |                 |



### extra info

The configuration files for rlm_sql are split. There's the main config file in mods-available/sql, and then the SQL dialect specific one in mods-config/sql/main/<dialect>/queries.conf

mods-available/sql is where you set connection parameters (IP address of the server, port, credentials, database flavour).
queries.conf is where you can alter the default queries and index values.

## users operators
> **How the users file works**
pairs is referring to Attribute Value Pairs (AVPs), that is, a tuple consisting of an attribute an operator and a value.
There are three lists of attribute(s) (pairs) that are accessible from the users file. These are bound to the request the server is currently processing, and they don't persist across multiple request/response rounds.

- request - Contains all the pairs from the original request received from the NAS (Network Access Server) via the network.
- control - Initially contains no pairs, but is populated with pairs that control how modules process the current request. This is done from the users file or unlang (the FreeRADIUS policy language used in virtual servers).
- reply - Contains pairs you want to send back to the NAS via the network.

The users file module determines the list it's going to use for inserting/searching, by where the pair is listed in the entry, and the operator.
The first line of the entry contains check pairs that must match in order for the entry to be used. The first line also contains control pairs, those you want to be inserted into the control list if all the check pairs match.
Note: It doesn't matter which order the pairs are listed in. control pairs will not be inserted unless all the check pairs evaluate to true.
check and control pairs are distinguished by the operator used. If an assignment operator is used i.e. ':=' or '=' then the pair will be treated as a control pair. If an equality operator such as '>', '<', '==', '>=', '<=', '=~' is used, the pair will be treated as a check pair.
Subsequent lines in the same entry contain only reply pairs. If all check pairs match, reply pairs will be inserted into the reply list

> **Cleartext-Password**
Cleartext-Password is strictly a control pair. It should not be present in any of the other lists.
Cleartext-Password is one of a set of attributes, which should contain the 'reference' (or 'known good') password, that is, the local copy of the users password. An example of another pair in this set is SSHA-Password - this contains a salted SHA hash of the users password.
The reference password pairs are searched for (in the control list) by modules capable of with authenticating users using the 'User-Password' pair. In this case that module is 'rlm_pap'.

> **User-Password**
User-Password is strictly a request pair. It should not be present in any of the other lists.
User-Password is included in the request from the NAS. It contains the plaintext version of the password the user provided to the NAS. In order to authenticate a user, a module needs to compare the contents of User-Password with a control pair like Cleartext-Password.

In a users file entry when setting reference passwords you'll see entries like:

`my_username Cleartext-Password := "known_good_password"`

That is, if the username matches the value on the left (my_username), then insert the control pair Cleartext-Password with the value "known_good_password".

To answer the first question the reason why:

`shad Cleartext-Password == "test"`

Does not work, it is because you are telling the files module to search in the request list, for a pair which does not exist in the request list, and should never exist in the request list.
You might now be thinking oh, i'll use User-Password == "test" instead, and it'll work. Unfortunately it won't. If the password matches then the entry will match, but the user will still be rejected, see below for why.

**Auth-Type**

Auth-Type is strictly a control pair. It should not be present in any of the other lists.
There are three main sections in the server for dealing with requests 'authorize', 'authenticate', 'post-auth'.
authorize is the information gathering section. This is where database lookups are done to authorise the user, and to retrieve reference passwords. It's also where Auth-Type is determined, that is, the type of authentication we want to perform for the user.
Authenticate is where a specific module is called to perform authentication. The module is determined by Auth-Type.
Post-Auth is mainly for logging, and applying further policies, the modules run in Post-Auth are determined by the response returned from the module run in Authenticate.
The modules in authorize examine the request, and if they think they can authenticate the user, and Auth-Type is not set, they set it to themselves. For example the rlm_pap module will set Auth-Type = 'pap' if it finds the User-Password in the request.
If no Auth-Type is set the request will be rejected.
So to answer your second question, you're forcing pap authentication, which is wrong, you should let rlm_pap set the Auth-Type by listing pap after the files module in the authorize section.
When rlm_pap runs in authenticate, it looks for a member of the set of 'reference' passwords described above, and if it can't find one, it rejects the request, this is what's happening above.
There's also a 'magic' Auth-Type, 'Accept', which skips the authenticate section completely and just accepts the user. If you want the used to do cleartext password comparison without rlm_pap, you can use:

`shad Auth-Type := Accept, User-Password == "test"`

### Attribiutes and values list

https://wiki.mikrotik.com/wiki/Manual:RADIUS_Client/reference_dictionary

### Commands to mikrotik connect and shell execution

create a pool

`[admin@MikroTik] > ip/pool/add name=extra-pool ranges=80.80.80.2-80.80.80.254`

 for getting pool list 
  
 `[admin@MikroTik] > ip/pool/print`
 
 add radius server in Mikrotik
 
 `[admin@MikroTik] > /radius add service=hotspot,ppp address=10.0.0.3 secret=ex`
 
 add pppoe client
 
 `[admin@R1] > /interface pppoe-client
add add-default-route=yes disabled=no interface=ether2 name=Client1 password=test user=test`
  
 Create a new one or update the default ppp profile:

`[admin@R2] > /ppp profile set [find name=default] remote-address=pool1 local-address=192.168.79.1`
 
 Enable user authentication via RADIUS. If entry in local secret database is not found, then client will be authenticated via RADIUS

`[admin@R2] > /ppp aaa set use-radius=yes`
  
