
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

# Standard Attributes (defined in RFC 2865, 2866 and 2869)
`
ATTRIBUTE       User-Name                    1    string
ATTRIBUTE       User-Password                2    string  encrypt=1
ATTRIBUTE       Password                     2    string  encrypt=1
ATTRIBUTE       CHAP-Password                3    string
ATTRIBUTE       NAS-IP-Address               4    ipaddr
ATTRIBUTE       NAS-Port                     5    integer
ATTRIBUTE       Service-Type                 6    integer
ATTRIBUTE       Framed-Protocol              7    integer
ATTRIBUTE       Framed-IP-Address            8    ipaddr
ATTRIBUTE       Framed-IP-Netmask            9    ipaddr
ATTRIBUTE       Framed-Routing               10   integer
ATTRIBUTE       Filter-Id                    11   string
ATTRIBUTE       Framed-Mtu                   12   integer
ATTRIBUTE       Framed-Compression           13   integer
ATTRIBUTE       Login-Ip-Host                14   ipaddr
ATTRIBUTE       Login-Service                15   integer
ATTRIBUTE       Login-Port                   16   integer

ATTRIBUTE       Reply-Message                18   string
ATTRIBUTE       Login-Callback-Number        19   string
ATTRIBUTE       Framed-Callback-Id           20   string

ATTRIBUTE       Framed-Route                 22   string
ATTRIBUTE       Framed-Ipx-Network           23   integer
ATTRIBUTE       State                        24   string
ATTRIBUTE       Class                        25   string
ATTRIBUTE       Vendor-Specific              26   string
ATTRIBUTE       Session-Timeout              27   integer
ATTRIBUTE       Idle-Timeout                 28   integer
ATTRIBUTE       Termination-Action           29   integer
ATTRIBUTE       Called-Station-Id            30   string
ATTRIBUTE       Calling-Station-Id           31   string
ATTRIBUTE       NAS-Identifier               32   string
ATTRIBUTE       Proxy-State                  33   string
ATTRIBUTE       Login-Lat-Service            34   string
ATTRIBUTE       Login-Lat-Node               35   string
ATTRIBUTE       Login-Lat-Group              36   string
ATTRIBUTE       Framed-Appletalk-Link        37   integer
ATTRIBUTE       Framed-Appletalk-Network     38   integer
ATTRIBUTE       Framed-Appletalk-Zone        39   string
ATTRIBUTE       Acct-Status-Type             40   integer
ATTRIBUTE       Acct-Delay-Time              41   integer
ATTRIBUTE       Acct-Input-Octets            42   integer
ATTRIBUTE       Acct-Output-Octets           43   integer
ATTRIBUTE       Acct-Session-Id              44   string
ATTRIBUTE       Acct-Authentic               45   integer
ATTRIBUTE       Acct-Session-Time            46   integer
ATTRIBUTE       Acct-Input-Packets           47   integer
ATTRIBUTE       Acct-Output-Packets          48   integer
ATTRIBUTE       Acct-Terminate-Cause         49   integer
ATTRIBUTE       Acct-Input-Gigawords         52   integer
ATTRIBUTE       Acct-Output-Gigawords        53   integer

ATTRIBUTE       Event-Timestamp              55   date

ATTRIBUTE       CHAP-Challenge               60   string
ATTRIBUTE       NAS-Port-Type                61   integer
ATTRIBUTE       Port-Limit                   62   integer

ATTRIBUTE       Eap-Packet                   79   raw
ATTRIBUTE       Message-Authenticator        80   raw

ATTRIBUTE       Acct-Interim-Interval        85   integer
ATTRIBUTE       NAS-Port-Id                  87   string
ATTRIBUTE       Framed-Pool                  88   string
ATTRIBUTE       Chargeable-User-Id           89   string

ATTRIBUTE       Nas-Ipv6-Address             95   addr6
ATTRIBUTE       Framed-Ipv6-Prefix           97   prefix6
ATTRIBUTE       Framed-Ipv6-Pool             100  string
ATTRIBUTE       Error-Cause                  101  integer

ATTRIBUTE       Delegate-Ipv6-Prefix         123  prefix6
ATTRIBUTE       Framed-Ipv6-Address          168  addr6
ATTRIBUTE       Dns-Server-Ipv6-Address      169  addr6
ATTRIBUTE       Delegate-Ipv6-Pool           171  string
`

# FreeRADIUS internal attributes (they can not be transmitted via the RADIUS
# protocol - they are used for internal purposes only)

ATTRIBUTE       Auth-Type                    1000 integer
ATTRIBUTE       Acct-Unique-Session-Id       1051 string
ATTRIBUTE       Client-IP-Address            1052 ipaddr
ATTRIBUTE       SQL-User-Name                1055 string
ATTRIBUTE       NT-Password                  1058 string

# Standard Values

VALUE           Service-Type                 Framed                         2

VALUE           Framed-Protocol              PPP                            1

VALUE           Acct-Status-Type             Start                          1
VALUE           Acct-Status-Type             Stop                           2
VALUE           Acct-Status-Type             Interim-Update                 3

VALUE           Acct-Authentic               RADIUS                         1
VALUE           Acct-Authentic               Local                          2

VALUE           NAS-Port-Type                Async                          0
VALUE           NAS-Port-Type                ISDN-Sync                      2
VALUE           NAS-Port-Type                Virtual                        5
VALUE           NAS-Port-Type                Ethernet                       15
VALUE           NAS-Port-Type                Cable                          17
VALUE           NAS-Port-Type                Wireless-802.11                19

VALUE           Acct-Terminate-Cause         User-Request                   1
VALUE           Acct-Terminate-Cause         Lost-Carrier                   2
VALUE           Acct-Terminate-Cause         Lost-Service                   3
VALUE           Acct-Terminate-Cause         Idle-Timeout                   4
VALUE           Acct-Terminate-Cause         Session-Timeout                5
VALUE           Acct-Terminate-Cause         Admin-Reset                    6
VALUE           Acct-Terminate-Cause         Admin-Reboot                   7
VALUE           Acct-Terminate-Cause         Port-Error                     8
VALUE           Acct-Terminate-Cause         NAS-Error                      9
VALUE           Acct-Terminate-Cause         NAS-Request                    10
VALUE           Acct-Terminate-Cause         NAS-Reboot                     11
VALUE           Acct-Terminate-Cause         Port-Unneeded                  12
VALUE           Acct-Terminate-Cause         Port-Preempted                 13
VALUE           Acct-Terminate-Cause         Port-Suspended                 14
VALUE           Acct-Terminate-Cause         Service-Unavailable            15
VALUE           Acct-Terminate-Cause         Callback                       16
VALUE           Acct-Terminate-Cause         User-Error                     17
VALUE           Acct-Terminate-Cause         Host-Request                   18

VALUE           Auth-Type                    System                         1



# Cisco Attributes

VENDOR          Cisco           9
BEGIN-VENDOR    Cisco
ATTRIBUTE       H323-Remote-Address          23   string
ATTRIBUTE       H323-Connection-Id           24   string
ATTRIBUTE       H323-Setup-Time              25   string
ATTRIBUTE       H323-Call-Direction          26   string
ATTRIBUTE       H323-Call-Type               27   string
ATTRIBUTE       H323-Connect-Time            28   string
ATTRIBUTE       H323-Disconnect-Time         29   string
ATTRIBUTE       H323-Disconnect-Cause        30   integer
ATTRIBUTE       H323-Voice-Quality           31
ATTRIBUTE       H323-Gw-Name                 33   string
ATTRIBUTE       H323-Call-Treatment          34

# Cisco Values
VALUE           H323-Disconnect-Cause        Local-Clear                    0
VALUE           H323-Disconnect-Cause        Local-No-Accept                1
VALUE           H323-Disconnect-Cause        Local-Decline                  2
VALUE           H323-Disconnect-Cause        Remote-Clear                   3
VALUE           H323-Disconnect-Cause        Remote-Refuse                  4
VALUE           H323-Disconnect-Cause        Remote-No-Answer               5
VALUE           H323-Disconnect-Cause        Remote-Caller-Abort            6
VALUE           H323-Disconnect-Cause        Transport-Error                7
VALUE           H323-Disconnect-Cause        Transport-Connect-Fail         8
VALUE           H323-Disconnect-Cause        Gatekeeper-Clear               9
VALUE           H323-Disconnect-Cause        Fail-No-User                   10
VALUE           H323-Disconnect-Cause        Fail-No-Bandwidth              11
VALUE           H323-Disconnect-Cause        No-Common-Capabilities         12
VALUE           H323-Disconnect-Cause        Facility-Forward               13
VALUE           H323-Disconnect-Cause        Fail-Security-Check            14
VALUE           H323-Disconnect-Cause        Local-Busy                     15
VALUE           H323-Disconnect-Cause        Local-Congestion               16
VALUE           H323-Disconnect-Cause        Remote-Busy                    17
VALUE           H323-Disconnect-Cause        Remote-Congestion              18
VALUE           H323-Disconnect-Cause        Remote-Unreachable             19
VALUE           H323-Disconnect-Cause        Remote-No-Endpoint             20
VALUE           H323-Disconnect-Cause        Remote-Off-Line                21
VALUE           H323-Disconnect-Cause        Remote-Temporary-Error         22

END-VENDOR    Cisco


# DHCP Attributes
VENDOR          DHCP       54
BEGIN-VENDOR    DHCP
ATTRIBUTE       DHCP-Classless-Static-Route  121  string
END-VENDOR      DHCP


# Microsoft Attributes (defined in RFC 2548)
VENDOR          Microsoft       311
BEGIN-VENDOR    Microsoft

ATTRIBUTE       MS-CHAP-Response             1    string
ATTRIBUTE       MS-MPPE-Encryption-Policy    7    string
ATTRIBUTE       MS-MPPE-Encryption-Types     8    string
ATTRIBUTE       MS-CHAP-Domain               10   string
ATTRIBUTE       MS-CHAP-Challenge            11   string
ATTRIBUTE       MS-CHAP-Mppe_Keys            12   string
ATTRIBUTE       MS-MPPE-Send-Key             16   string  encrypt=2
ATTRIBUTE       MS-MPPE-Recv-Key             17   string  encrypt=2 
ATTRIBUTE       MS-CHAP2-Response            25   string
ATTRIBUTE       MS-CHAP2-Success             26   string
ATTRIBUTE       MS-Primary-Dns-Server        28   ipaddr
ATTRIBUTE       MS-Secondary-Dns-Server      29   ipaddr
ATTRIBUTE       MS-Primary-Nbns-Server       30   ipaddr
ATTRIBUTE       MS-Secondary-Nbns-Server     31   ipaddr

END-VENDOR    Microsoft


# Ascend Attributes
VENDOR          Ascend          529
BEGIN-VENDOR    Ascend

ATTRIBUTE       Ascend-Client-Gateway       132   ipaddr
ATTRIBUTE       Ascend-Data-Rate            197   integer
ATTRIBUTE       Ascend-Xmit-Rate            255   integer
END-VENDOR    Ascend


# Redback Attributes
VENDOR          Redback       2352
BEGIN-VENDOR    Redback
ATTRIBUTE       Redback-Agent-Remote-Id     96  string
ATTRIBUTE       Redback-Agent-Circuit-Id    97  string
END-VENDOR      Redback


# ADSL Attributes
VENDOR          ADSL       3561
BEGIN-VENDOR    ADSL
ATTRIBUTE       ADSL-Agent-Circuit-Id                       1  string
ATTRIBUTE       ADSL-Agent-Remote-Id                        2  string
ATTRIBUTE       ADSL-Actual-Data-Rate-Upstream              0x81  integer
ATTRIBUTE       ADSL-Actual-Data-Rate-Downstream            0x82  integer
ATTRIBUTE       ADSL-Minimum-Data-Rate-Upstream             0x83  integer
ATTRIBUTE       ADSL-Minimum-Data-Rate-Downstream           0x84  integer
ATTRIBUTE       ADSL-Attainable-Data-Rate-Upstream          0x85  integer
ATTRIBUTE       ADSL-Attainable-Data-Rate-Downstream        0x86  integer
ATTRIBUTE       ADSL-Max-Data-Rate-Upstream                 0x87  integer
ATTRIBUTE       ADSL-Max-Data-Rate-Downstream               0x88  integer
ATTRIBUTE       ADSL-Min-Data-Rate-Upstream                 0x89  integer
ATTRIBUTE       ADSL-Min-Data-Rate-Downstream               0x8a  integer
ATTRIBUTE       ADSL-Max-Interleaving-Delay-Upstream        0x8b  integer
ATTRIBUTE       ADSL-Actual-Interleaving-Delay-Upstream     0x8c  integer
ATTRIBUTE       ADSL-Max-Interleaving-Delay-Downstream      0x8d  integer
ATTRIBUTE       ADSL-Actual-Interleaving-Delay-Downstream   0x8e  integer
ATTRIBUTE       ADSL-Encap                                  0x90
END-VENDOR      ADSL



# WISPr Attributes (defined in httpwww.wi-fi.orggetfile.aspf=WISPr_V1.0.pdf)
VENDOR          WISPr           14122
BEGIN-VENDOR    WISPr

ATTRIBUTE       WISPr-Location-Id                  1    string
ATTRIBUTE       WISPr-Location-Name                2    string
ATTRIBUTE       WISPr-Logoff-URL                   3    string
ATTRIBUTE       WISPr-Redirection-URL              4    string
ATTRIBUTE       WISPr-Bandwidth-Min-Up             5    integer
ATTRIBUTE       WISPr-Bandwidth-Min-Down           6    integer
ATTRIBUTE       WISPr-Bandwidth-Max-Up             7    integer
ATTRIBUTE       WISPr-Bandwidth-Max-Down           8    integer
ATTRIBUTE       WISPr-Session-Terminate-Time       9    string
ATTRIBUTE       WISPr-Session-Terminate-End-Of-Day 10   string
ATTRIBUTE       WISPr-Billing-Class-Of-Service     11   string

END-VENDOR      WISPr


# MikroTik Attributes
VENDOR          Mikrotik        14988
BEGIN-VENDOR    Mikrotik

ATTRIBUTE       Mikrotik-Recv-Limit             1   integer
ATTRIBUTE       Mikrotik-Xmit-Limit             2   integer
ATTRIBUTE       Mikrotik-Group                  3   string  
ATTRIBUTE       Mikrotik-Wireless-Forward       4   integer
ATTRIBUTE       Mikrotik-Wireless-Skip-Dot1x    5   integer
ATTRIBUTE       Mikrotik-Wireless-Enc-Algo      6   integer
ATTRIBUTE       Mikrotik-Wireless-Enc-Key       7   string
ATTRIBUTE       Mikrotik-Rate-Limit             8   string
ATTRIBUTE       Mikrotik-Realm                  9   string
ATTRIBUTE       Mikrotik-Host-IP                10  ipaddr
ATTRIBUTE       Mikrotik-Mark-Id                11  string
ATTRIBUTE       Mikrotik-Advertise-URL          12  string
ATTRIBUTE       Mikrotik-Advertise-Interval     13  integer
ATTRIBUTE       Mikrotik-Recv-Limit-Gigawords   14  integer
ATTRIBUTE       Mikrotik-Xmit-Limit-Gigawords   15  integer
ATTRIBUTE       Mikrotik-Wireless-PSK           16  string
ATTRIBUTE       Mikrotik-Total-Limit            17  integer
ATTRIBUTE       Mikrotik-Total-Limit-Gigawords  18  integer
ATTRIBUTE       Mikrotik-Address-List           19  string
ATTRIBUTE       Mikrotik-Wireless-MPKey         20  string
ATTRIBUTE       Mikrotik-Wireless-Comment       21  string
ATTRIBUTE       Mikrotik-Delegated-IPv6-Pool    22  string
ATTRIBUTE       Mikrotik-DHCP-Option-Set        23  string
ATTRIBUTE       Mikrotik-DHCP-Option-Param_STR1 24  string
ATTRIBUTE       Mikrotik-DHCP-Option-Param_STR2 25  string
ATTRIBUTE       Mikrotik-Wireless-VLANID        26  integer
ATTRIBUTE       Mikrotik-Wireless-VLANIDtype    27  integer
ATTRIBUTE       Mikrotik-Wireless-Minsignal     28  string
ATTRIBUTE       Mikrotik-Wireless-Maxsignal     29  string
ATTRIBUTE       Mikrotik-Switching-Filter       30  string

# MikroTik Values

VALUE           Mikrotik-Wireless-Enc-Algo            No-encryption                  0
VALUE           Mikrotik-Wireless-Enc-Algo            40-bit-WEP                     1
VALUE           Mikrotik-Wireless-Enc-Algo            104-bit-WEP                    2
VALUE           Mikrotik-Wireless-Enc-Algo            AES-CCM                        3
VALUE           Mikrotik-Wireless-Enc-Algo            TKIP                           4 
VALUE           Mikrotik-Wireless-VLANIDtype          802.1q                         0
VALUE           Mikrotik-Wireless-VLANIDtype          802.1ad                        1

END-VENDOR      Mikrotik

### Commands to mikrotik connect and shell execution

create a pool

`[admin@MikroTik] > ip/pool/add name=extra-pool ranges=80.80.80.2-80.80.80.254`

 for getting pool list 
  
 `[admin@MikroTik] > ip/pool/print`
  
  
