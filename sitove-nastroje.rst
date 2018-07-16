###############
Síťové nástroje
###############

****************
Diagnostika sítí
****************

.. todo: "sudo" odkaz do I.

.. caution:: Dejte si pozor na to, že některé diagnostické programy mohou zobrazit různé informace
   při použití bez a s právy roota. Jistější je proto vždy je spoustět se sudo.

.. todo: networkctl status

.. _ifconfig-diagnostika:

ifconfig
========

.. todo: že zde jen čteme informace, kdežto v :ref:`ifconfig_konfigurace` nastavujeme.

.. todo: ip - vztah k ifconfigu, místo "ifconfig" spíše "ip addr"
         vztah k "networkctl status enp0s3"

.. ifconfig bez -a zobrazuje jen aktivní rozhraní
   volba -s pro stručný výpis

V předchozí kapitole jsme použili :ref:`ifconfig ke konfiguraci <ifconfig_konfigurace>` některých
parametrů síťových adaptérů, ale nejčastěji slouží pro diagnostiku.

Bez parametrů vypíše informace o povolených síťových rozhraních. Zajímá-li nás jen konkrétní
rozhraní, uveďte ho jako ``ifconfig <rozhraní>``::

    $ sudo ifconfig enp0s3
    enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.88.46  netmask 255.255.255.0  broadcast 192.168.88.255
            inet6 fe80::a00:27ff:fe28:70d0  prefixlen 64  scopeid 0x20<link>
            ether 08:00:27:28:70:d0  txqueuelen 1000  (Ethernet)
            RX packets 28386  bytes 42827546 (42.8 MB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 15327  bytes 1163100 (1.1 MB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

Chcete-li zobrazit i všecha rozhraní (i zakázaná), použijte volbu ``-a`` (all).

Užitečná volba je ``-s`` (short) pro stručnější výpis. Vše můžeme zkombinovat::

    $ sudo ifconfig -a -s
    Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    enp0s3    1500    93652      0      0 0         46177      0      0      0 BMRU
    enp0s8    1500        0      0      0 0             0      0      0      0 BM
    lo       65536      231      0      0 0           231      0      0      0 LRU


.. caution:: Parametry pro ifconfig je nutné uvádět samostatně (např. ``ifconfig -a -s``). Většina
   programů by akceptovala "spojený" zápis jako ``ifconfig -as``, ale pro ifconfig je to neznámý
   parametr ``-as``.

lshw
====

Již dříve zmíněný diagnostický program :ref:`lshw <lshw_disky>` můžeteme použít s parametrem
``-class network`` pro detailní údaje o hardwaru síťových rozhraních jako chipset, sběrnici ap.::

    $ sudo lshw -c network
      *-network
           description: Wireless interface
           product: Wireless 3165
           vendor: Intel Corporation
           physical id: 0
           bus info: pci@0000:01:00.0
           logical name: wlp1s0
           version: 79
           serial: 58:fb:84:45:b8:a2
           width: 64 bits
           clock: 33MHz
           capabilities: pm msi pciexpress bus_master cap_list ethernet physical wireless
           configuration: broadcast=yes driver=iwlwifi driverversion=4.13.0-43-generic firmware=29.610311.0 ip=192.168.88.242 latency=0 link=yes multicast=yes wireless=IEEE 802.11
           resources: irq:285 memory:d1000000-d1001fff

.. _ping:

ping / ping6
============

Notoricky známý program ping (resp. ping6 pro IPv6) zjišťuje spojení mezi dvěma síťovými rozhraními
pomocí výzvy ECHO_REQUEST a odpovědi ECHO_REPLY protokolu ICMP.

Ping kontroluje pouze dosažitelnost cíle. Přesto je v některých sítích s ohledem na bezpečnost
zakazován (resp. ICMP je zakázáno).

Protože ICMP pracuje nad UDP protokolem, nelze ho použít k ověření, zda běží služby fungující nad
TCP jako HTTP, SSH ap.

Na rozdíl od pingu ve Windows linuxový ping pracuje, dokud není zastaven pomocí :kbd:`Ctrl-C`. Pak
vypíše statistiku.

::

    $ ping vacademy.cz
    PING vacademy.cz (95.85.61.132) 56(84) bytes of data.
    64 bytes from 95.85.61.132: icmp_seq=1 ttl=53 time=24.5 ms
    64 bytes from 95.85.61.132: icmp_seq=2 ttl=53 time=23.1 ms
    64 bytes from 95.85.61.132: icmp_seq=3 ttl=53 time=23.1 ms
    ^C
    --- vacademy.cz ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2001ms
    rtt min/avg/max/mdev = 23.101/23.607/24.524/0.672 ms

.. todo: link na SUID v I.

.. note:: Vysílat ICMP je dovoleno jen rootovi, ale protože má ping nastaven SUID právo, mohou ho
   spouštět všichni uživatelé.

tracepath / tracepath6
======================

.. note:: Pokud tracepath6 chybí, doinstalujte ho z balíčku ``iputils-tracepath``.

Ke zjištění trasy k cíli přes všechny brány slouží dvoje tracepath / tracepath6 (pro IPv6). Využívá
k tomu stejně jako `ping`_ ICMP nad protokolem UDP.

::

     $ tracepath vacademy.cz
     1?: [LOCALHOST]                      pmtu 1500
     1:  router                                                3.692ms
     1:  router                                                3.393ms
     2:  191.198.164.1                                         3.029ms
     3:  10.8.87.1                                             5.565ms
     4:  10.8.254.153                                          8.403ms
     5:  10.8.254.29                                           6.825ms
     6:  xitel0.onpow.cz                                       8.889ms
     7:  no reply
     8:  no reply
     9:  vl1369.ss1.r4-1-9.dc4.cejl.brq.masterinter.net       14.229ms
    10:  vl1612.sf2.r3-1-8.dc3.cejl.brq.masterinter.net       15.492ms
    11:  185-58-41-93.static.masterinter.net                  16.138ms reached
         Resume: pmtu 1500 hops 11 back 12


traceroute / traceroute6
========================

.. note:: Některé verze Ubuntu mají pouze traceroute6 pro IPv6. Starší verze Ubuntu neobsahují tento
   program vůbec. Variantu pro IPv4 si doinstalujte z balíčku ``traceroute``.

Traceroute je pokročilejší varianta sledování trasy, která dokáže ověřovat trasu nejen pomocí ICMP
ECHO jako `tracepath / tracepath6`_, ale nabízí i další metody vhodnější do dnešních sítí plných
firewallů, které často blokují UDP porty nebo ICMP ECHO.

.. csv-table:: Porovnání tracepath a traceroute
   :header: "", "tracepath", "traceroute"
   :stub-columns: 1

   "bývá nainstalován?", "většinou ano", "většinou ne"
   "vyžaduje root oprávnění?", "ne", "podle metody"
   "metoda zjišťování", "jen ICMP ECHO (UDP)", "ICMP ECHO (UDP), TCP SYN, ale i další"

Zhruba odpovídajícím programem ve Windows je tracert.

.. rubric:: UDP datagramy -- bez parametrů

Pokud neuvedete žádné parametry, použije traceroute tradiční, ale poněkud zastaralou metodu pomocí
UDP datagramů na neočekávané porty. V případě funkčnosti spojení protistrana obvykle odpoví
"nedosažitelný port".

Tato metoda nevyžaduje root oprávnění.

.. rubric:: ICMP ECHO -- parametr ``-I, --icmp``

Vhodnější metoda pro dnešní sítě. Stejné zjišťování spojení používá `ping / ping6`_ a `tracepath /
tracepath6`_.

Moderní verze linuxového jádra umožňují nastavit vysílání ICMP ECHO i pro neprivilegované uživatele.

.. rubric:: TCP SYN -- parametry ``-T, --tcp`` a ``-p, --port``

Nejvhodnější metoda pro dnešní sítě. Když vynecháte port, použije se 80. V moderních sítích
je možné, že při zjišťování trasy skončíte na prvním firewallu, který blokuje UDP nebo ICMP. Způsob,
jak překonat firewall je použít protokoly a porty, které firewall propustí (TCP SYN, ale
lze specifikovat i další druhy paketů).

Např. ``traceroute -T`` použije port 80 běžně používaný pro HTTP servery, je-li třeba projít jako
mail server vhodný port je zase 25 (SMTP) (``traceroute -T -p 25``) ap.

TCP metoda vyžaduje obvykle root oprávnění.

::

    $ sudo traceroute -T -n vacademy.cz
    traceroute to vacademy.cz (55.15.62.72), 30 hops max, 60 byte packets
    1  12.68.323.8 (192.168.123.1)  0.403 ms  0.411 ms  0.471 ms
    2  20.18.60.1 (10.5.60.1)  32.804 ms  33.770 ms  33.760 ms
    3  20.18.214.91 (10.5.224.41)  33.751 ms  34.319 ms  34.297 ms
    4  20.18.214.19 (10.5.224.29)  34.667 ms  35.213 ms  37.564 ms
    5  63-188-70-118.foo.com (118.45.28.16)  38.136 ms  38.117 ms  38.121 ms
    6  * * *
    9  55.15.62.72 (55.15.62.72)  56.568 ms  54.614 ms  54.652 ms

.. _netstat:

netstat
=======

Hlavní informací, kterou netstat poskytuje je výpis síťových spojení pro TCP, UDP i unixové doménové
sokety. Kromě toho s ním dokážeme zjistit routovací tabulky, maškarády a různé statistiky.

.. note:: Pro některé parametry např. ``-p`` je třeba třeba root oprávnění, proto je vhodnější se
   naučit používat netstat se sudo.

Představíme nejdůležitější parametry:

* ``-a, --all`` -- naslouchající i nenaslouchající sokety
* ``-t, --tcp`` a ``-u, --udp`` -- jen TCP nebo UDP spojení
* ``--route, -r`` -- zobrazí routovací tabulku
* ``-p, --program`` -- zobrazí PID a jméno programu využívající socket
* ``-e, --program`` -- pro proces využívající socket zobraz i jeho uživatele
* ``-n, --numberic`` -- zobrazuj všechny údaje číselně místo jména (IP místo hostname, port místo
  protokolu, UID místo username ap.)

Parametry můžete kombinovat. Nejčastější sled parametry je ``netstat -etupan``
(číselné vyjádření) nebo ``netstat -etupa`` (jména místo čísel).

::

    $ sudo netstat -etupan
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      101        16975      782/systemd-resolve
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      0          19685      898/sshd
    tcp        0      0 192.168.41.64:22        192.168.12.42:53422     ESTABLISHED 0          21357      1214/sshd: sally [p
    tcp        0      0 192.168.41.64:22        192.168.12.42:53424     ESTABLISHED 0          25531      2997/sshd: sally [p
    tcp6       0      0 :::22                   :::*                    LISTEN      0          19697      898/sshd
    udp    42240      0 127.0.0.53:53           0.0.0.0:*                           101        16974      782/systemd-resolve
    udp        0      0 192.168.18.16:68        0.0.0.0:*                           100        27908      763/systemd-network

.. todo: "grep" odkaz do I.

Občas nás zajímají jen aktivní spojení (aktivní spojení se nachází ve stavu "ESTABLISHED"). Můžeme si
proto grepem::
   
    $ sudo netstat -etupan | grep -i esta
    Active Internet connections (servers and established)
    tcp        0      0 192.168.18.16:22        192.168.18.42:53422    ESTABLISHED 0          21357      1214/sshd: sally [p
    tcp        0      0 192.168.18.16:22        192.168.18.42:53424    ESTABLISHED 0          25531      2997/sshd: sally [p

nslookup
========

Nslookup je základní program pro dotazování a diagnostice systému DNS.

Jeho použití pro jednduché dotazy vypadá::

    $ nslookup vacademy.cz
    Server:		127.0.0.53
    Address:	127.0.0.53#53

    Non-authoritative answer:
    Name:	vacademy.cz
    Address: 185.58.41.93
    Name:	vacademy.cz
    Address: 2a01:430:144::2

dig
===

Dig je pokročilejší nástroj k dotazování a diagnostice systému DNS. Obecná syntaxe pro jeho spuštění
vypadá::

    dig [@<server>] <jméno> [<typ>]

kde

* ``<server>`` je jméno nebo IP adresa DNS serveru, kterého se chcete dotazovat. Když server
  vynecháte, dig se podívá do ``/etc/resolf.conf``. Pokud nenajde záznam ani tam, dotáže se aktuální
  počítače (localhostu).
* ``<jméno>`` je doménové jméno
* ``<typ>`` je typ hledáného DNS záznamu jako např. A, MX, TXT, SRV, CNAME atd. Když vynecháte,
  hledají se A záznamy.

Nař. dotaz na MX záznamy::

    $ dig vacademy.cz mx

    ; <<>> DiG 9.9.5-3ubuntu0.5-Ubuntu <<>> mx vacademy.cz
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54955
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 4, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1280
    ;; QUESTION SECTION:
    ;vacademy.cz.			IN	MX

    ;; ANSWER SECTION:
    vacademy.cz.		3600	IN	MX	10 alt3.aspmx.l.google.com.
    vacademy.cz.		3600	IN	MX	1 aspmx.l.google.com.

    ;; Query time: 132 msec
    ;; SERVER: 127.0.1.1#53(127.0.1.1)
    ;; WHEN: Mon Dec 14 16:40:54 CET 2018
    ;; MSG SIZE  rcvd: 254

****************
Analýza a ladění
****************

telnet
======

Telnet je klient pro stejnojmenný protokol sloužící pro komunikaci se vzdáleným hostitelem.
Nepoužívá žádné zabezpečení a proto je již dlouho nahrazován protokolem :ref:`SSH`. Nicméně pro jeho
jednoduchost je telnet často využíván pro "simulaci" klienta jiných textově orientovaných protokolů
jako HTTP, SMTP ap.

Syntaxe pro připojení je::

    telnet <host> <port>

Zkusme "předstírat" např. činnost webového browseru::

    $ telnet vacademy.cz 80
    Trying 95.85.61.132...
    Connected to vacademy.cz.
    Escape character is '^]'.

V tento moment se zobrazí výzva a můžete začít psát. Např. HTTP příkaz pro získání hlavní stránky
(/) je::

    GET / HTTP/1.1
    Host: vacademy.cz

a odešlete 2x stiskem :kdb:`Enter`. Uvidíte HTTP hlavičky odpovědi a zdrojový kód HTML stránky (jde
o přesměrování na https://vacademy.cz)::

    HTTP/1.1 301 Moved Permanently
    Server: nginx
    Date: Wed, 16 May 2018 16:20:29 GMT
    Content-Type: text/html
    Content-Length: 178
    Connection: keep-alive
    Location: https://vacademy.cz/
    X-Rosti: lb.rosti.cz

    <html>
    <head><title>301 Moved Permanently</title></head>
    <body bgcolor="white">
    <center><h1>301 Moved Permanently</h1></center>
    <hr><center>nginx</center>
    </body>
    </html>

.. todo: telnet pro https a další SSL protokoly (nástrojem z openssl)

nmap
====

Nmap je program pro skenování portů a běžících služeb. Umí odhalit nejen otevřené porty (např. pro
testování funkčnosti firewallu), ale v některých případech i druh zařízení a jeho software, či živé
síťové zařízení, které má ale zakázaný ping. Nmap je pro správce sítě doslova nepostradatelný
nástroj.

Nmap nainstalujete standardním způsobem z repozitářů::

    sudo apt install nmap

Použití nmapu vypadá::

    nmap [volby] <cílový_hostitel>

Hostitel může být specifikován IP adresou nebo jménem::

    nmap 192.168.123.47

ale můžete určit více cílů pomocí intervalu např. všechny IP 192.168.1.1 až 192.168.123.254::

    nmap 192.168.1-123.1-254

Mezi užitečné volby patří zejm.

* ``-PN`` -- ověření hosta i když blokuje ICMP ping
* ``-p 8081``, ``-p 8081,2900``, ``-p 8081-8090`` -- jen port 8081, porty 8081 a 2900, porty 8081 až 8090
* ``-F`` -- rychlý sken (méně služeb, než ve výchozím skenu)
* ``-A`` -- pokusí se zjistit OS a jeho verzi
* ``-v`` -- podrobný výpis

.. code-block:: text
   :caption: Příklad použití nmapu
   
   $ nmap google.com

   Starting Nmap 7.60 ( https://nmap.org ) at 2018-06-15 14:40 CEST
   Nmap scan report for google.com (216.58.201.110)
   Host is up (0.012s latency).
   Other addresses for google.com (not scanned): 2a00:1450:4014:801::200e
   rDNS record for 216.58.201.110: prg03s02-in-f110.1e100.net
   Not shown: 998 filtered ports
   PORT    STATE SERVICE
   80/tcp  open  http
   443/tcp open  https

   Nmap done: 1 IP address (1 host up) scanned in 7.42 seconds

tcpdump
=======

.. todo: tcmpdump pro SSL provoz (HTTPS ap.) - jak na to?


.. todo:: musí mít fakt rozhraní promiskuitní režim ke sniffování?
    .. important:: Pro odposlouchávání tcmpdumpem nebo později probíraným
       :ref:`Wiresharkem <wireshark>` je třeba, aby síťové rozhraní podporovalo a mělo zapnutý tzv.
       :ref:`promiskuitní režim <ifconfig-promisc>`.

Tcpdump je základní, ale velmi dobře použitelný *sniffer (program pro odposlouchávání)*. V Ubuntu je
předinstalovaný, dokáže odposlouchávat příchozí i odchozí pakety na síťových rozhraních, filtrovat
je jen na určitého hostitele, port ap. Provoz zobrazuje rovnou na obrazovku nebo ukládá do souboru
PCAP pro pozdější analýzu.

Pro odposlouchávání je třeba spouštět tcpdump s právy superuživatele.

Tcpdump má velmi mnoho voleb. Představíme si jen několik důležitých parametrů a příkladů. Úplný
výčet najdete, jako vždy, v ``man tcpdump``. Obecná syntaxe se skládá z voleb a především hledacího
výrazu::

    tcpdump <volby> <hledací_výraz>

.. rubric:: Sleduj jen určité rozhraní

Pomocí volby ``-i`` bude vypisovat přijaté a odeslané pakety jen na určeném síťovém rozhraní. Adresy
se pokusí překládat na jména.

::

    sudo tcpdump -i enp0s3

.. rubric:: Nepřekládej jména

Překlad na jména je často zbytečný a vždy zdlouhavý. Parametrem ``-n`` se vypíšou jen IP adresy.

::

    sudo tcpdump -i enp0s3 -n

.. rubric:: Omezení na IP adresu, port, protokol

Také sledovat veškerý provoz je často nežádoucí. Zde poprvé použije hledací výraz. Filtrovat můžeme
jen komunikaci na určený cíl nebo jen z určeného zdroje syntaxí ``dst|src host <ip>``.

::

    sudo tcpdump -i enp0s3 -n dst host 192.168.123.150
    sudo tcpdump -i enp0s3 -n src host 192.168.123.150

Podobně lze omezit provoz i na konkrétní port pomocí ``port <port>``.

::

    sudo tcpdump -i enp0s3 -n port 8080

Nebo na protokol TCP, UDP nebo ICMP pomocí stejnojmenných parametrů.

::

    sudo tcpdump -i enp0s3 -n tcp
    sudo tcpdump -i enp0s3 -n udp
    sudo tcpdump -i enp0s3 -n icmp

Podmínky lze kombinovat s omezením na IP nebo použít samostatně.

::

    sudo tcpdump -i enp0s3 -n src host 192.168.123.150 port 8080
    sudo tcpdump -i enp0s3 -n src host 192.168.123.150 port 8080 tcp

Někdy může hledací výraz kolidovat s volbami pro tcpdump a proto je uzavřen do jednoduchých uvazovek (``'``). Např. ``tcpdump 'gateway snup and (port ftp or ftp-data)'``.

Syntaxe hledacího výrazu je obsáhlá a popsána v ``man pcap-filter``.

.. rubric:: Nepřekládat jména hostitelů a služeb

Standardně tcmpdump zkouší překládat adres hostitelů podle DNS a názvy služeb podle
:ref:`/etc/services <etc_services>`. To je ale často zbytečné a především zdržující. Volbou ``-n``
toto chování vypneme.

::

    sudo tcpdump -n -i enp0s3

.. rubric:: Zobrazení protékajících data

tcpmdump standardně nezobrazuje obsah, jen hlavičky z provozu. Abyste zobrazili opravdu veškerý
obsah, který "teče" sítí použijeme několik voleb

::

    $ tcpdump -nXSs 0 'port 80'

* ``-n`` nepřekládá adresy a porty na jména a služby
* ``-X`` vypisuje každý paket jak v hexa, tak jako ascii
* ``-s 0`` standardně tcmpdump zaznamená jen začátek každého paketu. 0 znamená, aby uložil celý paket.

Připravte se na to, že výstup bude velmi obsáhlý.


.. _tcpdump_pcap:
.. rubric:: Uložení do PCAP souboru

Pokud je i tak zobrazovaný provoz příliš veliký nebo chcete provést analýzu později, je tu možnost
uložit soubor do speciální PCAP formátu pomocí ``-w <soubor.pcap>``, který umí číst např. dále
zmiňovaný `Wireshark`_.

Pozor na to, že standardně se omezuje délka ukládaných paketů na 65 535 bajtů a proto je vhodné
ještě použít ``-s 0`` pro zrušení tohoto limitu.

::

    sudo tcpdump -i enp0s3 -n -w soubor.pcap -s0

.. _wireshark:

Wireshark
=========

Wireshark (dříve Ethereal) je grafický sniffer. Umí sám zachytávat, ale často je používán k tzv.
*post-mortem analýze*, kdy zachytíme provoz na problémové negrafické stanici do :ref:`.pcap souboru
<tcpdump_pcap>` a na stolním počítači s Wiresharkem následně v pohodlně analyzujeme.

.. figure:: img/wireshark.png

   Wireshark vyniká ve snadném ovládání, hledání v zachyceném provozu a je nabízen zdarma pro
   všechny platformy.

***************
Přenosy po síti
***************

V této části zkusíme několik linuxových programů a protokolů pro stahování, nahrávání, kopírování po
vnitřní síti a internetu.

wget
====

Wget a `curl`_ patří mezi velmi populární download managery pro příkazovou řádku. Je nainstalován na většině systémů již v základu a dobře se úkoluje ze skriptu. Dokáže stahovat soubory přes HTTP, HTTPS a FTP.

Ovládání pro základní použití - stažení souboru - je velmi jednoduché. Výsledný soubor se bude
jmenovat, tak jak je uvedeno v URL adrese (``master.zip``)

::

    wget https://github.com/vacademycz/book-usrv2-cz/archive/master.zip

V případě, že URL je na složku (např.
https://vacademy.cz/knihy/usrv2/ubuntu-debian-pro-spravce-serveru-ii/stahnout/pdf/ jako v
následujícím příkladě), wget bohužel vytvoří soubor ``index.html`` i když výsledný soubor je PDF a
má se jmenovat ``ubuntu-debian-pro-spravce-serveru-ii.pdf``. Je třeba mu dopomoci volbou ``-O``,
kterým nastavíme jméno staženého souboru::

    $ wget https://vacademy.cz/knihy/usrv2/ubuntu-debian-pro-spravce-serveru-ii/stahnout/pdf/ -O book.pdf
    --2018-07-16 15:28:31--  https://vacademy.cz/knihy/usrv2/ubuntu-debian-pro-spravce-serveru-ii/stahnout/pdf/
    Resolving vacademy.cz (vacademy.cz)... 185.58.41.93, 2a01:430:144::2
    Connecting to vacademy.cz (vacademy.cz)|185.58.41.93|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 2085765 (2,0M) [application/pdf]
    Saving to: ‘book.pdf’

    book.pdf                              100%[================================>]   1,99M  3,82MB/s    in 0,5s

    2018-07-16 15:28:32 (3,82 MB/s) - ‘book.pdf’ saved [2085765/2085765]

curl
====

Curl (někdy čteno "kárl") je daleko mocnější, než wget - kromě podpory mnoha dalších protokolů jako
IMAP(S), LDAP(S), POP3(S), SCP, SFTP, SMTP a dovede i soubory nahrávat. Je dobře skriptovatelný a
často se používá pro testování a volání RESTful webových služeb z příkazové řádky.

Bez parametrů vypisuje stahovaný soubor na STDOUT (terminál). S volbou ``-O`` uloží soubor pod
jménem z URL adresy (tj. to co obvykle chcete, když stahujete). Stažený soubor se bude jmenovat
``master.zip``::

    $ curl -O https://github.com/vacademycz/book-usrv2-cz/archive/master.zip

Chcete-li ale vybrat pro stažený soubor jiné jméno použijte ``-o <nazev-souboru>``::

    $ curl -o prirucka_usrv2.zip https://github.com/vacademycz/book-usrv2-cz/archive/master.zip

Curl zvládá ohromné množstí protokolů. Např. užitečné do skriptu může být poslání emailu::

    $ curl --mail-from foo@vacademy.cz --mail-rcpt bar@vacademy.cz smtp://somemailserver.com

Další příklad může být např. HTTP POST požadavek na WWW-Basic zabezpečený cíl s tělem jako
``application/x-www-form-urlencoded`` (HTML formulář)::

    $ curl -X POST --user "sally:mypass" -d "image_id=y4a7cc" -d "file=sallymage" http://localhost:4002/api/images

Zbývající obrovský počet možnosti curlu a tomu odpovídající parametrů najdete v manuálové stránce.

scp
===

Program scp (secure copy) používá ke kopírování :ref:`SSH`. Můžeme ho použít pro jednorázový
zabezpečený přesun složky nebo souborů z/do vzdáleného počítače, kam máte přístup přes SSH. Je možné
dokonce přenášet soubory i mezi dvěma vzdálenými servery.

Jeho obecná syntaxe vypadá

::

    scp <volby> [[uživatel@]host1:]soubor1... [[uživatel@]host2:]soubor2

kde *soubor1* i *soubor2* mohou být cíle pro stažení i nahrání v závislosti na podobě parametru. Ke
stažení na lokál a upload na vzdálený server jen přehazujeme první a druhé místo.

Ke stahování je první cíl vzdálené místo a druhý místní::

    $ scp sally@tristar:/var/log/cups/error.log ~/tmp/

Naopak pokud je první cíl místní, jde o upload na druhý cíl::

    $ scp ~/tmp/error.log sally@tristar:/tmp/

Mezi užitečné volby patří:

* ``-P`` určující port, není-li SSH na standardním 22 (pozor nepoužívá ``-p`` jako SSH klient)
* ``-i`` použití jiného SSH klíče, než našeho hlavního v ``~/.ssh/``
* ``-r`` kopírování složek (bez této volby vypíše při pokusu o kopírování složky scp chybu!)

rsync
=====

Posledním důležitým programem každého správce Linuxu pro kopírování a přesuny souborů je rsync. Ten
se od ostatních liší tím, že přenáší jen rozdíly. Když změníte ve 100 MiB souboru 1 bajt budete s
programy cp, scp a dalšími přenášet znovu celých 100 MiB. S rsync jen pár bajtů - změněný bajt a
několik bajtů jako režii.

Mezi další úžasné vlastnosti rsync patří, že dovede přenášet lokálně i po síti. Pro ještě větší
ušetření kapacity dokáže přenosy komprimovat a přenos po síti lze zabezpečit přes :ref:`SSH`.

.. rubric:: Syntaxe a parametry

Základní syntaxe je

::

    $ rsync [volby] <zdroj> <cíl>

Zdroj nebo cíl může být soubor nebo složka. Mezi důležité volby patří

* ``-a`` — "archivační režim" - zachovej u kopírovaných souborů oprávnění, vlastnictví, přenes
  symbolické odkazy ap.
* ``-r``, ``--recursive`` -- pracuj rekurzivně (jdi do podsložek)
* ``-v``, ``-vv`` nebo ``-vvv`` — vypisuj prováděnou činnost od nejdůležitějších (jedno "v") po
  velmi detailní (tři "v")
* ``--progress`` — zobrazí postup činnosti
* ``--exclude`` — vynech soubory/složky vyhovující masce. Např. ``--exclude="*.bak"`` nebude
  přenášet BAK soubory. Pro vynechání více souboru nebo složek musíte parametr opakovat.
* ``--delete`` — z cílové složky odstraní soubory, které již ve zdrojové složce neexistují.
  Vhodné pro "synchronizaci" mezi dvě místy. **Při používání této volby buďte opatrní!**
* ``--dry-run`` — běh "na sucho", tj. jen předstírej, ale neprováděj žádnou činnost. Vhodné pro
  odzkoušení nebezpečných voleb jako --delete před během na ostro.
* ``-z, --compress`` — přenos komprimuj pro ušetření kapacity pásma

Volby můžete kombinovat: např. ``-av`` je rovnocenné k ``-a -v``.

.. rubric:: Koncová závorka ve zdrojové cestě

Rsync se chová odlišně, pokud je ve zdrojové cestě ukončující lomítko (trailing slash) nebo není.
(Koncová závorka se pro cílovou cestu nerozlišuje.) Jinými slovy jiné chování způsobí bez a s
uvedením "/" u zdroje.

::

$ rsync -r /var/log  /mnt/backup    # bez ukončujícího lomítka
$ rsync -r /var/log/ /mnt/backup    # s ukončujícím lomítkem

Při vynechání ukončujícího lomítka "/" rsync nejdříve vytvoří zdrojovou složku v cíli, pak až do ní
nakopíruje obsah.

::

    $ rsync -r var_log mnt_backup
    $ tree mnt_backup
    mnt_backup
    └── var_log
        ├── auth.log
        ├── auth.log.1
        ├── auth.log.2.gz
        ├── syslog
        ├── syslog.1
        └── syslog.2.gz


Při uvedení ukončujícího lomítka "/" kopíruje podsložky a soubory zdroje rovnou do cíle.

::

    $ rsync -r var_log/ mnt_backup
    $ tree mnt_backup
    mnt_backup
    ├── auth.log
    ├── auth.log.1
    ├── auth.log.2.gz
    ├── syslog
    ├── syslog.1
    └── syslog.2.gz


.. rubric:: Vzdálené přenosy přes SSH

Rsync obsahuje i serverovou část démona rsyncd, který je sice rychlý, ale přenosy nešifruje. Proto
se v praxi téměř výlučně používá přenos přes :ref:`SSH`. Přenos na vzdálený stroj přes SSH vypadá
např.::

    rsync -av -e ssh --delete /home/sally/logs sally@tristar:/mnt/backup

Nová je tu volba ``-e ssh``. Jiná je také syntaxe cíle. Samozřejmě můžeme i kopírovat zpět
(obnovit) ze vzdáleného serveru na lokální prohozením cíle a zdroje::

    rsync -av -e ssh sally@tristar:/mnt/backup /home/sally/logs

Pokud SSH nepoužívá standardní port 22, ale např. 2810 vypadá příkaz trochu krkoloměji::

    rsync -av -e "ssh -p 2810" /home/sally/logs sally@tristar:/mnt/backup