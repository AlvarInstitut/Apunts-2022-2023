---
typora-copy-images-to: ../assets/img/LlIntTallers/
typora-root-url: ../../
layout: post
title:  Llevar internet dels tallers
categories: parte1
conToc: true
permalink: llevar-internet-dels-tallers

---

### Regles firewall Mikrotik

Per a poder llevar internet dels tallers hi ha més d'una possibilitat. Una d'elles serà posant regles en el router Mikrotik, concretament les reglas van en:

**IP -> Firewall**

![image-20220513101347564](/manteniment/assets/img/LlIntTallers/image-20220513101347564.png)

En les regles podem posar l'acció (accept - reject) adreça d'origen (qui fa la sol·licitud), adreça de destí i interface d'eixida. Mikrotik, quan troba la primera regla que fa match la segueix, per tant només es tracta de posar el que volem per ordre.

### Llista d'adreces permeses

Per a no haver d'anar activant cadascuna de les adreces permeses, farem una llista. Es fa en **IP -> Firewall -> Address Lists**

![image-20220513101422257](/manteniment/assets/img/LlIntTallers/image-20220513101422257.png)

Com veiem de moment estan:

- **89.36.214.106**: És el **servidor de dades** on tenim instal·lat PostgreSQL, MySQL, MongoDB, Redis, eXist-db
- **172.27.111.20**: és **aules.edu.gva.es**
- **195.77.19.153**: és **portal.edu.gva.es**, que li fa falta aules

Per a afegir alguna adreças, hem de recordar de posar-li el nom **AdrecesPermeses2** (va ser la segona prova).

![image-20220513101436391](/manteniment/assets/img/LlIntTallers/image-20220513101436391.png)

### Regles per a cada taller

La idea seria per ordre:

- Permetre a l'equip del professorat accedir a tot
- Permetre a tot el taller accedir a una sèrie d'adreces: Aules, Servidor de dades...
- Rebutjar la resta d'accessos a Internet.

Si **activem** aquestes regles, impedirem l'accés a Internet (excepte la llista d'adreces permeses). Si **desactivem** aquestes regles, tindran accés normal a Internet.

Per tant en **IP -> Firewall -> Filter Rules**

![image-20220513101456752](/manteniment/assets/img/LlIntTallers/image-20220513101456752.png)



#### Permetre a l'equip del professorat accedir a tot

<img src="/manteniment/assets/img/LlIntTallers/image-20220513182542325.png" alt="image-20220513182542325" style="zoom:80%;" />

#### Permetre a tot el taller accedir a la llista d'adreces permeses

<img src="/manteniment/assets/img/LlIntTallers/image-20220513184424989.png" alt="image-20220513184424989" style="zoom:80%;" />

#### Rebutjar la resta d'accessos a Internet

<img src="/manteniment/assets/img/LlIntTallers/image-20220513184037006.png" alt="image-20220513184037006" style="zoom:80%;" />



#### Regles de les Aules d'Informàtica

Les Aules d'Informàtica tenen un comportament diferent, ja que les 2 aules comparteixen la mateixa VLAN.

Afortunadament les adreces són:

- Aula 1: de la 192.168.209.101 fins la 192.168.209.121, totes elles inferiors a 128
- Aula 2; de la 192.168.209.201 fins la 192.168.209.221, totes elles superiors a 128

Per tant podem discriminar posant una màscara de **25 bits**



### Creació d'un usuari que execute els scripts

Millor no execfutar com a admin els scripts que crearem per a habilitar-deshabilitar les regles. Per això creem un usuari que no tinga tantos permisos. En **System -> Users** 

![image-20220513134030181](/manteniment/assets/img/LlIntTallers/image-20220513134030181.png)

L'usuari és **user1**

La contrasenya l'hem de passar al professorat

No és suficient amb els permisos (group) **read**, ja que aleshores no pot executar. Li posem **write**



### Scripts

Són scripts de Mikrotik, que estan situats en **System -> Scripts**. Crearem 2 scripts per a cada taller, incloent les Aules d'Informàtica que a més seran un poc diferents. 

![image-20220513134523862](/manteniment/assets/img/LlIntTallers/image-20220513134523862.png)

#### Tx-LlevarInternet

**Habilitem** les regles corresponents al taller, cosa que farà que es desactive Internet per a l'alumnat. No podem posar el número de la regla, perquè podem anar afegint més coses i es desquadraria. Aleshores **busquem** (find) les regles per dos conceptes:

- La **chain**, que ha de ser **EVALUA-A-INTERNET**
- L'adreça que vol fer l'accés (**src-address**), que ha de coincidir bé amb l'equip del professorat, be amb tot el taller

![image-20220513135153638](/manteniment/assets/img/LlIntTallers/image-20220513135153638.png)

La primera instrucció habilitarà la que afecta a l'equip de professorat per a permetre-li l'accés a Internet

La segona instrucció habilitarà les 2 regles que afecten als equips d'alumnat: la que permet accedir a les adreces permeses i la que rebutja la resta d'internet

Hem posat que **No requereix permisos** per a que ho puga executar **user1**



#### Tx-PosarInternet

**Deshabilitem** les regles corresponents al taller, cosa que farà que s'active Internet per a l'alumnat, ja que són les que posen restriccions. És pràcticament igual a l'anterior, però amb **disable**

![image-20220513135655865](/manteniment/assets/img/LlIntTallers/image-20220513135655865.png)



### Scripts que executa el professorat

Només ens queda poder executar els scripts de Mikrotik des de qualsevol equip i de forma còmoda per al professorat.

Optem per posar els scripts en el recurs **Departament ** de **brufol**, ja que a aquest recurs només té accés el professorat. És una de les unitat que es munta a cada equip, a més de recurs personal i **Assignatura**.

En **departament/CONNECTAR-DESCONNECTAR INTERNET DELS TALLERS**, dins d'una carpeta per a cada taller tenim els dos scripts. Hem de substituir la **x** pel número de taller

#### Tx-DesconnectarInternet.sh

```bash
#!/bin/bash
ssh user1@192.168.191.2 "system script run Tx-LlevarInternet"
```

#### Tx-ConnectarInternet.sh

```bash
#!/bin/bash
ssh user1@192.168.191.2 "system script run Tx-PosarInternet"
```



No podem oblidar-nos de donar-los permís d'execució, i a més el recurs Departament ha d'estar muntat de manera que tinga permís d'execució. Ho fem en **/etc/security/pam_mount.conf.xml**

```bash
<volume sgrp="domain users" fstype="cifs" server="brufol" path="departament" mountpoint="/media/departament" options="vers=2.1,sec=ntlmv2,nodev,nosuid,dir_mode=0700,file_mode=0700"/>
```


