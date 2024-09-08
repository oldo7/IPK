# Štruktúra zdrojového kódu
Po spustení aplikácie sa spracujú argumenty. V prípade chybných argumentov program vypíše nápovedu a ukončí program na chybe. Následne sa nezávisle od toho, aký mód je použitý, volá funkcia get adress. Tá slúži na vytvorenie premennej typu sockaddr_in, ktorá je nutná pri volaní funkcie connect(), prípadne sendto/recvfrom. Následne sa program podla toho, aký mód je použitý rozvetví. 

#### TCP:
Ak je použitý mód tcp, volá sa funkcia get_socket_tcp(), a so získaným socketom a predtým získanou adresou sa volá funkcia connect() ktorá ustanoví spojenie a daný socket naviaže k tomuto spojeniu. Nasleduje smyčka while, ktorá v jednom cykle získava vstup od uživatela, posiela ho, získa odpoveď zo servera a vypíše ju na štandardný výstup. V prípade že uživateľ zadá príkaz BYE, program po získaní a vypísaní odpoveďe od servera uzavrie spojenie a ukončí sa.

#### UDP:
V prípade že je použitý mód udp sa získa socket pomocou get_socket_udp(). Nasleduje podobný cyklus ako v prípade tcp: Získanie vstupu, odoslanie serveru, získanie odpovede a vypísanie odpovede. V tomto prípade sa však po získaní vstupného reťazca musia ešte pred daný reťazec pridať položky "Opcode" a "Payload length". Z prijatého reťazca sa musia naopak nadbytočné položky odstrániť. To sa deje vo funkcii print_udp_response(), ktorá skontroluje správnosť hlavičky, a následne podla políčka "Payload length" vypíše daný počet charakterov z políčka "Payload Data".

# Započatie a ukončenie spojenia
#### TCP:
Spojenie sa pokúsi ustáliť ešte pred odoslaním prvej správy. Ak sa to podarí, pri samotnom odoslaní už sa špecifikuje iba socket, keďže spojenie už je naviazané. Ak sa to nepodarí, program skončí na chybe.

Ukončenie spojenia prebieha vo funkcii terminate(). Tá uzavrie používaný socket a ukončí program. Uživateľ môže spojenie ukončiť buď odoslaním príkazu BYE, alebo signálom prerušenia (Ctrl + C). Obidva spôsoby vedú na volanie tej istej funkcie. Pri ukončení pomocou signálu prerušenia sa však volá funkcia signal(), ktorá funkciu terminate zavolá bez možnosti priradenia parametrov. Keďže vo funkcii sa uzatvára socket, ktorý tam však nieje možné dostať cez parametre, premenná client_socket je implementovaná ako globálna premenná.

#### UDP:
Žiadne započatie spojenia sa nekoná, a pri volaní funkcii odoslania a prijatia je vždy nutné uviesť okrem socketu aj adresu. Toto vedie na všeobecne menej spoľahlivé spojenie.

Ukončenie spojenia prebieha rovnako ako pri TCP spojení, t.j. zavolá sa funkcia terminate ktorá uzavrie socket a ukončí program. Rozdielom je že jediný spôsob ako uživateľ môže ukončiť spojenie je pomocou signálu prerušenia.

# Testovanie
Testovanie bolo vykonané na referenčnom stroji poskytnutom v zadaní. Stroj beží na operačnom systéme NexOS a bol spustený pomocou aplikácie VirtualBox. Na danom stroji bol spustený program ipkpd, ktorý slúži ako server implementujúci IPKCP. Na rovnakom stroji bola v druhom termináli preložená a spustená implementácia klienta. Na strane klienta bolo následne otestovaných niekoľko vstupov, a boli skúmané odpovede zo strany servera:

#### TCP:
Príkaz spustenia ipkpd: 
>ipkpd -m tcp

Príkaz spustenia klienta: 
>./ipkcpc -h 0.0.0.0 -p 2023 -m tcp


![](screenshots/tcp1.png?raw=true)
![](screenshots/tcp2.png?raw=true)
![](screenshots/tcp3.png?raw=true)
![](screenshots/tcp4.png?raw=true)
![](screenshots/tcp5.png?raw=true)

#### UDP:
Príkaz spustenia ipkpd: 
>ipkpd -m udp

Príkaz spustenia klienta: 
>./ipkcpc -h 0.0.0.0 -p 2023 -m udp

![](screenshots/udp1.png?raw=true)
![](screenshots/udp2.png?raw=true)


## Dodatočné testovanie:
Program bol tiež otestovaný na operačnom systéme Ubuntu, s pripojením na školský server merlin na porte 10002, na ktorom je implementovaný server s protokolom ipkcp. Pre dané vstupy boli výstupy rovnaké ako na referenčnom stroji.

Funkcionalita programu bola porovnávaná s nástrojom netcat. Jediný nájdený rozdiel bol, že po odoslaní príkazu BYE sa netcat neukončí, narozdiel od ipkcpc. Toto je však zamýšlané chovanie, pretože v protokole je špecifikované že odoslaním príkazu BYE si klient želá ukončenie spojenia, a teda je bezpečné predpokladať že už nechce posielať žiadne ďalšie správy.