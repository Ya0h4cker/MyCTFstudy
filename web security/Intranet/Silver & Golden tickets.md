# Silver & Golden tickets

## Silver ticket

In the kerberos protocol, if a client wants to access a service, he must provider the TS to the server, which is encrypted with the NT hash of the account that is running the service.

Thus, if an attacker manages to extract the password or NT hash of a service account, he can then forge a service ticket (TS) by choosing the information he wants to put in it in order to access that service, without asking the KDC. It is this forged TS that is called silver ticket.

Through silver ticket, we can improve our rights and access designated services, such as WMI, LDAP, CIFS and so on.

## Golden ticket

In the kerberos protocol, if a client wants to access a service, he must provider the TGT to the TGS, which is encrypted with the key's hash of the **krbtgt** account.

Therefore, if an attacker ever manages to find the key's hash of the **krbtgt** account, he will then be able to forge a TGT and ask the KDC for any TS for any service. It is this forged TGT that is called golden ticket.

Through golden ticket, we can obtain the access rights of any Kerberos services and the golden ticket is usually used for rights maintenance.
