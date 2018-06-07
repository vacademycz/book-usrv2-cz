##########################
Řízení a plánování procesů
##########################

.. todo:: Navazujeme na pojednání o procesech z I. a přepokládáme, že tam popsané základní
   informace znáte.

*********************
Vznik a zánik procesů
*********************

Proces může vyvolat nový proces a ten další proces. Nový proces je kopií svého rodiče - liší se jen
PID číslem, ale jinak má stejné prostředí, vstupy a výstupy, proměnné prostředí a prioritu

Tyto stromové vazby zobrazíte programem ``pstree``.

.. code-block:: text
   :caption: Výpis programu pstree bez parametrů. Výstup na Ubuntu 16.04. Ve starším Ubuntu 14.04
        je kořenovým procesem "init".

   $ pstree
   systemd─┬─ModemManager───2*[{ModemManager}]
           ├─NetworkManager─┬─dhclient
           │                ├─dnsmasq
           │                └─3*[{NetworkManager}]
           ├─accounts-daemon───2*[{accounts-daemon}]
           ├─gnome-keyring-d───5*[{gnome-keyring-d}]
           ├─irqbalance
           ├─kerneloops
           ├─lightdm─┬─Xorg───2*[{Xorg}]
           │         ├─lightdm─┬─init─┬─/usr/bin/termin─┬─bash───pstree
           │         │         │      │                 ├─gnome-pty-helpe
           │         │         │      │                 └─4*[{/usr/bin/termin}]
           │         │         │      ├─AsciidocFX───java───81*[{java}]
           │         │         │      ├─GoogleTalkPlugi───7*[{GoogleTalkPlugi}]
           │         │         │      ├─at-spi-bus-laun─┬─dbus-daemon
           │         │         │      │                 └─3*[{at-spi-bus-laun}]
   ...

.. todo:: "zabití" přepsat na "zaslání signálu k zabití", odkaz do I.

Když proces skončí očekávaným způsobem (nedojde k jeho zabití nebo havárii), může rodičovskému
procesu vrátit návratový kód. Zda a jaký význam jednotlivým číselným kódům bude rodič přikládat se
různí. Podle konvence vše kromě nuly obvykle indikuje chybu.

.. _init-systemy:

************
Init systémy
************

.. rubric:: System V (nebo SysV)

Na výpisech pstree jistě povšimnete, že všechny "skutečné" procesy jsou potomky procesu systemd. Tak
se jmenuje od Ubuntu 15.04 výchozí init manager. Tento úplně první proces spuštěný jádrem proto
dostane PID vždy 1 a je přímým či nepřímým rodičem všech ostatních procesů. Prvotní proces init má
historicky několik implementací. Tradičně všechny vycházejí z tzv. System V init (SysV).

.. rubric:: Upstart

Ubuntu dlouho používalo svůj vlastní init manager Upstart, který vznikl přímo na míru Ubuntu.
Používala ho však jistou dobu i konkureční Fedora a stále používá např. Chrome OS.

.. rubric:: systemd

V Ubuntu 15.04 došlo k důležité, i když na první pohled težko postřehnutelné, změně - Upstart byl
nahražen manažerem systemd.

.. note:: Autoři systemd si přejí, abychom název důsledně psali jako systemd, nikoli SystemD,
   System D ap. Pokud bude název na začátku věty, budeme však používat Systemd.

Systemd je taktéž výchozím init systémem Debianu 8 (Jessie) a vyšší.

Tato změna na systemd byla poměrně silně kontroverzní a řada uživatelů Ubuntu i Debian rozhodnutí
považuje za špatné. Důvod k této nepopularitě je především podle názoru odpůrců zbytečná složitost,
a příliš široký záběr, který porušuje unixová pravidla menších jednodušších nástrojů. systemd je
však bezesporu nejpokročilejší init systém nabízející agresivní paralelní provádění, bohatou
konfiguraci, logovací služby, správu mount a automount pointů a další.

Příjmání systemd usnadňuje, že je koncipován jako náhrada za sysvinit a naštěstí tedy podporuje
tradiční SysV init skripty.

V důsledku tohoto historického vývoje najdete v Ubuntu vše - znát bychom měli tradiční SysV skripty,
jejich Upstart ekvivalent tzv. joby i nově se rozšiřující targets jak skriptům říká systemd.

.. csv-table:: Porovnání SysV, Upstart a systemd
   :header: "", "SysV", "Upstart", "systemd"
   :stub-columns: 1

   "konfiguraci služby se říká", "skript", "job", "unit"
   "hlavní složka s konfigurací služeb", "/etc/init.d/", "/etc/init/", "/etc/systemd/"
   "výchozí init systém od", "", "", "v Ubuntu od 15.04 (Vivid), v Debianu od 8 (Jessie)"

Upstart
=======

.. caution:: Vzhledem k tomu, že se Upstart již od Ubuntu 15.04 neupoužívá a v Debianu se
  nepoužíval nikdy, zmíníme Upstart jen okrajově. Případné zájemce odkazujeme na
  http://upstart.ubuntu.com/.

Upstart konfiguraci načítal ze složky ``/etc/init/``, kde hledá textové soubory s koncovkou
``.conf``. Jeden soubor pro jednu službu (zvanou v Upstartu job). Obvykle obsahuje něco málo
dokumentace, a zejm. jak proces nastartovat a ukončit, ve kterých runlevech ap. Více se o podobě
souborů dozvíte v ``man 5 init``.

Tradiční System V init používal ke konfiguraci soubor ``/etc/inittab``. V Ubuntu s Upstartem
nepoužívá a ani neexistuje.

.. _sysv_init:

System V init
=============

System V je natolik tradiční, že Upstart i systemd mají podporu pro tradiční System V init skripty
umisťované do složky ``/etc/init.d/``. Tento typ init skriptů je široce rozšířený a díky podpoře
pozdějších init systémů můžeme nadále zůstat u těchto skriptů. Rovněž množství DEB balíčků je
dodáváno s SysV init skripty.

.. note:: Bohužel se liší (i když ne zásadně) styl psaní Debian/Ubuntu a Red Hat init.d skriptů. My
   se zaměříme na Debianí styl.

Vytvoření démonu znamená vytvoření (Bash) skriptu v /etc/init.d/. Ve složce existuje šablona
skeleton, kterou použijte pro vytvoření nového démonu::

    cp /etc/init.d/skeleton /etc/init.d/helloworld
    sudo chmod 775 /etc/init.d/helloworld

Skripty by měli začínat tzv. *shebang* řádkem, pak musí obsahovat hlavičku s povinnými a nepovinnými
údaji, a zbytek souboru jsou běžné příkazy pro spuštění služby.

.. code-block:: bash
   :caption: Ukázka init.d skriptu

   #!/bin/sh
   
   ### BEGIN INIT INFO
   # Provides:          samba
   # Required-Start:    smbd nmbd
   # Required-Stop:     smbd nmbd
   # Default-Start:
   # Default-Stop:
   # Short-Description: ensure Samba daemons are started (nmbd and smbd)
   ### END INIT INFO
   
   ...příkazy pro spuštění služby...


.. todo:: "startovač": odkaz do I. na "service"

Zbytek skriptu je obsluha parametrů "startovače" service: start, stop, restart, reload, force-reload
a status odpovídající bash funkcím ``do_*()`` ve skriptu. Např.:

.. code-block:: bash

   ....
   
   do_start()
   {
           # Return
           #   0 if daemon has been started
           #   1 if daemon was already running
           #   2 if daemon could not be started
           start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON --test > /dev/null \
                   || return 1
           start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON -- \
                   $DAEMON_ARGS \
                   || return 2
           # Add code here, if necessary, that waits for the process to be ready
           # to handle requests from services started subsequently which depend
           # on this one.  As a last resort, sleep for some time.
   }
   
   ...

.. _update-rc.d:

Instalace init.d skriptů pomocí ``update-rc.d``
-----------------------------------------------

Přestože vytvoření a mazání init.d skriptů se nezdá těžké, použijte nástroj ``update-rc.d``.

.. note:: Důvod souvisí s :ref:`runlevely (úrovně běhu) <runlevel>`, protože ``update-rc.d`` umí
   především aktualizovat symlinky ``NNjméno`` v složkách ``/etc/rc?.d/``. Nástroji jen parametrem
   určíte ve kterých runlevelech se má služba spouštět. Manipulace v těchto složkách ručně by byla
   náročná a náchylná na chyby (překlep v názvu, opomenutí symlinku, ...).

Ke všem příkazům můžete přidat ``-n``, aby se povel neprovedl, ale jen se vypsalo, co by se provedlo
("běh na sucho").

Nejčastější použití je::

    $ update-rc.d <služba> defaults

např.::

    $ update-rc.d helloworld defaults

pro vytvoření služby ``helloworld``. Jméno služby musí odpovídat jménu souboru v ``/etc/init.d/``,
tj. ze skriptu ``/etc/init.d/helloworld``. Vytvoření znamená ve skutečnosti vytvoření linků v
``/etc/rc?.d/`` složkách. Volba ``defaults`` znamená start pro runlevely 2, 3, 4 a 5; a stop pro
runlevely 0, 1 a 6.

Smazání smaže všechny linky z ``/etc/rc?d/`` složek. Skript v ``/etc/init.d/`` musí být odstraněn
ručně, jinak se vypíše chyba::

    $ update-rc.d helloworld remove

.. hint:: Viz ``man update-rc.d``.

.. todo:: "spouštečem": odkaz do I. na "service"

Ovládání služby (spuštění, zastavení, ...) provádíme přímo spuštěním skriptu nebo (lépe) spouštečem
služeb service.

.. _runlevel:

Runlevel (úroveň běhu)
======================

Tradiční System V podporuje koncept zvaný runlevel, kdy určité služby běží jen v určité úrovni běhu,
protože v jiné nejsou třeba. Např. při opravě poškozeného serveru, jinak obsluhujícího mnoho
uživatelů, se při spuštění do runlevel 1 nepřihlásí nikdo kromě vás.

Runlevely jsou dnes se systemd zastaralým způsobem, jak spouštět a zastavovat skupiny SysV služeb,
ale kvůli zpětné kompatibilitě stále fungují. Pro seskupování služeb v systemd slouží tzv. targets
(cíle). Systemd dokáže provozovat více cílů, může být v SystemV aktivní jen jeden runlevel v jeden
okamžik. Srovnání runlevelu a targetu v následující tabulce je tedy spíše orientační.

.. csv-table:: Runlevely v Debian/Ubuntu a jejich odpovídající systemd target
   :header: "SysV runlevel", "systemd target", "význam"

   "0", "poweroff.target", "zastavení"
   "1", "rescue.target", "jednouživatelský mód"
   "2, 3, 4", "multi-user.target", "víceuživatelský režim"
   "5", "graphical.target", "grafický víceuživatelský režim"
   "6", "reboot.target", "reboot"

Předchozí a aktuální runlevel oddělený mezerou vám řekne příkaz runlevel. Pokud není předchozí
runlevel znám, vypíše se "N"::

    $ runlevel
    N 2

/etc/rc?.d/ složky
------------------

.. warning:: S obsahem těchto složek raději nemanipujte ručně, ale s pomocí nástroje
   :ref:`update-rc.d`.

V /etc/ se nachází několik složek pojmenovaných ``rc?.d``, kde ? je číslo odpovídající runlevelu.

.. code-block:: text
   :caption: rc?.d/ složky

   $ ls -d /etc/rc?.d/
   /etc/rc0.d  /etc/rc2.d  /etc/rc4.d  /etc/rc6.d
   /etc/rc1.d  /etc/rc3.d  /etc/rc5.d  /etc/rcS.d

Složka ``/etc/rcS.d/`` je pro skripty spouštěné výhradně při bootování systému. Všechny ostatní jsou
číslovány podle runlevelu ve kterém jsou skripty vykonány.

.. todo:: "symlinky": odkaz do I.

Tyto složky neobsahují skutečné soubory, ale jen symlinky na skripty v ``/etc/init.d/``. To proto,
aby bylo možné přidávat a mazat skripty odpovídající služeb do více úrovní současně a bez ovlivnění
init.d skriptů samotných.

Odkazy mají prefix K pro skript při ukončení a S při vstupu do runlevelu. Po K/S následuje číslo a
jméno. Číslo určuje pořadí provedení. Je-li stejného čísla více souborů, seřadí se abecedně a proto
jméno nemá jen popisný charakter.

.. code-block:: text
   :caption: Ukázka obsahu složky /etc/rc4.d/

   $ ls -l /etc/rc6.d/
   total 4
   lrwxrwxrwx 1 root root  17 bře 22  2015 K09apache2 -> ../init.d/apache2
   lrwxrwxrwx 1 root root  29 úno 23  2015 K10unattended-upgrades -> ../init.d/unattended-upgrades
   lrwxrwxrwx 1 root root  20 úno 23  2015 K20kerneloops -> ../init.d/kerneloops
   lrwxrwxrwx 1 root root  15 úno 23  2015 K20rsync -> ../init.d/rsync
   lrwxrwxrwx 1 root root  27 úno 23  2015 K20speech-dispatcher -> ../init.d/speech-dispatcher
   lrwxrwxrwx 1 root root  15 úno 23  2015 K21mysql -> ../init.d/mysql
   lrwxrwxrwx 1 root root  31 úno 24  2015 K65vboxautostart-service -> ../init.d/vboxautostart-service
   lrwxrwxrwx 1 root root  33 úno 24  2015 K65vboxballoonctrl-service -> ../init.d/vboxballoonctrl-service
   lrwxrwxrwx 1 root root  25 úno 24  2015 K65vboxweb-service -> ../init.d/vboxweb-service
   lrwxrwxrwx 1 root root  17 úno 24  2015 K80vboxdrv -> ../init.d/vboxdrv
   -rw-r--r-- 1 root root 351 bře 13  2014 README
   lrwxrwxrwx 1 root root  18 úno 23  2015 S20sendsigs -> ../init.d/sendsigs
   lrwxrwxrwx 1 root root  17 úno 23  2015 S30urandom -> ../init.d/urandom
   lrwxrwxrwx 1 root root  22 úno 23  2015 S31umountnfs.sh -> ../init.d/umountnfs.sh
   lrwxrwxrwx 1 root root  18 úno 23  2015 S40umountfs -> ../init.d/umountfs
   lrwxrwxrwx 1 root root  20 úno 23  2015 S60umountroot -> ../init.d/umountroot
   lrwxrwxrwx 1 root root  16 úno 23  2015 S90reboot -> ../init.d/reboot

.. _systemd:

Systemd
=======

.. note:: Systemd je rozsáhlý software, který řeší daleko více úkolů, než pouze provoz služeb.
  Zahrnuje např. správu mount pointů, uživatelských sessions (sezení), logování a několik dalších.
  Popis věškeré jeho funkcionality by vydal na samostnou příručku a školení. V této knize se
  zaměříme na to nejpotřebnější z pohledu správce - správu služeb.

.. note:: Systemd má vlastní bohatou terminologii jako target, unit ap. Možný český překlad
   uvedeme v závorce při prvním výskytu. Nadále však budeme používat počeštěné anglické názvy jako
   "několik targetů" místo kompletně českého překladu "několik cílů", který by byl podle našeho
   názoru více zmatením, než přínosem.

Hlavním programem pro komunikaci se systemd se jmenuje ``systemctl``. Systemd dokáže ovládat více druhů
tzv. *units (jednotek)* podle typu spravovaného zdroje. Konfigurace unitů je v textových souborech,
které mají syntaxi velmi podobou INI souborům. Přípona souboru určuje typ unitu. Např. ``cups.service``
je konfigurací služby CUPS nebo ``graphical.target`` targetu "graphical". Protože nejčastější jsou
servisy, tak pokud příponu neuvedete systemd doplní ``.service``.

.. topic:: Šetřete prsty - alias pro sudo systemctl

   Program ``systemctl`` skoro ve všech případech ho musíme zavolat jako root (se sudo). Vypisování
   tak často používateného programu s celkem dlouhým názvem je skvělým kandidátem na alias.
   
   Do vašeho ``~/.bash_aliases`` (soubor případně vytvořte) přidejte alias např. pojmenovaný
   ``sc``::
   
       $ echo alias sc='sudo systemctl' >> ~/.bash_aliases

   ukončete současný terminál, spusťe nový a místo např. ``sudo systemctl list-units`` můžete psát
   jen ``sc list-units``.

   Nevýhoda je, že tím přicházíte o poměrně propracovaný :kbd:`Tab` auto-completion.

Units (jednotky)
----------------

Units jsou pro systemd obecné jednotky mezi kterými je možné určovat vzájemné závislosti. Existuje
12 druhů unitů podle spravovaného zdroje. Mezi nejdůležitější však patří:

* *service units* - startují a kontolují démony - pro nás nejdůležitější druh unitu
* *socket units* - zapouzdřují lokální IPC nebo síťové sokety
* *target units* - seskupení více units
* *device units* - vystavují zařízení jádra
* *mount units* - kontrolují přípojné body

Unit mohou mohou nacházet v těchto 5 základních stavech:

* *activating* (v průběhu aktivace) a *active* (již aktivované)
* *deactivating* (v průběhu inactivace) a *inactive* (již inaktivované)
* *failed* (nepodařilo se aktivovat)

Unit určitého typu mohou mít vlastní "podstavy".

Spouštění, zastavení a restart služeb
-------------------------------------

K ovládání všech typů unit slouží program ``systemctl``, který jako povinný parametr potřebuje
prováděnou operaci, jednotku. Pokud není uvedena přípona přepokládá se .service. Obecná syntaxe je
tedy::

    sudo systemctl <operace> <unit>[.service]

Pro základní ovládání služeb používáme operace

* ``start`` pro spouštění unit
* ``stop`` pro zastavení unit
* ``restart`` pro restart unit

Např. zastavení služby CUPS zařídíme pomocí::

    sudo systemctl stop cups

Některé unit mohou podporovat tzv. ``reload`` neboli načtení konfigurace bez výpadku služby.
Užitečná operace je ``reload-or-restart``, která zkusí ``reload`` a pokud není unitou podporovanán
udělá ``restart``.

::

    sudo systemctl reload-or-restart cups

Povolení a zakázání služeb
--------------------------

Operace start, stop a další výše popsané znamenají "proveď teď". Aby se unit spustil automaticky při
startu ho musíme povolit pomocí operace enable::

    sudo systemctl enable <nazev_sluzby>

.. todo:: "symlinku" - link do I.

Povolení znamená vytvoření symlinku výchozí konfigurace služby (obvykle ve složce
``/lib/systemd/system/``) do místa, kde systemd očekává units aktivované při bootování (obvykle ve
složce ``/etc/systemd/system/<nejaky_target>.target.wants``). Více o :ref:`targets` za okamžik.

Opačná operace disable::

    sudo systemctl disable <nazev_sluzby>

odstraní symlinky vytvořené při povolení služby.

.. important:: Povolení/zakázání neznamená spouštění/zastavení služby v aktuální session. Chcete-li
   službu povolit i spustit musíte použít dva příkazy - enable následované start.

Konfigurace služeb
------------------

Systemd hledá jednotky na dvou hlavních místech. Konfigurace tak, jak vypadají ve výchozím stavu,
byste našli ve složce ``/lib/systemd/system/``. Např. ``cups.service`` soubor služby CUPS
zajišťující tisk má systémovou definici zde::

    $ ll $(find /lib/systemd/system -name cups.service)
    -rw-r--r-- 1 root root 175 úno 16 21:46 /lib/systemd/system/cups.service

Definice CUPS služby určená k přizpůsobení správcem je ve složce ``/etc/systemd/system/``. Tam najdete
kopii ``cups.service`` a symlink na originál v ``/lib/systemd/system/`` ze složky cíle
``printer.target.wants/``::

    $ ll $(find /etc/systemd/system -name cups.service)
    -rw-r--r-- 1 root root 175 úno 16 21:46 /etc/systemd/system/cups.service
    lrwxrwxrwx 1 root root  32 bře  5 18:13 /etc/systemd/system/printer.target.wants/cups.service -> /lib/systemd/system/cups.service

Stav služby
-----------

Detailní informaci o stavu služby poskytne operace ``status``. Z výstupu zjistíme kromě stavu
cgroups hierarchii a užitečné je i několik prvních řádků z logu::

    $ sudo systemctl status docker
    ● docker.service - Docker Application Container Engine
    Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
    Active: active (running) since Čt 2017-03-02 13:58:48 CET; 3 days ago
        Docs: https://docs.docker.com
    Main PID: 11667 (dockerd)
        Tasks: 56
    Memory: 889.2M
        CPU: 10min 14.811s
    CGroup: /system.slice/docker.service
            ├─11667 /usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:4243
            ├─11674 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-
            ├─14172 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 32769 -container-ip 172.17.0.2 -container-port 80
            └─14177 docker-containerd-shim e71dee1a12ebf7579765d53b1d15376c4a5f186c5644d60a768e5167b7bbbf6f /var/run/docker/libcon

    Mar 04 08:32:59 jell-nb dockerd[11667]: time="2017-03-04T08:32:59.872897132+01:00" level=info msg="IPv6 enabled; Adding default I
    Mar 04 18:00:44 jell-nb systemd[1]: Started Docker Application Container Engine.
    Mar 04 18:03:40 jell-nb systemd[1]: Reloading Docker Application Container Engine.
    Mar 04 18:03:40 jell-nb dockerd[11667]: time="2017-03-05T18:03:40.911321154+01:00" level=info msg="Got signal to reload configura
    Mar 04 18:03:40 jell-nb dockerd[11667]: time="2017-03-05T18:03:40.911382634+01:00" level=error msg="open /etc/docker/daemon.json:
    Mar 04 18:03:40 jell-nb systemd[1]: Reloaded Docker Application Container Engine.
    Mar 04 18:04:03 jell-nb systemd[1]: Reloading Docker Application Container Engine.
    Mar 04 18:04:03 jell-nb dockerd[11667]: time="2017-03-05T18:04:03.078121402+01:00" level=info msg="Got signal to reload configura
    Mar 04 18:04:03 jell-nb dockerd[11667]: time="2017-03-05T18:04:03.078275229+01:00" level=error msg="open /etc/docker/daemon.json:
    Mar 04 18:04:03 jell-nb systemd[1]: Reloaded Docker Application Container Engine.

Postačí-li odpověď, zda je služba aktivní použijte::

    $ sudo systemctl is-active docker
    active

nebo, zda je povolená::

    $ sudo systemctl is-enabled docker
    enabled

Výpis aktivních nebo všech units
--------------------------------

Ve výpisu všech aktivních jednotek v systemd operací ``list-units`` si všimnete růzdných typů unit,
které poznáte podle přípony (zkráceno)::

    sudo systemctl list-unit

Výpis aktivních jednotek je výchozí operací, takže stejný výstup nám dá i jen ``systemctl``
(zkráceno)::

    $ sudo systemctl
    UNIT                                                LOAD   ACTIVE SUB       DESCRIPTION
    proc-sys-fs-binfmt_misc.automount                   loaded active waiting   Arbitrary Executable File Formats File System Autom
    sys-devices-pci0000:00-0000:00:02.0-drm-card0-card0\x2deDP\x2d1-intel_backlight.device loaded active plugged   /sys/devices/pci
    sys-devices-pci0000:00-0000:00:14.0-usb1-1\x2d6-1\x2d6:1.0-bluetooth-hci0.device loaded active plugged   /sys/devices/pci0000:0
    ....
    snap-core-1264.mount                                loaded active mounted   Mount unit for core
    snap-core-1287.mount                                loaded active mounted   Mount unit for core
    snap-core-1337.mount                                loaded active mounted   Mount unit for core
    snap-keepassx\x2delopio-1.mount                     loaded active mounted   Mount unit for keepassx-elopio
    ...
    init.scope                                          loaded active running   System and Service Manager
    session-c2.scope                                    loaded active running   Session c2 of user jell
    accounts-daemon.service                             loaded active running   Accounts Service
    acpid.service                                       loaded active running   ACPI event daemon
    alsa-restore.service                                loaded active exited    Save/Restore Sound Card State
    apparmor.service                                    loaded active exited    LSB: AppArmor initialization
    apport.service                                      loaded active exited    LSB: automatic crash report generation
    avahi-daemon.service                                loaded active running   Avahi mDNS/DNS-SD Stack
    bluetooth.service                                   loaded active running   Bluetooth service
    ...
    LOAD   = Reflects whether the unit definition was properly loaded.
    ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
    SUB    = The low-level unit activation state, values depend on unit type.

    231 loaded units listed. Pass --all to see loaded but inactive units, too.
    To show all installed unit files use 'systemctl list-unit-files'.

Jednotlivé sloupce mají tento význam:

* *UNIT* - název systemd unity
* *LOAD* - stav načtení unity
* *ACTIVE* - je unit aktivní?
* *SUB* - podstav (substate) unitu. Každá unita může mít své vlastní podstavy
* *DESCRIPTION* - popis účelu unity

Vypisované unity můžeme omezit na určitý typ pomocí ``--type``, např.::

    sudo systemctl list-units --type=service

Výše použité ``list-units`` vypisuje jen na units, které se systemd pokusil načíst. Pro zobrazení
všech unitů, které systemd zná slouží ``list-unit-files``::

    $ sudo systemctl list-unit-files
    UNIT FILE                                  STATE
    proc-sys-fs-binfmt_misc.automount          static
    proc-sys-fs-binfmt_misc.mount              static
    snap-core-1264.mount                       enabled
    snap-core-1287.mount                       enabled
    snap-tpad-17.mount                         enabled
    sys-fs-fuse-connections.mount              static
    alsa-utils.service                         masked
    anacron-resume.service                     enabled
    anacron.service                            enabled
    apport-forward@.service                    static
    apt-daily.service                          static
    autovt@.service                            enabled
    avahi-daemon.service                       enabled
    ...

Sloupec *STATE* může nabývat těchto hodnot:

* *enabled* - unit je možné používat (neznamená, že je aktivní)
* *disabled* - unit je zakázaná
* *static* - unit nemůže být samostatně použita, ale jen jako např. závilost jiné unit
* *masked* - unit je :ref:`zamaskovaná <systemd-maskovani>` a neschopná startu

.. _systemd-maskovani:

Maskování služeb
----------------

Ve výstupu ``list-unit-files`` jste si mohli povšímnout stavu *masked*. Službu (nebo obecně unit),
který označíte jako maskovaný je nemožné automaticky i manuálně spustit. Systemd toho dosáhne opět
pomocí symlinků, kdy unit nacílí na ``/dev/null``.

Řekne nám to ``systemctl``, když službu zamaskujeme operací ``mask``::

    $ sudo systemctl mask cups
    Created symlink from /etc/systemd/system/cups.service to /dev/null.

Pokus o start se nám pak nepovede::

    $ sudo systemctl start cups
    Failed to start cups.service: Unit cups.service is masked.

Dokud ji neodmaskujeme s ``unmask``::

    $ sudo systemctl unmask cups
    Removed symlink /etc/systemd/system/cups.service.

Vytváření služeb
----------------

V následujícím cvičení vytvoříme minimální, ale plně funkční službu, která bude do souboru zapisovat
aktuální datum.

Začneme napsáním skriptu, který zjistí aktuální datum a zapíše ho do souboru. Skript můžete napsat v
jakémkoli oblíbeném jazyce - např. v Bashi nebo Pythonu. My zvolíme Bash. Někde na disku, např. ve
vaší domovské složce, vytvořte spustitelný soubor ``today.sh``::

    $ touch /home/joe/today.sh
    $ chmod +x /home/joe/today.sh

a pomocí editoru (např. nano) vložte do souboru tento obsah::

    #!/bin/bash
    while true; do sleep 15; date -I > /tmp/today.txt; done

Vytvořte minimální definici služby ve složce ``/etc/systemd/system/``::

    sudo nano /etc/systemd/system/today.service

s tímto obsahem::

    [Unit]
    Description=Today date service

    [Service]
    ExecStart=/home/joe/today.sh

    [Install]
    WantedBy=multi-user.target

Příkaz ``ExecStart`` je příkaz, který se spustí při aktivaci služby.

.. note:: Kromě odkazu na náš soubor skriptu by se takto jednoduchý skript nechal zapsat přímo v
   ``.service`` souboru::

       ...
       [Service]
       ExecStart=/bin/bash -c "while true; do sleep 15; date -I > today.txt; done"
       ...

Druhá klíčová věc v definici služby je ``WantedBy`` určující v jakém targetu očekáváme spuštění této
služby.

Každopádně zbývá již jen povolení služby neboli vytvoření odpovídajích symlinků::

    $ sudo systemctl enable today.service
    Created symlink from /etc/systemd/system/multi-user.target.wants/today.service to /etc/systemd/system/today.service.

A nyní už jen restart do určeného targetu nebo spuštění můžeme provést okamžitě::

    sudo systemctl start today

Případně si ověřit, že jsme neprovedli někde chybu::

    $ sudo systemctl status today
    ● today.service - Today date
       Loaded: loaded (/etc/systemd/system/today.service; enabled; vendor preset: enabled)
       Active: active (running) since Sun 2017-03-05 22:34:41 CET; 31s ago
     Main PID: 8409 (today.sh)
        Tasks: 2
       Memory: 616.0K
          CPU: 2ms
       CGroup: /system.slice/today.service
               └─829 python3 /bin/bash /home/joe/today.sh
               └─837 sleep 15

    Mar 05 22:34:41 vbox-joe systemd[1]: Started Today date service.

    $ cat /tmp/today.txt
    2017-03-05

.. _targets:

Targets (cíle)
--------------

Targety jsou obdobou :ref:`runlevelů <runlevel>` v System V. Slouží hlavně pro seskupení ostatních
druhů unitů. Konfigurační soubory mají příponu ``.target``. Seskupením jiných unitů do targetu
nastavujeme stav celého systému. Aktivace určitého targetu znamená aktivaci všech unitů v targetu.

.. rubric:: Definice targetů

Např. target ``multi-user.target`` odpovídající zhruba runlevelům 2, 3 a 4 je definován takto::

    $ sudo systemctl cat multi-user.target
    # /lib/systemd/system/multi-user.target
    #  This file is part of systemd.
    #
    #  systemd is free software; you can redistribute it and/or modify it
    #  under the terms of the GNU Lesser General Public License as published by
    #  the Free Software Foundation; either version 2.1 of the License, or
    #  (at your option) any later version.

    [Unit]
    Description=Multi-User System
    Documentation=man:systemd.special(7)
    Requires=basic.target
    Conflicts=rescue.service rescue.target
    After=basic.target rescue.service rescue.target
    AllowIsolate=yes

Většinou nám více poví operace ``list-dependencies``, která sestaví strom závislostí jakékoli
jednotky. Použito pro náš ``multi-user.target`` zjistíme (zkráceno)::

    $ sudo systemctl list-dependencies multi-user.target
    multi-user.target
    ● ├─anacron.service
    ● ├─apport.service
    ● ├─avahi-daemon.service
    ● ├─cgroupfs-mount.service
    ● ├─cron.service
    ● ├─cups-browsed.service
    ● ├─cups.path
    ● ├─dbus.service
    ● ├─dns-clean.service
    ● ├─docker.service
    ● ├─flexibee.service
    ● ├─glances.service
    ● ├─grub-common.service
    ● ├─hddtemp.service
    ...

.. rubric:: Výchozí target

Výchozí target můžeme zjistit pomocí ``systemctl get-default``. Na desktopové stanici to bude
nejspíš ``graphical.target``::

    $ sudo systemctl get-default
    graphical.target

Změnit výchozí target můžete pomocí ``systemctl set-default``::

    sudo systemctl set-default multi-user.target

.. rubric:: Dostupné a aktivní targety

Všechny dostupné targety zjistíte známým příkazem ``systemctl list-unit-files --type=target``. Na
rozdíl od runlevelů může být targetů aktivních v jeden moment více - důkaz nám poskytne::

    $ sudo systemctl list-units --type=target
    UNIT                   LOAD   ACTIVE SUB    DESCRIPTION
    basic.target           loaded active active Basic System
    bluetooth.target       loaded active active Bluetooth
    cryptsetup.target      loaded active active Encrypted Volumes
    getty.target           loaded active active Login Prompts
    graphical.target       loaded active active Graphical Interface
    local-fs-pre.target    loaded active active Local File Systems (Pre)
    local-fs.target        loaded active active Local File Systems
    multi-user.target      loaded active active Multi-User System
    network-online.target  loaded active active Network is Online
    network-pre.target     loaded active active Network (Pre)
    network.target         loaded active active Network
    nss-user-lookup.target loaded active active User and Group Name Lookups
    paths.target           loaded active active Paths
    remote-fs-pre.target   loaded active active Remote File Systems (Pre)
    remote-fs.target       loaded active active Remote File Systems
    slices.target          loaded active active Slices
    sockets.target         loaded active active Sockets
    sound.target           loaded active active Sound Card
    swap.target            loaded active active Swap
    sysinit.target         loaded active active System Initialization
    time-sync.target       loaded active active System Time Synchronized
    timers.target          loaded active active Timers

    LOAD   = Reflects whether the unit definition was properly loaded.
    ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
    SUB    = The low-level unit activation state, values depend on unit type.

    22 loaded units listed. Pass --all to see loaded but inactive units, too.
    To show all installed unit files use 'systemctl list-unit-files'.

.. rubric:: Izolování targetů

Občas se může hodit spustit jen unity v targetu a zastavit všechny ostatní z dalších targetů. K tomu
slouží operace ``isolate``. Je to dost podobné změně runlevelů v System V.

Např. přepnutí na ``rescue.target`` a vypnutí všech units, které nepatří do tohoto targetu napíšte
jako::

    sudo systemctl isolate rescue.targets
    
Systemd má pro izolaci několik čas urychlujících zkratek. Např.

* místo ``systemctl isolate rescue.target`` můžete psát jen ``systemctl rescue``
* místo ``systemctl isolate default`` můžete psát jen ``systemctl default``

Další zkratky jsou pro zastavení systému::

    sudo systemctl halt

Vypnutí systému::

    sudo systemctl poweroff

Restart systému::

    sudo systemctl reboot

Tyto zkratky jsou vhodnější, než pouhá izolace na určený target a dovedou informovat přihlášené
uživatele, že se bude stroj např. restartovat.

.. todo:: "programy pro ukončení a restart PC" odkaz do I.

Pro ještě větší komfort původně samostatné programy pro ukončení a restart PC halt, poweroff a
reboot volají ve skutečnosti tyto příkazy systemd.

**************
Plánování úloh
**************

Chcete-li pozdržet spuštění procesu, použijte ``sleep``. Chcete-li ho odložit, pak ``at``. Chcete-li
ho naplánovat na za týden, 2x měsíčně atd., využijte cron.

sleep
=====

Sleep je velmi jednoduchý - prostě nic nedělá zadaný počet sekund. Hodí se to hlavně do skriptů, kdy
sleep "pozastaví" jeho provádění. Nebo si chcete nechat něco připomenout za 10 minut::

    (sleep 10m; echo 'Miluju Linux!')&

Číslo bez označení nebo číslo s příponou "s" znamená počet sekund (např. 30s). Dále je možné použít
"m" pro minuty, "h" pro hodiny, "d" pro dny.

at
==

S at naplánujete spuštění v určitý čas nebo za určitý interval. Definice "kdy" je velmi lidská,
např.::

    $ at now + 2 hours
    $ at tomorrow + 4 days
    $ at 10:39

.. todo:: "promptu" odkaz do I. na přikazová řádka, nadpis Bash prompt. "EOF" do I. na příkazová
   řádka na "Kl. zkratka Ctrl-D (EOF)".

Na to se objeví prostředí podobné promptu, kde můžete zadat jakékoli příkazy ke spuštění v daný čas.
Sekvenci příkazů ukončíte znakem EOF neboli stiskem Ctrl+D.

Výpis čekajících úloh ve frontě zjistíte pomocí ``atq``.

Z výpisu uvidíte také ID úlohy, které můžete použít k odstranění pomocí ``atrm <ID>``.

Cron
====

Pro pravidelné provádění úloh poslouží Cron. Z konfiguračních souborů zvaných crontab zjistí kdy a
jaké příkazy spustit.

.. tip:: Uživatelé GUI můžou zkusit grafický nástroj pro správu cronů
   http://gnome-schedule.sourceforge.net/ (sudo apt-get install gnome-schedule).

crontab
-------

Crontab je jednoduchý textový soubor se seznamem příkazů a výrazem určující "kdy" příkaz provést.

.. rubric:: Editace

Soubor srontab byste nikdy neměli editovat na přímo, ale prostřednictvím ``crontab -e``, který
otevře váš textový editor a po uložení a opuštění editoru zkontroluje případné chyby.

Vyvolání ``crontab -e`` otevře váš uživatelský crontab. Pro editaci roota použijte ``sudo crontab
-e``.

.. tip:: Náš "oblíbený" editor určíte do proměnné prostředí EDITOR. Např. má-li to být nano, pak
   ``export EDITOR=nano``.

.. rubric:: Zobrazení a vymazání

Podobně pro zobrazení můžete použít ``crontab -l``, resp. pro odstranění ``crontab -r``.

.. rubric:: Povolení/omezení uživatelů cronu

V Ubuntu a Debianu standardně můžou všichni uživatelé použít cron. Můžete ale určit uživatele, kteří
smí/nesmí používat crontab editací souborů ``/etc/cron.allow`` a ``cron.deny``. Tyto soubory dokonce
v Ubuntu ani standardně neexistují. Prostudujte dokumentaci, protože toto řešení má řadu "ale".

Viz také ``man 1 crontab``.

cron výrazy
-----------

.. note:: Cron výrazy je důležité alespoň zběžně ovládat. Používají se i v řadě knihoven
   programovacích jazyků. V podstatě je cron výraz de facto standard pro zápis časového plánu.

Na první pohled je syntaxe crontabů zcela nepochopitelná::

    5 * * * * curl http://virtage.com

.. todo:: "přesměrování" odkaz do I.

Každý řádek je rozdělen na plán a příkaz(y) k provedení Bashem. Výstup příkazů se standardně posílá
na mail uživatele daného crontabu, ale často bývá přesměrován do log souboru.

Plán obsahuje povinných 5 elementů, kdy každá hodnota postupně odpovídá

* minutě (0-59)
* hodině (0-23)
* dne měsíce (1-31)
* měsíci (1-12)
* dni týdne jako číslo (0-6, kde 0 je neděle) nebo zkratka anglického názvu (MON, TUE, WED, THU, FRI, SAT, SUN)

.. note:: Pro jistotu uvádíme, že první minuta nového dne je 0:00 a poslední je 23:59. Podobně první
   minuta nové hodiny je 0 a poslední 59.
   
Na místě elementu můžete taky použít

* hvězdičku (*) pokud chcete vyjádřit každou minutu, hodinu, atd.
* čárku (,) pro seznam hodnot
* operátor dělení (/) pro každou n. hodnotu
* operátor rozsah (-) pro rozpětí hodnot

.. role:: raw(raw)
   :format: html latex

.. csv-table:: Příklady cron výrazů
   :header: "Výraz", "Popis"

   ":raw:`* * * * *`", "každou minutu"
   ":raw:`15 * * * *`", "každých 15. minut každé hodiny"
   ":raw:`0 9 * * 1`", "každé pondělí v devět hodin"
   ":raw:`0 8,12,17,22 * * 6,0`", "každou sobotu a neděli v 8, 12, 17 a 22 hodin"
   ":raw:`*/15 * * * *`", "každých 15 minut"
   ":raw:`*/5 * * * 1-5`", "každých 5 minut od pondělí do pátku (1-5)"

Kromě těchto výrazů můžete použít i tzv. "@zkratky" pro časté plány.

.. csv-table:: @Zkratky cron výrazů
   :header: "@Zkratka", "Popis"

   "@reboot", "po startu PC"
   "@yearly nebo @annualy", "ročně (jako :raw:`0 0 1 1 *`)"
   "@monthly", "měsíčně (jako :raw:`0 0 1 * *`)"
   "@weekly", "týdně (jako :raw:`0 0 * * 0`)"
   "@daily nebo @midnight", "denně (jako :raw:`0 0 * * *`)"
   "@hourly", "každou hodinu (jako :raw:`0 * * * *`)"

.. tip:: Na internetu najdete spoustu testerů a rozepisovačů cron výrazů. Např.
   http://cron.schlitt.info nebo http://www.cronchecker.net.

.. tip:: Viz také ``man 5 crontab``.

System-wide a uživatelské crontaby
----------------------------------

Každý uživatel má vlastní crontab. Cron démon provede crontab uživatele bez ohledu na to, jestli je
zrovna přihlášen. System-wide crontab(y) jsou editovatelné jen rootem obsahující úkoly jako rotování
logů, zálohování ap.).

Crontaby uživatelů leží ve ``/var/spool/cron/crontabs/``. Systémové crontaby jsou na několika místech.

.. caution:: Hlavní soubor ``/etc/crontab`` byste nikdy neměli editovat, ale umístit crontaby do
   centrální system-wide složky ``/etc/cron.d/``.

Syntaxe systémových crontabů je téměř stejná jako u uživatelských. Uživatelské crontaby se provádí
pod daným uživatelem. Pro systémové musíte uživatele určit (nejčastěji to bývá root).

Podoba záznamu systémového crontabu navíc proto obsahuje username uživatele::

    <výraz> <uživatel> <příkaz>

např.::

    @daily root apt-get autoremove

Anacron a Debian/Ubuntu vylepšení
---------------------------------

Anacron je modernější implementací tradičního Cronu zvaného Vixie podle svého autora. Jedno
vylepšení je hlavně pro uživatele, kteří nemají počítač stále zapnutý - anacon umí spustit úlohy,
které měli provést, když byl počítač vypnutý.

Debian/Ubuntu nabízí také rozšíření Vixie Cronu pro system-wide úkoly prováděné každou hodinu, den,
týden a měsíc. Stačí umístit skript (nebo odkaz) do příslušné složky v daný čas provedou:

* ``/etc/cron.hourly/``
* ``/etc/cron.daily/``
* ``/etc/cron.weekly/``
* ``/etc/cron.monthly/``