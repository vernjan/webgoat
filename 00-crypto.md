# Crypto Basics

## Lesson 6

### Get RSA modulus
```
$ openssl rsa -in data/00-crypto/id_rsa -modulus -noout
Modulus=AAA5B2DF11A0DA89EEAAE169FE3FC4D5DFF0266B6C792BE7BEE968323AEA691CA9748870332E4260DFE91204C03A0B4236472CC519143035E9DC660C4E405B3FCBC189D1C27E17141F12269C883680E68BDD276EAD4219EA11D3E7D31FD5867B5D79BC145F09AC1624E22AEB530248BACDCDB13D3EA203A90EF3A5B97A602DDF4CBAC707CDAEB3A74EC38672C3BC6C4AA430C0D462FF1CE569534A9A6AE5639BEB12936E205C8F7FDDE0573DE368F6D8663681E31E9752BB4D420CD27AF43612A0BD7E52B6608724A52C7576B3765EBB1DFA588A8C067631E05A4868D03D837340B811031DC762D574E54924C0950CBA77C0D8FD39161F2D986D6023DB0645F7
```

### Sign RSA modulus
The assignment description is incomplete here, you must look into source code to get the details:
- Use SHA256
- Signature is expected _base64_ encoded

```
$ openssl dgst -sha256 -sign data/00-crypto/id_rsa data/00-crypto/rsa_modulus.txt | base64 --wrap=0
LK/k8SeLA2QanwTkRcrl4eoryraxSmrYk2t88ndZ9YhMZQmjTpGkq+HF19vRNQg3LQthlvJLL+Z0IGBYrHqnnq6cc8/JMPeTL6dco/V0QHqqea417G3sMKOeLRFQQJWRXTDwSMq86yFp4ytyphyB+vkj534BwNnFKPNl+aCSdjCfF8GSQeXRhuUH7Rw7QbuarL/59NXdrxFqkzY5fbNUyEvCJuxKB+ZwC1jN/8oF0/YAtsuVPKz2MfjtSMWN5F0OZ2XVq7L7jZGULY8ZL3tLBDPpIaDm9VFsNM/haPV2NYgkSJzevY0LAbgr3DdBA1Lxa3UykmCL0EnhrIOOriKcfA==
```

## Lesson 8
The secret file is located in `/root` but our container runs with `webgoat` user.
However, it's very simple to change to `root` even without knowing the root's password:
```
$ docker run -d webgoat/assignments:findthesecret
473372d0c71d8bdb9cf97f7e95082183016683e0fbd963608a977a650b3fc068

$ docker exec -ti --user 0 473372d0c71d bash

root@473372d0c71d:/# cat /root/default_secret
ThisIsMySecretPassw0rdF0rY0u

root@473372d0c71d:/# echo "U2FsdGVkX199jgh5oANElFdtCxIEvdEvciLi+v+5loE+VCuy6Ii0b+5byb5DXp32RPmT02Ek1pf55ctQN+DHbwCPiVRfFQamDmbHBUpD7as=" | openssl enc -aes-256-cbc -d -a -kfile /root/default_secret
Leaving passwords in docker images is not so secure
```