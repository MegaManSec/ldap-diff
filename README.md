# LDAP Diff

This script is a slightly edited version of Nick Urbanik's [ldap-diff](https://nicku.org/software/ldap-diff) perl script which can compare two LDAP directory outputs and perform a diff.

Based on 'orig' (old) and 'target' (new) dumps, the perl script creates an .ldif file which should be the exact same ldif which was executed on the server which created the difference.

The original ldap-diff compared distinguished names in the files to compare results, however it is possible to modify an entry's distinguished name with a modrdn operation.

Therefore, there is a slight change in the code here which defines unique entries based on their entryUUID (which _are_ unique per entry), not their distinguished name, meaning it is able to keep track of changes involving the same entry but with different distinguished names.


## Usage

```
$ ldapsearch -o ldif-wrap=no -x -LLL -H ldaps://ldap.local -b dc=rabbit,dc=com '(&(|(objectClass=inetOrgPerson)(objectClass=groupOfNames)))' '*' '+' > ldap.new
$ sleep 360
$ mv ldap.new ldap.old
$ ldapsearch -o ldif-wrap=no -x -LLL -H ldaps://ldap.local -b dc=rabbit,dc=com '(&(|(objectClass=inetOrgPerson)(objectClass=groupOfNames)))' '*' '+' > ldap.new
$ perl ./ldap-diff  --orig ldap.old --target ldap.new

dn: cn=superadmins,ou=Groups,dc=rabbit,dc=com
changetype: modify
add: memberUid
memberUid: oscarmausser

dn: uid=oscarmausser,ou=People,dc=rabbit,dc=com
changetype: modify
replace: lastLogin
lastLogin: 1700673781
read 102 records from ldap.new, and 102 from ldap.old;
found 2 mods
0 adds
0 deletes
```

The final four lines are stderr, and can be sent to `/dev/null` if necessary.

This may also be used to generate rollback data, in case it is necessary to roll back an ldif. This can be done by replacing the orig (old) ldap dump with the target (new) one.
