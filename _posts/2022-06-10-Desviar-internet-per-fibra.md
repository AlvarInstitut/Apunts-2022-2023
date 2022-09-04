---
typora-copy-images-to: ../assets/img/LlIntTallers/
typora-root-url: ../../
layout: post
title:  Desviar internet per la fibra nostra
categories: parte1
conToc: true
permalink: desviar-internet-per-fibra

---

### Desviar internet per la fibra nostra

La manera de desviar per la fibra nostra serà etiquetant els paquets de les màquines que vulguem, per a així posteriorment enrutar per el gateway de la fbra.

Per a muntar-ho de forma eficient:

### 1.- Llista d'equips permesos

Serà la llista d'equips que desviarem per la fibra quan vulguem.

Ho farem per tallers i també de forma individual

Des de **IP -> Firewall -> Address Lists**

![image-20220610114948825](/manteniment/assets/img/LlIntTallers/image-20220610114948825.png)

Per a cadascun dels que vulgem crearem una nova entrada amb el nom **equips_permesos_fibra** i el rang d'adreces:

![image-20220610115139113](/manteniment/assets/img/LlIntTallers/image-20220610115139113.png)





### 2.- Etiquetatge de paquets: IP ->Firewall -> Mangle

Ací és on als paquets que venen des dels equips que volem desviar, els etiquetarem

![image-20220610115407714](/manteniment/assets/img/LlIntTallers/image-20220610115407714.png)

**La primera regla** etiqueta els paquets amb Mark Connection:

![image-20220615132132233](/manteniment/assets/img/LlIntTallers/image-20220615132132233.png)

![image-20220615132227495](/manteniment/assets/img/LlIntTallers/image-20220615132227495.png)

![image-20220610115703160](/manteniment/assets/img/LlIntTallers/image-20220610115703160.png)

El que hem fet és dir-li que els paquets que venen de la Llista **equips_permesos_fibra** que **no **van a la xarxa interna (és a dir, que van a Internet) es marquen amb l'etiqueta **proveta_bona**. La llista de direccions que no són xarxa interna l'haurem de revisar. L'hem etiquetada com a **adreces_sempre_institut** : 

- **192.168.0.0/16**  . Segurament no haurem de posar les altres de l'Institut perquè Mikrotik ja sap on estan, i per a ell són de la 192.168.2.0/24 (per exemple fsserver és 192.168.2.250)
- **aules.edu.gva.es** . Sinó, quan desviem per la fibra no troba aules. També hem inclós (de moment) **172.27.111.20**, que és habitualment aules.edu.gva.es

Aquest llista l'hem creada en **IP -> Firewall -> Address List**

![image-20220615132526885](/manteniment/assets/img/LlIntTallers/image-20220615132526885.png)



**La segona regla**, que activarem quan vulguem desviar per la fibra, el que fa és els paquets etiquetat amb **proveta_bona** que venen dels equips de la llista que volem desviaretiquetar-los amb **mark routing** com a **proveta_bona_ruta**

![image-20220610120116509](/manteniment/assets/img/LlIntTallers/image-20220610120116509.png)

![image-20220610120145967](/manteniment/assets/img/LlIntTallers/image-20220610120145967.png)









### 3.- Enrutar els paquets

En **IP -> Routes** senzillament el que fem és els paquets etiquetats amb **proveta_bona_ruta**, posar com a gateway la de la fibra

![image-20220610121153259](/manteniment/assets/img/LlIntTallers/image-20220610121153259.png)

La priema regla és la bona. La segona la deixem per si de cas, però desactivada, perquè ara pensem que no cal

![image-20220610121259832](/manteniment/assets/img/LlIntTallers/image-20220610121259832.png)

Tots els paquets etiquetas amb **proveta_bona_ruta** són encaminats per la fibra: **192.168.1.1**

