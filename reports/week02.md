

			Viikko 2

1. Johdanto

-SNMP eli Simple Network Management Protocol on verkonhallintaprotokolla, jota käytetään verkkolaitteiden valvomiseen,
 hallintaan ja tietojen keräämiseen IP-verkoissa. SNMP muodostuu kahdesta pääkomponentista, Agentista ja Managerista.
 Agentti toimii valvottavassa laitteessa (Linux-palvelin, reititin, kytkin, jne.) ja vastaa valvontapalvelimen
 kyselyihin. Manager taas kerään tietoa agenteilta. SNMP Manager kerää tietoa agenteilta valvontaohjelmistonjen, kuten
 Zabbix, PRTG, LibreNMS, avulla.	

2. Asennus

-Asennus aloitettiin kirjautumalla web1-palvelimelle komennolla:
 docker exec -it clab-hamk-verkonhallinta-golden-web1 bash

-SNMP-agentti asennettiin komennolla: 
 apt update && apt install snmp snmpd -y

 jonka jälkeen tarkastettiin palvelun tila komennolla:
 service snmpd status

-nano puuttui, joten se ladattiin komennolla:
 apt install nano -y

-Konfiguraatiotiedosto avattiin muokkausta varten komennolla:
 nano /etc/snmp/snmpd.conf

-Agent Operating Mode -osion viimeiselle riville lisättiin: 
 agentaddress 0.0.0.0,[::] 

-Tiedostoon oli tarkoitus muuttaa tai lisätä yhteisöksi tai yhteisön tilalle rivi:
 rocommunity public
 
 mutta tässä tapauksessa se oli jo tehty.

-Tämän jälkeen palvelu käynnistettiin uudestaan komennolla:
 service snmpd restart

-Prosessin tarkastettiin olevan käynnissä komennolla:
 ps aux | grep snmpd

-Komennolla snmpwalk -v2c -c public web1 system tulee ilmoitus system: Unknown Object Identifier (Sub-id not found: (top) -> system) eli MIB-tiedostot
 ilmeisesti puuttuvat.

-Mennään tiedostoon snmp.conf komennolla: nano /etc/snmp/snmp.conf ja muutetaan mibs : muotoon #mibs : ja tallennetaan

-Tämän jälkeen asennetaan MIBit komennolla: apt update && apt install snmp-mibs-downloader -y && download-mibs

-Testataan yhteys uudestaan komennolla: snmpwalk -v2c -c public web1 system


-Yllämainittu asennusprosessi käydään läpi myös db1:lle ja branch-clientille.


3. Kerätyt tiedot

-Ensimmäinen SNMP-kysely web1-palvelimelta:

 * Järjestelmän nimi: web1 (sysName.0)

 * Käyttöjärjestelmä: Linux

 * sysDescr.0 = STRING: Linux web1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64

 * Uptime: Timeticks: (992632) 2:45:26.32 (2h 45min 26.32s)

 
4. Verkkorajapinnat

-Löytyi 3 verkkorajapintaa:

 * IF-MIB::ifDescr.1 = STRING: lo

 * IF-MIB::ifDescr.670 = STRING: eth0

 * IF-MIB::ifDescr.680 = STRING: eth1

-En osaa tämän hetkisillä tiedoillani selvittää kumpi eth-rajapinta yhdistää laitteen verkkoon. 


5. (I) OID-analyysi

     OID			Tarkoitus
 
   sysName.0		      laitteen nimi

   sysDesc.0                  laitteen kuvaus

   sys.Uptime.0               laitteen toiminnassaoloaika

   ifDescr                    rajapintojen listaus

   ifOperStatus               rajapintojen status


5. (II) Usean laitteen valvonta

   Laite	nimi	Käyttöjärjestelmä	Uptime

   web1		web1	    Linux		3:28:26.80

   db1 		db1 	    Linux		1:14:37.42

branch-client branch-client Linux		0:17:56.26


6. Pohdinta

-SNMP:n avulla voidaan lukea verkkolaitteiden ja palvelimien tilatietoja, kuten CPU-kuorma, muistin käyttöä, rajapintojen liikennemääriä, virhelaskureita
 ja laitteiden lämpötiloja. Tämä tiedonkeruumenentelmä mahdollistaa sen, että SNMP-manageri kerää tiedoja SNMP-agenteilta ja koostaa niistä hyödyllisiä 
 raportteja sekä tarjoaa tärkeää dataa verkonhallinnan ammattilaiselle.

-Yhteisöpohjaisessa SNMPv2:ssa on se ongelma, että verkkoliikennettä ei salata.

-SNMPv3:a käyttäisin mieluummin tuotantoympäristöissä, koska se tukee salausta, käyttäjätunnistusta ja viestien eheyden tarkistusta.
