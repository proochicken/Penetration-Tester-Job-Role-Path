- Nmap:
```
22/tcp  open  ssh
110/tcp open  pop3
143/tcp open  imap
993/tcp open  imaps
995/tcp open  pop3s
```
- Nmap UDP:
```
port 161 open SNMP
```
- Dùng `onesixtyone` brute-force community string
- Dùng `snmpwalk` --> Lấy được credential `tom NMds732Js2761` cho imaps
- Vô `imap` với credential trên, lấy được private-key
