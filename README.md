Gigabit-Ethernet-Chip RTL8211 Verilog-Programm

Dieses Programm besteht aus mehreren Verilog-Dateien und enthält alle relevanten Dateien im hdl-Ordner sowie einige notwendige IP-Cores.
Beschreibung der Dateifunktionen
crc.v:Wird hauptsächlich verwendet, wenn udp gesendet wird, um den CRC-Wert des udp-Pakets zu berechnen.
ethernet.v:Wrapper für die Dateien udp_send.v und udp_receive.v.
ethernet_top.v: Beispiel einer Top-Level-Datei für das gesamte Projekt, die nur zeigt, wie die Datei und andere verwandte ip core-Dateien aufgerufen werden.
udp_receive.v:Verantwortlich für die Empfangsverarbeitung von udp.
udp_send.v:Verantwortlich für die Sendeverarbeitung von udp.
Erklärung der Datei
udp_receive.v
port:

clk :Der Haupttakt des Empfangsmoduls.
rst_n :Das globale Reset-Signal des Empfangsmoduls.
rx_data_len :Die udp-Paketlänge der empfangenen Daten, die der Parameter des udp-Headers ist.
rx_total_len :Die Länge des ip-Pakets für die empfangenen Daten, die der Parameter des ip-Daten-Headers ist.
update :Dieser Wert wird einen Zyklus nach oben gezogen, wenn ein neues udp-Paket empfangen wird.
ip_header :Der gesamte IP-Paket-Header der empfangenen Daten, es werden nur 160-Bit-IP-Datagramm-Header unterstützt.
udp_header :Der gesamte empfangene UDP-Paket-Header, es werden nur 64-Bit-UDP-Header unterstützt.
mac: Das gesamte empfangene MAC-Paket, einschließlich Ziel-MAC, Quell-MAC und IP-Paket-Typ-Kennung.
data_o :Die empfangenen gültigen Daten.
src_mac :Die MAC-Quelladresse des Pakets.
src_addr :Die IP-Quelladresse des Pakets.
src_port :Die Quell-Port-Adresse des Pakets.
DF :Der Parameter des IP-Pakets, ein Wert von 0 bedeutet, dass das Paket slicable ist, ein Wert von 1 bedeutet, dass es nicht slicable ist.
MF :Der Parameter des IP-Pakets, eine 1 bedeutet, dass noch ein Slice vorhanden ist, eine 0 bedeutet, dass dies das letzte Slice ist.
e_rxdv :Das Kontroll-IO des RTL8211, wenn es high ist, bedeutet es, dass die empfangenen Daten gültig sind.
rxd :Empfangsdaten-IO des RTL8211.
Definierte Konstanten:

CAN_RECEIVE_BROADCAST:Ob der Empfang von Broadcast-Paketen unterstützt werden soll, d.h. Pakete mit der MAC-Adresse FFFFFFFFFFFFF.
DST_ADDR: Setzt die IP-Adresse des Ziels, d.h. die NIC-Adresse des RTL8211.
DST_PORT: Stellt die Port-Adresse der Zieladresse ein, die der Port des RTL8211 ist.
DST_MAC: Legt die MAC-Adresse der Zieladresse fest, die die MAC-Adresse des RTL8211 ist.
Status machine state jump Einstellung:

IDLE: Leerlaufzustand, sobald der Präambelcode empfangen wird, geht er in den Zustand R_PRE über.
R_PRE: Empfang des Präambelcodes, Übergang in den R_MAC-Zustand, sobald 7 Präambelcodes und ein Startbit Daten empfangen wurden.
R_MAC:Empfang von MAC-Daten, nach deren Empfang geht er in den Zustand R_HEADER über.
R_HEADER:Empfängt IP-Daten-Header und UDP-Daten-Header und geht nach dem Empfang in den Zustand R_DATA über.
R_DATA:Empfängt gültige UDP-Daten und geht nach Beendigung in den Zustand R_FINISH über.
R_FINISH:Rückkehr zum IDLE-Zustand.
udp_send.v
Ports:

clk,rst_n:Signale wie oben;
data_i:zu sendende Daten, überträgt die Daten, die an das UDP-Paket gesendet werden sollen, über diesen Anschluss, wenn tx_dv high ist.
tx_data_len:Länge der zu sendenden Daten, Parameter des UDP-Pakets.
crc:CRC-Prüfsumme des gesamten Ethernet-Pakets, reserviert für die Datenprüfsumme.
crcen:Freigabesignal des crc-Moduls, für hohe crc-Prüfsumme kann nur das Modul berechnen.
crcrst:Das Reset-Signal des crc-Moduls, für die hohe crc-Prüfsumme, die das Modul zurücksetzen kann.
start:Das Startsignal des UDP-Sendemoduls, damit das High-Driver-Modul mit der Übertragung von Gruppenrahmen beginnen kann.
busy: Das Busy-Signal des UDP-Sendemoduls, das auf High gezogen wird, wenn das Modul gerade Frames sendet.
tx_dv:Das Anzeigesignal des UDP-Sendemoduls, das anzeigt, dass das aktuelle UDP-Modul bereit ist, die Daten zu senden, bitte senden Sie die zu sendenden Daten.
dst_mac:Die MAC-Adresse der Zieladresse.
dst_addr:Die IP-Adresse der Zieladresse.
dst_port:Die Portadresse der Zieladresse.
DF,MF wie oben beschrieben.
tx_en:Das Sendefreigabesignal des RTL8211, gültig für hohe Sendedaten.
txer:Das Sendefehlersignal des RTL8211.
txd:Das Sendedaten-IO des RTL8211.
Definierte Konstanten:

IP_HEADER_LEN:Die Länge des IP-Pakets, Standard ist in Ordnung, muss nicht geändert werden.
TTL: Der Parameter des IP-Pakets, die Zeit, die es überlebt.
SRC_ADDR: IP-Adresse der Quelladresse.
SRC_PORT: Der Port der Quelladresse.
SRC_MAC: MAC-Adresse der Quelladresse.
Einstellungen für das State Machine State Hopping:

IDLE: Leerlaufzustand, Start ist hoch, um in den MAKE_IP-Zustand zu gelangen.
MAKE_IP: Erzeugt IP-Pakete und wechselt in den Zustand MAKE_SUM.
MAKE_SUM:Berechnet die erste Prüfsumme des IP-Pakets und wechselt nach der Berechnung in den Zustand SEND_PRE.
SEND_PRE:sendet 7 führende Codes und 1 Startcode. Dann tritt es in den Zustand SEND_MAC ein.
SEND_MAC:Sendet die Ziel-MAC, die Quell-MAC und den IP-Pakettyp und geht dann in den Zustand SEND_HEADER über.
SEND_HEADER:Sendet den IP-Datenheader und den UDP-Datenheader und geht dann in den SEND_DATA-Zustand über.
SEND_DATA:Sendet gültige Daten und wechselt nach dem Senden in den Zustand SEND_CRC.
SEND_CRC:Senden von CRC-Daten und Eintritt in den Zustand IDLE_CODE nach dem Senden.
IDLE_CODE:Senden von 12 Leerlaufcodes, Ethernet-Paketanforderung. Rückkehr in den IDLE-Zustand nach dem Senden

Um die Timing-Anforderungen von 125 MHz zu erfüllen, wurden die beiden Kernteile des Codes so weit wie möglich geändert. So wird beispielsweise in udp_send die Zustandsmaschine verwendet, um das entsprechende Datenbit entsprechend dem Zählerwert zu senden, wenn der MAC- und der Datenheader gesendet werden. Diese Methode konnte jedoch die Timing-Anforderungen von 125 MHz nicht erfüllen und wurde daher durch die aktuelle Methode ersetzt. Ein weiteres Beispiel ist die Berechnung der Summe der ersten Prüfungen, die in mehrere Schritte unterteilt ist, wodurch die Gesamtzahl der Zeitüberschreitungen effektiv reduziert wird.
Dennoch entsprechen einige der Signale nicht den Timing-Anforderungen, und der Timing-Bericht ergibt einen Timing-Pfad von minus 1ns oder so.

Unerledigte Aufgaben, die darauf warten, ausgefüllt zu werden.
Artikel zum Ausfüllen der Lücke
(Gigabit-Netzwerke erforschen) 1. die gesamte Struktur eines Ethernet-Rahmens verstehen
(Exploring Gigabit Networks) 2. Bemühungen zur Gewährleistung von Zeitspannen für digitale Schaltungen
