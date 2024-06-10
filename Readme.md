## Creating example LDAP server via OpenLDAP

Here is the guidline to create a LDAP server for testing purpose:

If you're using Public Cloud providers, might be better idea to use Private IP address:

```
#for example:
{
    PRIVATE_IP=10.0.1.6
    LDAP_FQDN=ldap-server.example.local
    echo "$PRIVATE_IP $LDAP_FQDN" >> /etc/hosts
}
```

#### Install OpenLDAP package:

Here in this steps, it will prompt you for configuring LDAP administrator password.

```
{
    apt update
    apt install slapd ldap-utils
}
```

Now and after installation has finished, we need to configure LDAP server:

```
dpkg-reconfigure slapd
```
![alt text](images/image.png)

Make sure to click `No` and proceed.

At the next step we need to configure a DNS name for the LDAP base DN:

![alt text](images/image-1.png)

In our example it would be `example.local`

And also it's same for the next step as `Organization`

after providing the administrator password, select `No` when it prompted for purging the db:

![alt text](images/image-2.png)

And finally select `Yes` for moving the old DB:

![alt text](images/image-3.png)


#### Configuring OpenLDAP config file:

Now it's time to reconfigure LDAP config file to use the new information we just configured:

```
{
    vi /etc/ldap/ldap.conf
}
```
#Uncomment and provide the correct info:

`BASE    dc=example,dc=local`

`URI     ldap://ldap-server.example.local`

```
{
    systemctl restart slapd
    systemctl status slapd
}
```

You can check and verfy the LDAP configuration by this command:

```
{
    ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:///
}
```

#### Creating two main OUs for storing other groups and users:

Run below command to add OUs to the LDAP:

```
ldapadd -x -D cn=admin,dc=example,dc=local -W -f organizationalUnits.ldif
```

#### Creating some random groups for storing users:

Run below command to add groups to the LDAP:

```
ldapadd -x -D cn=admin,dc=example,dc=local -W -f groups.ldif
```

#### Creating some random users:

First we need to generate a password for users, so you can run this command to generate a new one and replace the while encrypted result with password `userPassword` in users.ldif file.

```
slappasswd
```

Next, run below command to add users to the LDAP:

```
ldapadd -x -D cn=admin,dc=example,dc=local -W -f users.ldif
```

#### Verifying all created objects.

Now you can run below commands to verify all the objects we just created:

```
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:///
```
```
ldapsearch -D "cn=admin,dc=example,dc=local" -W -b "OU=users,DC=example,DC=local"
```

```
ldapsearch -x -LLL -b dc=example,dc=local '(uid=Charlotte)' cn uidNumber gidNumber
```