#################
Síťová bezpečnost
#################

.. _firewall:

********
Firewall
********

.. rubric:: Potřebuji firewall?

Otázka je to celkem složitá a také by měla být lépe položená, ale jednoduchá odpověď zní - pokud
neprovozujete server, tak ne.

Nyní se na tuto problematiku podíváme trochu šířeji. Jak se dočtete níže, váš Linux obsahuje
vestavěný firewall iptables, který se automaticky stará o veškerý síťový provoz a podle
přednastavených pravidel (respektive podle vámi předepsaných pravidel, pokud se je rozhodnete
změnit) jej také spravuje a rozhoduje, co s jednotlivými pakety udělá.

Ve výchozím nastavení je iptables nastaveno velmi volně a síťový provoz prakticky nijak neomezuje.
Ač to nemusí být zřejmé, vaše bezpečnost není nijak výrazně ohrožena. Většina domácích počítačů je
firewallem chráněna aniž o tom jejich uživatelé vědí (firewall se totiž nachází v různých zařízeních
jako jsou switche, routery a další) a navíc je většina "zneužitelných" služeb a nástrojů (jako SSH,
webový server a další) vypnuta nebo není v základní instalaci Ubuntu vůbec zahrnuta.

.. _UFW:

UFW
===

UFW, neboli Uncomplicated FireWall, je pro Ubuntu výchozí a velmi jednoduché řešení, které však
pokryje takřka 100% požadavků na firewall. Jak název napovídá program se snaží o nekomplikovaný
způsob nastavování firewallu a sami zjistíte, že se mu to daří. Ve skutečnosti se jedná o nadstavbu
nad poměrně složitým iptables firewallem.

Instalace není nutná - UFW je součástí Ubuntu desktop i server edicí.

Stavové informace
-----------------

Nejprve zjistěte stav firewallu::

    $ sudo ufw status

Ten je po instalaci Ubuntu vypnutý. Můžeme ho povolit i dočasně zakázat jednoduchými příkazy::

    $ sudo ufw enable
    $ sudo ufw disable

Pravidla
--------

Po zapnutí použije defaultní sadu pravidel, která bude vyhovovat většině domácích uživatelů -
povolen ping, zakázána všechna příchozí spojení s výjimkou portů 21 (ftp), 554 (rstp - domácí
streaming), 7070 (RealAudio streaming), povolena všechny odchozí spojení.

Teď i později můžete použít tyto příkazy pro více informací::

    $ sudo ufw status verbose        # detailní stav firewallu
    $ sudo ufw status raw            # výpis ve formátu iptables
    $ sudo ufw status numbered       # pravidla firewallu

Pravidla povolení i zákazu mají stejnou obecnou syntaxi::

    $ sudo ufw (allow|deny) <port>/[protocol]

Pokročilejší podoba pro určení zdrojového hostitele a cílového portu::

    $ sudo ufw (allow|deny) from <target> to any port <port-number>

Příklady povolujících pravidel::

    # Povolení portu 80 (viz /etc/services)
    $ sudo ufw allow http
    $ sudo ufw allow 3306

    # Povolení UDP provozu na portu 2807
    $ sudo ufw allow 2807/udp

    # Povol příchozí provoz od dané IP
    $ sudo ufw allow from 207.46.232.182

    # Můžete použít i netmask sítě
    $ sudo ufw allow from 192.168.1.0/24

    # Povolení zdrojové IP přístup jen na port 22
    $ sudo ufw allow from 192.168.0.4 to any port 22

Příklady pravidel zákazu::

    $ sudo ufw deny ssh
    $ sudo ufw deny 3306
    $ sudo ufw deny 2807/udp
    $ sudo ufw deny from 192.168.0.1
    $ sudo ufw deny from 192.168.0.1 to any port 22

Pro smazání pravidla musíte nejprve zjistit její číslo v ufw status numbered a potom

::

    $ sudo ufw delete <čísloPravidla>

.. caution:: Pořadí vyhodnocování pravidel

   Při hledání důvodu, proč provoz neprochází firewallem se často zapomíná, že jakmile je
   nalezeno vyhovující pravidlo, tak se další již nevyhodnocují. Jinými slovy **nejprve vytvořte
   specifičtější pravidlo a teprve potom obecnější**.

Další dovednosti
----------------

Defaultně UFW loguje svůj provoz do souborů ``/var/log/ufw*``. Logování můžete kdykoli vypnout nebo
zapnout::

    $ sudo ufw logging [off|on]

Kdyby se chtěli vrátit do výchozího stavu můžete UFW resetovat pomocí

    $ sudo ufw reset

Iptables
========

Nástroj pro nastavování pravidel firewallu v jádře. Pro složitost se nebudeme zabývat. Nahradil
zastaralý ipchains.

UFW je nadstavbou nad iptables, která skrývá složitost iptables úkolů a spolehlivě postačí na
většinu požadavků na nastavení firewallu.

GUI nadstavby
=============

* GUFW - nadstavba nad UFW
* Fwbuilder
* Shorewall
* Firestarter
* Lokkit

************
TCP wrappers
************

* ``/etc/hosts.allow`` a ``hosts.deny``
* aplikace musí přístup konzultovat s TCP wrapper vrstvou, což už příliš mnoho aplikací nedělá
* iptables/ufw jsou mnohem efektivnějším řešením fungujícím vždy

Je aplikace kompatibilní ap.?
https://ubuntu-tutorials.com/2007/09/02/network-security-with-tcpwrappers-hostsallow-and-hostsdeny/

********
AppArmor
********

* rozšíření jádra pro řízení přístupu
* umožňuje definovat oprávnění k provedení určité operace na úrovni jednotlivých procesů v
  závislosti na umístění spustitelného souboru tím, že na kritická místa jádra umisťuje volání svých
  kontrolních rutin. Tím dochází ke zvýšení režie systému, avšak je možné zabránit programu, aby
  provedl potenciálně nebezpečnou akci, která může vést k narušení bezpečnosti (včetně elevace
  oprávnění).
* podobný SELinuxu v Red Hat