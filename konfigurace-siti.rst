#########################
Konfigurace sítí v Linuxu
#########################

***************
Síťová rozhraní
***************

Síťová rozhraní či síťové adaptéry jsou :ref:`souborova-zarizeni` ve složce ``/dev/``. Podle typu
adaptéru rozlišujeme

* eth0, eth1, ... - Klasické kabelově vedené ethernetové rozhraní. První rozhraní je eth0 atd.
* lo - Loopback zařízení. Je speciální síťové rozhraní, které je vždy přítomné i na PC bez skutečné
  síťové karty (dnes se již asi s takovým počítačem nesetkáte). Slouží k provozu síťových aplikací i
  bez nutnosti "jít" na skutečnou síť. Má přiřazenou známou pevnou IP adresu 127.0.0.1.
* wlan0, wlan1, .. Bezdrátové Wi-Fi karty jsou označeny wlan jako wireless LAN. První rozhraní je
  wlan0 atd.

.. topic:: Speciální IP adresy

   Kromě IP adresy přiřazené administrátorem nebo DHCP systémem se používá několik speciální IP
   adres s pevným významem:

   * 0.0.0.0 — všechna síťová rozhraní počítače
   * 127.0.0.1 — loopback rozhraní (/dev/lo) známé také jako localhost.

***********************
/etc/network/interfaces
***********************

Konfigurační soubor ``interfaces`` obsahuje informace, jak se připojit k síti. Na rozdíl od později
probíraných programů jako `ifconfig`_ je nastavení v interfaces přečteno při startu počítače a tedy
trvalé.

Pokud používáte v síti DHCP není třeba vůbec žádné nastavení a váš soubor interfaces obsahuje jen
dva řádky pro zapnutí (``auto``) a nastavení ``iface`` loopback zařízení::

  auto lo
  iface lo inet loopback

Povely ``auto``, ``iface`` a další nazývá dokumentace jako *stanzas*. Konfigurace je velmi rozsáhlá
a proto pro pokročilejší nastavení odkazujeme na ``man 5 interfaces``. Ukážeme si jen další typický
setup se statickou IP adresou bez DHCP::

  auto eth0
  iface eth0 inet static
          address 192.168.1.1
          netmask 255.255.255.0
          gateway 192.168.1.50
          dns-nameservers 192.168.1.50



Potřebujete-li nastavit sekundární, terciární, … DNS do předchozího řádku přidejte mezerou oddělení
jejich IP adresy::

  dns-nameservers 192.168.1.50 192.168.1.51

Pro aplikaci nastavení je třeba provést `restart sítí`_.

todo: /etc/resolv.conf
**********************

ifconfig
********

.. note:: V DOSu a Windows je podobný (i když daleko méně mocnější) program nazván ipconfig.

Ifconfig neboli interface configuration je hlavní linuxový nástroj pro konfiguraci a diagnostiku
sítích. Je používán z řady startovacích skriptů, ale lze ho použít i na přímo z příkazové řádky.

S ipconfig nenastavíte bránu (gateway) a DNS. Bránu nastavujete v
:ref:`routovacích tabulkách <routovaci-tabulky>`. DNS v souboru /etc/network/interfaces.

.. caution:: Veškeré změny provedené ifconfigem "nepřežijí" restart. Pro trvalou změnu
   konfigurace sítových adaptérů musíte editovat /etc/network/interfaces.

.. rubric:: Informace o všech zařízeních

Bez parametrů vypíše základní informace o síťových rozhraních::

  $ sudo ifconfig
  eth0      Link encap:Ethernet  HWaddr e0:db:55:a8:d7:0a
            inet addr:192.168.123.102  Bcast:192.168.123.255  Mask:255.255.255.0
            inet6 addr: fe80::e2db:55ff:fea8:d70a/64 Scope:Link
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
            RX packets:436 errors:0 dropped:0 overruns:0 frame:0
            TX packets:518 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000
            RX bytes:363793 (363.7 KB)  TX bytes:82767 (82.7 KB)
            Interrupt:17

  lo        Link encap:Local Loopback
            inet addr:127.0.0.1  Mask:255.0.0.0
            inet6 addr: ::1/128 Scope:Host
            UP LOOPBACK RUNNING  MTU:65536  Metric:1
            RX packets:1065332 errors:0 dropped:0 overruns:0 frame:0
            TX packets:1065332 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:0
            RX bytes:116355367 (116.3 MB)  TX bytes:116355367 (116.3 MB)

  wlan0     Link encap:Ethernet  HWaddr 68:17:29:74:1b:75
            inet addr:192.168.123.104  Bcast:192.168.123.255  Mask:255.255.255.0
            inet6 addr: fe80::6a17:29ff:fe74:1b75/64 Scope:Link
            UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
            RX packets:2539568 errors:0 dropped:1 overruns:0 frame:0
            TX packets:1962100 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000
            RX bytes:2776193863 (2.7 GB)  TX bytes:552714924 (552.7 MB)

.. rubric:: Povolení/zakázání rozhraní

::

  # Povolení
  sudo ifconfig eth0 up
  # nebo jen
  sudo ifconfig eth0 down

  # Zakázání
  sudo ifconfig eth0 down

.. rubric:: Přiřazení IP adresy, masky podsítě

::

  # Jen IP
  sudo ifconfig eth0 192.168.123.141
  # IP a masky
  sudo ifconfig eth0 192.168.123.141 netmask 255.255.255.224

Ověření můžete provést přes ``ifconfig eth0``.

.. _ifconfig-promisc:

.. rubric:: Povolení/zakázání promiskuitního režimu

::

  # Povolení
  sudo ifconfig eth0 promisc
  # Zakázání
  sudo ifconfig eth0 -promisc

.. rubric:: Změna MAC adresy

Inconfig dokonce umožňuje nastavit MAC adresu rozhraní::

  sudo ifconfig eth0 hw ether AA:BB:CC:DD:EE:FF

.. topic:: Ethtool

   Další pokročilejší nastavení jako duplex režim, wake-on-LAN můžete spravovat nástrojem ethtool
   (není standardně přeinstalovaný). Stejně jako u všech ostatních nástrojů jsou změny netrvalé. Aby
   nastavení provedené ``ethtool``em zůstalo i po restartu je třeba změnu přidat do
   /etc/network/interfaces do části ``pre-up``. Např. změna portu na 1000 Mb/s ve full duplex režimu
   (ostatní řádky ukázky nejsou relevantní)::

      auto eth0
      iface eth0 inet static
      pre-up /sbin/ethtool -s eth0 speed 1000 duplex full

.. _routovaci-tabulky:

Routovací tabulky
*****************

Routovací nebo taktéž směrovací tabulky.

Příkazem ipconfig nemůžeme nastavit výchozí nebo dodatečnou bránu. K manipulaci s routovacími
tabulkami jádra slouží samostatný příkaz ``route``.

.. rubric:: Nastavení výchozí brány (default gateway)

Brána se nenastavuje na rozhraní, ale v routovacích tabulkách jádra. Používáme proto příkaz
``route``.

::

  sudo route add default gw 192.168.123.1 eth0

.. rubric:: Výpis routovací tabulky

Pro ověření můžete vypsat tabulku pomocí ``route -n``::

  $ route -n
  Kernel IP routing table
  Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
  10.0.0.0        0.0.0.0         255.255.255.0   U     1      0        0 eth0
  0.0.0.0         10.0.0.1        0.0.0.0         UG    0      0        0 eth0

Stejný výstup získáme i pomocí ``netstat -r`` (viz ref:`netstat`).

Restart sítí
************

Po změně jakékoli síťové konfigurace (např. po změně z pevné IP na DNS) nemusíme restartovat celý
operační systém, ale stačí jen síťového subsystém pomocí příkazů::

  sudo /etc/init.d/networking stop
  sudo /etc/init.d/networking start

Tyto skripty používají příkazy ifup (zapnutí) a ifdown (vypnutí) všech rozhraních. Pokud potřebujete
restart jen určitého rozhraní, můžete je použít na přímo::

  sudo ifdown eth0
  sudo ifup eht0

Hostname (jméno počítače)
*************************

..note:: Jméno počítače nemá prakticky žádný vliv. Pokud nemáte centrální správu jmen (DNS) ostatní
  stanice vámi zvolený hostname neznají a mohou se na vás odkazovat pouze číselnou IP adresou.

Nový název počítače (hostname) nastavíte nebo stávající vypíšete programem ``hostname``:

  $ sudo hostname bomber
  $ hostname
  bomber

Mějte na paměti, že změna je opět platná dokud nerestartujete počítač. Pro trvalou změnu napište
jméno do souboru ``/etc/hostname``, který je čten startovacími skripty.

Soubory /etc/hosts a /etc/services
**********************************

Soubor ``/etc/hosts`` je textovým souborem do kterého se Linux podívá jako prvního, jestliže má
přeložit (resolve) jmenný název (hostname) na IP adresu (např. virtage.cz na 210.102.2.189). IP
mohou být jak místní, tak platné z internetu.

Protože tento soubor se prohledává ještě před dotazem na nastavený DNS server je to vhodné místo pro
"zfalšování" adresy hostname serveru na kterém má běžet aplikace, kterou teprve vyvíjím. Všechny
odkazy a dotazy na např. www.mujserver.cz tak můžete přesměrovat na 127.0.0.1 (místní počítač).

IP adresa je od DNS jména nebo jmen oddělena jedním tabelátorem. Na jednom řádku můžete pro stejnou
IP vypsat více hostname.

.. code-block:: text
   :caption: Příklad ``/etc/hosts``

   127.0.0.1       localhost nb-mujnb www.virtage.cz.local
   192.168.0.100   fileserver
   
   # The following lines are desirable for IPv6 capable hosts
   ::1     ip6-localhost ip6-loopback
   fe00::0 ip6-localnet
   ff00::0 ip6-mcastprefix
   ff02::1 ip6-allnodes
   ff02::2 ip6-allrouters

Soubor /etc/services má podobný účel, ale slouží k "překladu" služeb (protokolů) na čísla portů.
Používá ho řada programů, aby zobrazovala např. místo 22 symbolický název "ssh".

.. code-block:: text
   :caption: Příklad ``/etc/services``

   tcpmux          1/tcp                           # TCP port service multiplexer
   echo            7/tcp
   echo            7/udp
   discard         9/tcp           sink null
   discard         9/udp           sink null
   systat          11/tcp          users
   daytime         13/tcp
   daytime         13/udp
   netstat         15/tcp
   qotd            17/tcp          quote
   msp             18/tcp                          # message send protocol
   msp             18/udp
   chargen         19/tcp          ttytst source
   chargen         19/udp          ttytst source
   ftp-data        20/tcp
   ftp             21/tcp
   fsp             21/udp          fspd
   ssh             22/tcp                          # SSH Remote Login Protocol
   ssh             22/udp

Více informací naleznete v man 5 services.
