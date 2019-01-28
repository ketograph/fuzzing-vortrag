2 SWS, Selbststudium auf Leistungsschein




# Fuzzing - Ein kurzer Überblick
Hauptseminar zum Thema Fuzzing

Fuzzing ist eine Methode aus dem Software- bzw. Hardwaretesting und ist seit den 1980er Jahren bekannt und seitdem immer weiter erforscht und verbessert.
Die Grundidee ist dabei sehr simpel und besteht aus zwei Schritten:
1. ein System mit zufälligen Daten starten
2. einen Systemabsturz und dessen Ursache protokollieren
Diese beiden Schritten werden dann sehr oft ausgeführt (tausend- oder millionenfach).

Durch das simple Prinzip ist eine naive Implementierung sehr einfach und für neue Systeme ohne bestehende Fuzzing-Software lassen sich sehr schnell einfache Fuzzing-Tests schreiben. Da im Gegensatz zu Unit Test bzw. positiven Tests (TODO: Begriff positive Tests checken) nicht die Spezifikation herangenommen wird, werden relativ einfach auch Fehler bei Randfälle und Szenarien ausßerhalb der Spezifikation gefunden. Zusätzlich ist der Wartungsaufwand gering: beim initialen Aufsetzen muss man keine Testfälle spezifieren sondern nur das System. danach werden ohne weiteren manuellen Aufwand die Tests durchlaufen. Auch muss man bei Systemänderungen nicht die Testfallspezifikationen anpassen.
Nachteilig ist die Einschränkung von Fuzzing, dass nur die Robustheit getestet wird, dass heißt ob das Programm abstürzt oder sich aufhängt. Eine Verifikation der Ergebnisse findet nicht statt. Außerdem ist der Rechenaufwand hoch, so muss man bei komplexeren Systemen, die sich nicht isolieren lassen und große Eingabedaten benötigen sehr viele Tests ausführen und entweder tage- bzw. wochenlang fuzzen oder statt einem Rechner dutzende oder hunderte verwenden.


## Techniken
Grundsätzlich lassen sich mehrere Aspekte unterscheiden: zum einen welches System gefuzzt wird und zum anderen wie die Daten generiert werden.
In diesem Zusammenhang kann System für mehrere Sachen stehen: zum einen für Programme, dann Betriebssysteme (als Unterform von Programmen) sowie Hardware. 
Die Datengenerierung kann durch mehrere Methoden geschehen: zufälliges Fuzzing, mutationsbasiertes Fuzzing, regelbasiertes Fuzzing sowie instrumentiertes Fuzzing.
Wie die generierten Daten an das gefuzzte System kommen ist lediglich eine Frage, welche Schnittstellen das System zur Verfügung stellt. Bei  Programmen können dies beispielsweise Dateien,
`stdin` oder Netzwerkschnittstellen (beispielsweise bei Server) sein. Betriebssysteme werden oft über die spezifischen Schnittstellen wie `syscalls` oder `ioctl` angesprochen oder emulierte Hardwaregeräte
wie USB-Geräte senden Daten an das Betriebssystem und dessen Treiber. Hardware kann zum einen über das ändern der elektrischen Signale gefuzzt werden oder bei Prozessoren über die Maschinenbefehle. 
Ich lege den Fokus im Weiteren auf Software und gehe dabei auf ein paar Besonderheiten dabei ein. Die grundsätzlichen Methoden lassen sich dabei aber genauso gut auf Hardwaresysteme übertragen.



### Zufälliges Fuzzing  (random fuzzing)
Bei zufälligen Fuzzing werden, wie schon der Name andeutet, rein zufällige Daten an das System geschickt. Diese Methode ist sehr einfach und simpel zu implementieren (in Linux beispielsweise `cat /dev/urandom |  programm_to_fuzz`), aber dafür nicht sehr effizient. So reichen schon einfachste Validierungen der Eingangsdaten, um diese als fehlerhaft zu erkennen. Da das System in diesem Fall die Verarbeitung beendet, werden nur kleine Teile des Programms getestet.

### Mutationsbasiertes Fuzzing (mutation based fuzzing)
Eine deutlich effizientere Testmöglichkeit bietet das mutationsbasierte Fuzzing (engl. mutation-based fuzzing). Dabei wir zuerst eine Sammlung von Testdaten angelegt, Testkorpus genannt. Diese werden dann zufällig verändert (mutiert) und in das Programm gespeist. Diese Methode ist sehr einfach zu implementieren und effizient im Fehler finden. Da noch ein großer Teil der Daten valide ist, im Gegensatz zum zufälligen Fuzzing, werden die Daten angefangen zu verarbeiten. Simple Validierungen und Datenüberprüfungen erkennen die Daten als korrekt an.
Ein Nachteil dieser Methode ist die große Abhängigkeit von der Testdatenauswahl. Für eine möglichst hohe Testabdeckung müssen viele Programmzweige durchlaufen werden. Dies geschieht aber nur, falls die Testdaten geeignet dafür sind und möglichst viele Fälle abdecken. Beispielsweise sollte man um einen PDF-Reader zu testen nicht nur simple PDF-Dateien mit Text verwenden, sondern es sollten auch Bilder, Videos und komplizierte Layouts verwendet werden. 

* Tools: [`zzuf`](http://caca.zoy.org/wiki/zzuf), [`Radamsa`](https://gitlab.com/akihe/radamsa)
 
### Regelbasiertes Fuzzing (generation based fuzzing)
Eine ähnliche Methode ist das das regelbasierte Fuzzing (engl. generation based fuzzing). Dabei wird auf Grundlage dieser Spezifikation eine Beschreibung der Daten generiert, beispielsweise eine Grammatik oder ein Protokoll, und daraus werden Sequenzen generiert. Diese generierten Sequenzen lassen sich dann wieder zufällig mutieren. Zusätzlich lassen sich aber auch komplette Nachrichten der Kommunikation wiederholen oder weglassen. Dies hat den Vorteil einer einfachen Entdeckung auch komplexer Logik- und Protokollfehler, wenn beispielsweise die interne Zustandsautomat fehlerhafte Übergange hat.
Durch die Notwendigkeit der Beschreibung ist der anfängliche Aufwand sehr hoch, diese Beschreibung muss zuerst erstellt werden. Außerdem müssen überhaupt die Daten wohlgeformt sein und einer Spezifikation genügen. Zusätzlich ist dann der Erfolg des Fuzzing stark abhängig von der Güte der Beschreibung, ob diese alle Aspekte modeliert oder nicht.

* Tools: [`Peach Fuzzer`](https://www.peach.tech/products/peach-fuzzer/), [`Dharma`](https://github.com/MozillaSecurity/dharma/)
  
### Instrumentiertes Fuzzing (coverage guided fuzzing)
Die neuste Entwicklung ist das instrumentierte Fuzzing (engl. coverage guided fuzzing). Dabei wird für alle Eingangsdaten die Codeausführung beobachten und mit diesem Feedback die Eingangsdaten verändert, sodass möglichst viele Codezweige erreicht werden. Wenn beispielsweise durch Mutation eines bestimmten Bytes keinen neuen  Codezweige erreicht werden, wird die Mutation abgebrochen und durch Mutation anderer Bytes versucht, einen neuen Codezweig zu erreichen. Typischerweise ist auch hier Ausgangspunkt ein Korpus an Testdaten, die mutiert werden.
Diese Methode setzt natürlich voraus, dass man die Codeausführung beobachten kann. Wenn der Quellcode verfügbar ist, wird dies meist durch spezielle eingefügte Instruktionen erreicht. Natürlich ist auch die Verwendung eines Debuggers möglich. Der generelle Vorteil ist eine hohe Effizienz, da nur Bytes verändert werden, die einen Einfluss auf den Programmablauf haben. Dadurch wird außderdem eine hohe Code-Abdeckung erreicht. Nachteil ist eine geringere Performance, entweder durch die zusätzlichen Instruktionen oder durch den Debugger. Dieser Performancenachteil darf dabei nicht größer werden als die verbesserte Effizienz, da sonst die Gesamtzahl an gefunden Fehlern im Programm sinkt.
Ein weiterer Vorteil ist die Möglichkeit der Testkorpus- und Testfallminimierung. Bei der Minimierung des Testkorpus werden alle Testdaten zusammengefasst, bei deren Verarbeitung die gleichen Codezweige durchlaufen werden. Die Testfallminimierung verringert die Größe der einzelnen Testdaten, sodass eine möglichst kleiner Datensatz erzeugt wird, bei dessen Verarbeitung trotzdem diesselben Codepfade durchlaufen werden wie beim ursprünglichen, großen Datensatz. Beide Methoden führen zu einer höheren Performance und Effizienz bei den nächsten Fuzzing-Tests.

* Tools: `AFL`, `libFuzzer`, `honggfuzz`


Die Mächtigkeit des instrumentierten Fuzzing zeigt sich an einem Beispiel. Für das Fuzzing einer Bildbibliothek wurde als einziger Testfall eine Textdatei mit dein Inhalt `hello` genommen. Wenn dann das Fuzzing begonnen wird, werden nach und nach die Bytes so verändert, dass neue Codezweige erreicht werden und es nach und nach die Strukturen des Bildformates erhält. Nach einiger Zeit des Fuzzens wird dann sogar ein komplettes valides Bild erzeugt.
[![Erzeugung valider Bilder beim instrumentierten Fuzzing einer Bildbibliothek](https://lh6.googleusercontent.com/proxy/-6MjaR00hYA40HOvCaSW4PF_TvPpqAjNZIwGadsPVaYE9hRrGNTi91BBKlVdXtK4X7E5qf9hgk6kHMrxWaE-WaCckCsgZzA=s0-d "Erzeugung valider Bilder beim instrumentierten Fuzzing einer Bildbibliothek")](https://lcamtuf.blogspot.com/2014/11/pulling-jpegs-out-of-thin-air.html)



### Datenmutation
Das vollständige Verändern aller Bytes mit allen Möglichkeiten ist nicht machbar. So gibt es bei einem 32-Bit Wert 2^32 Möglichkeiten, was viele Tage dauern würde zu fuzzen. Aus diesem Grund muss man den Suchraum einschränken. So werden die Bits nur geflippt oder geshifftet und bei Zahlen Werte dazu addiert und subtrahiert. Außerdem kann man die Zahlen und Zeichenketten durch interesante Werte ersetzen, beispielsweise 0, 1, -1, die maximale und minimale Zahl des Zahlenbereichs, bei Gleitkommazahlen die speziellen Werte `NaN` oder `-Inf` und um Speicherfehler zu erkennen typische Buffergrößen wie 16, 32 oder 128. Interessante Zeichenketten sind beispielsweise komplett leere oder sehr lange Zeichenketten (`''`, `128*'a'`), spezielle Bytes und Zeichen wie `\0` als Ende von Zeichenketten in C oder `\n` als Zeilenumbruch oder Formatierungsbefehle wie `s%s%s%s` oder `%x %x %x` für die Funktion `printf`, die Format String-Angriffe ermöglichen.

Demonstration zufälliger Bit-Flips und Byte-Ersetzungen mittels `zzuf`:

| Ausgangstext   | mutierter Text    | 
| -------------- | ----------------- |
|![](https://github.com/ketograph/fuzzing-vortrag/blob/master/images/zzuf1.png "Unveränderter Text") | ![](https://github.com/ketograph/fuzzing-vortrag/blob/master/images/zzuf2.png "Manipulierter Text") |
 
Datenmanipulation mittels `radamsa`

![](https://github.com/ketograph/fuzzing-vortrag/blob/master/images/radamsa.png "Manipulierter Text")



### Sanitizer
Bestimmte Fehler in Programmen führen nicht oder nicht sofort zu einem Absturz. So kann ein Speicherfehler durch eine fehlerhaften Schreibvorgang erst nach vielen anderen Befehlen zu einem Absturz führen. Oder es wird nur ein sehr kleiner Teil des Speichers beschädigt, sodass nur unter sehr speziellen Umständen das Programm abstürzt. 
Eine Lösung für dieses Problem ist die Verwendung eines Sanitizers. Dieser fügt beim Kompilieren zusätzliche Instruktionen ein, um die Befehle zur Laufzeit zu prüfen. Beispielsweise kann vor jedem Speicherzugriff eine Überprüfung erfolgen, ob tatsächlich auf diesen Speicher zugegriffen werden darf. Ein Nachteil ist die verringerte Performance und der erhöhte Speicherbedarf, die sich beide um den Faktor zwei oder noch stärker verschlechtern können. Da aber beim Fuzzing deutlich mehr Fehler gefunden werden können, ist es trotzdem sinnvoll Sanitizer zu verwenden.
Es wurden verschiedene Sanitizer entwickelt, diese sind aber teilweise nicht für alle Kompiler verfügbar. In der folgenden Tabelle ist eine Übersicht über die verschiedenen Sanitizer, deren erkannten Fehlerklassen und welche Kompiler diese unterstützen. 

| Sanitizer   | Fehlerklassen    | Kompiler    |
| ----------- | ---------------- | ----------- |
| AddressSanitizer (ASan) | Out-of-bounds accesses, Use-after-free, Use-after-return, Use-after-scope, Double-free, invalid free | gcc, clang |
| UndefinedBehaviorSanitizer (UBSan)| Using misaligned or null pointer, igned integer overflow, overflowing conversions with floating-point types | gcc, clang |
| MemorySanitizer | Lesen von uninitialisiertem Speicher | clang |
| LeakSanitizer | Memory leaks | gcc, clang |
| ThreadSanitizer (TSan) | Data races | gcc, clang |
    
  
#### Demonstration Address Sanitizers:
Um die Funktion von Address Sanitizern zu zeigen, ist hier folgendes Beispielprogramm gegeben. Es liest eine Zeichenkette aus `stdin` und kopiert diese in einen 16 Byte großen Buffer. Da keine Überprüfung der Länge der Eingabe erfolgt, werden auch Zeichenketten länger als 16 Bytes in den Buffer kopiert, sodass es zu einem Buffer Overflow kommt. 

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
void func(char **argv) {
  printf("running strcpy...\n");
  char arr[16];
  strcpy(arr, argv[1]);
}
int main(int argc, char *argv[]) {
  if(argc == 2) {
    func(argv);
  }
  return(0);
}
```

Wenn dieses Programm kompiliert und ausgeführt wird, kommt es bei einem Buffer Overflow nicht sofort zum Absturz. Beim Kopieren der Eingabedaten werden einfach die Daten überschrieben. Erst bei einer deutliche längeren Zeichenkette kommt es zu einer Beschädigung des Speichers sodass das Programm abstürzt.

```console
>gcc -o buffer_overflow buffer_overflow.c
>./buffer_overflow aaaaaaaaaaaaaaa # 15*a
running strcpy...
>./buffer_overflow aaaaaaaaaaaaaaaa # 16*a, buffer overflowing
running strcpy...
>./buffer_overflow aaaaaaaaaaaaaaaaaaaaaaaaa # 25*a, buffer overflowing
running strcpy...
*** stack smashing detected ***: <unknown> terminated
```

Sobald aber das Programm mit dem Address Sanitizer kompiliert wird, wird sofort bei einem Überschreiten der zulässigen Zeichenlänge ein Fehler ausgegeben und die Programmausführung stoppt.

```console
>gcc -o buffer_overflow -fsanitize=address buffer_overflow.c
>./buffer_overflow aaaaaaaaaaaaaaa # 15*a
running strcpy...
>./buffer_overflow aaaaaaaaaaaaaaaa # 16*a, buffer overflowing
running strcpy...
=================================================================
==24698==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7fff58d871a0 at pc 0x7face647d741 bp 0x7fff58d87150 sp 0x7fff58d868f8
```


### Tipps und Ratschläge
In der Praxis muss immerzwischen Fuzzing-Geschwindigkeit und Effizienz abgewogen werden. Zum Ziel einer möglichst großen Anzahl an gefunden Fehler muss das Programm möglichst oft mit unterschiedlichen Daten ausgeführt werden. Gleichzeitig müssen die unterschiedlichen Daten möglichst oft zu einem fehlerhaften Programmverhalten führen. So ist unter Umständen ein dummer, aber dafür schenller Fuzzer besser als ein sehr intelligenter, dafür aber langsamer Fuzzer. 
Desweiteren sollte man immer Sanitizer verwenden, falls der Quellcode zur Verfügung steht. Dadurch können deutlich mehr Fehler gefunden werden. So wurde beispielsweise beim Kompililieren und darauffolgenden Benutzen eines kompletten Linux-Systems (Gentoo) schon Fehler in fast allen verwendeten Programmen gefunden. [SHA2017Böck](https://media.ccc.de/v/SHA2017-148-improving_security_with_fuzzing_and_sanitizers#t=593)
* Teilweise Codeanpassungen nötig: beispielsweise Deaktivierung von Checksummen oder kryptografischen Signaturen



## Tools
### AFL und libFuzzer
Die beiden bekanntesten Fuzzing-Tools sind aktuell der von Michal Zalewski entwickelte Fuzzer american fuzzy lob, kurz AFL, sowie der von der LLVM-Community gepflegte libFuzzer. Beide erreichen durch das instrumentieren des Codes eine hohe Effizienz und hohe Code-Abdeckung. Mit beiden wurden hunderte Fehler gefunden und die Heartbleed getaufte Lücke in der OpenSSL-Bibliothek hätte durch Fuzzing in Verbindung mit Sanitizern gefunden werden können. 
AFL und libFuzzer unterscheiden sich in mehreren Details: AFL ermöglicht einen sehr schnellen und simplen Einstieg, nach wenigen Minuten kann man einen ersten Fuzzing-Durchlauf starten. So wird der Testkorpus ausgelesen und über `stdin` oder der Dateiname direkt an das Programm übergeben. Man kann auf unterschiedlichen Wegen ein Feedback über die Programmausführung erhalten: entweder durch Neu-Kompilieren und dem Einfügen von Instrumentationsbefehlen oder durch das Versehen von Binaries zur Laufzeit mit diesen Befehlen. Standardmäßig wird für jeden einzelnen Programmdurchlauf ein neuer Prozess gestartet, durch Anpassungen des Codes ist aber auch In-Prozess-Fuzzing möglich.

Zum initilen Aufsetzen von libFuzzer benötigt man etwas mehr Zeit, da man zuerst ein kleines Helferprogramm schreiben und kompilieren muss, welche die generierten Daten des Fuzzers nimmt und das Programm damit aufruft. Die Kompilation muss dabei mit dem Kompiler `clang` erfolgen. Da standardmäßig In-Prozess-Fuzzing ausgeführt wird, ist die Fuzzinggeschwindigkeit höher im Vergleich zu AFL.


#### [Quickstart AFL](http://lcamtuf.coredump.cx/afl/QuickStartGuide.txt)
* Kompilierung:  `CC=afl-gcc ./configure --disable-shared` oder `afl-gcc -static -o programm_to_fuzz programm_to_fuzz.c`
* Start: `afl-fuzz -i testcorpus_directory -o crashing_files_directory ./programm_to_fuzz` 

#### [Quickstart libFuzzer](http://llvm.org/docs/LibFuzzer.html#getting-started)
* Implementierung des Helferprogramms: [![Code zum Erstellen eines Fuzzers mittels libFuzzer](https://github.com/ketograph/fuzzing-vortrag/blob/master/images/libfuzz-quickstart.png "Fuzzing Ziel erstellen")](http://llvm.org/docs/LibFuzzer.html#id22) 
* Kompilierung: `clang -fsanitize=fuzzer -o my_fuzzer fuzz_target.cc`   
* Ausführung ohne Testkorpus: `./my_fuzzer`, mit Testkorpus: `./my_fuzzer -i testcorpus_directory`  
  


### Kernel Fuzzer
* [`syzkaller`](https://githu.com/google/syzkaller)
  * Entwicklung durch Google
  * Instrumentiertes Fuzzing
  * Start des Systems in VM und Fuzzing der Syscalls
* Alternative: [`trinity` ](https://github.com/kernelslacker/trinity)
  
### Weitere Tools
* [`Sandsifter`](https://github.com/rigred/sandsifter)
  * Fuzzing von CPU-Instruktionen
  * Entdeckung von Bugs in Disassemblern, Emulatoren, Hypervisorn sowie x86-Chips 
* [`ClusterFuzz`](https://github.com/google/oss-fuzz/blob/master/docs/clusterfuzz.md) und [OSS-Fuzz-Projekt](https://github.com/google/oss-fuzz)
  * Google-Entwicklung zum Fuzzing von Chrome
  * Verteiltes Fuzzing auf hunderten Kernen
  * OSS-Fuzz: Fuzzing verbreiteter  Open-Source-Software


# Literatur
* Takanen, Ari/DeMott, Jared/Miller, Charlie: Fuzzing for software security testing and quality assurance, Artech House, 2008, Boston
* H. Liang, X. Pei, X. Jia, W. Shen and J. Zhang, "Fuzzing: State of the Art," in IEEE Transactions on Reliability, vol. 67, no. 3, pp. 1199-1218, Sept. 2018. URL: http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=8371326&isnumber=8452065
*  [B.P. Miller, L. Fredriksen, and B. So, "An Empirical Study of the Reliability of UNIX Utilities", Communications of the ACM 33, 12 (December 1990). Also appears (in German translation) as "Fatale Fehlertractigkeit: Eine Empirische Studie zur Zuverlassigkeit von UNIX-Utilities", iX, March 1991.](ftp://ftp.cs.wisc.edu/paradyn/technical_papers/fuzz.pdf) 
* [H. Böck: Improving security with Fuzzing and Sanitizers -  Free and open source software has far too many security critical bugs. SHA Konferenz 2017](https://media.ccc.de/v/SHA2017-148-improving_security_with_fuzzing_and_sanitizers)
* [Jack Random: Fuzzing. Cryptocon 2015](https://media.ccc.de/v/CC15_-_20_-__-_lounge_-_201505101900_-_fuzzing_-_jack_random)
* [Ben Nagy: Windows Kernel Fuzzing For Beginners. Ruxcon 2012](https://www.youtube.com/watch?v=FY-33TUKlqY)
* https://www.owasp.org/index.php/Fuzzing
* https://www.owasp.org/index.php/OWASP_Testing_Guide_Appendix_C:_Fuzz_Vectors
* https://www.mwrinfosecurity.com/our-thinking/15-minute-guide-to-fuzzing/
* https://lcamtuf.blogspot.com/2014/08/binary-fuzzing-strategies-what-works.html
