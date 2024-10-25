# 3 - Eines de mapatge

Actualment hi ha moltes eines de mapatge en el mercat. Existeixen eines de
mapatge per a la major part de llenguatges orientats a objectes PHP,
Objective-C, C++, SmallTalk i evidentment Java/Kotlin.

Bàsicament són eines en les quals podem distingir tres aspectes conceptuals
clarament diferenciats:

  * Un sistema per expressar el mapatge entre les classes i l’esquema de la Base de Dades,
  * Un llenguatge de consulta orientat a objectes (que en realitat accedirà a les taules) i per tant ens permetrà salvar el desfasament O-R
  * Un nucli funcional que possibilita la sincronització dels objectes persistents de l’aplicació amb la Base de Dades. 

**<u>Tècniques de mapatge</u>**

Entre les tècniques que aquestes eines fan servir per plasmar els mapes O-R,
destaquem:

  * Les que incrusten les definicions dins el codi de les classes
  * Les que guarden les definicions en fitxers independents.

Les primeres solen ser tècniques molt vinculades al llenguatge de programació,
així per exemple, en C++ se solen fer servir macros i en Java/Kotlin
s’utilitzen **anotacions**.

Les tècniques de definició basades en fitxers independents del codi solen
utilitzar el format **XML** , perquè és un llenguatge molt expressiu i
fàcilment extensible.

La major part d’eines accepten les dues tècniques de mapatge anteriors, i fins
i tot permeten la convivència dins d'una mateixa aplicació.

  * Les tècniques que utilitzen el propi llenguatge de programació per incrustar el mapatge dins el codi presenten una corba d’aprenentatge més baixa per a desenvolupadors experimentats en el llenguatge amfitrió, però per contra no serà possible aprofitar les definicions si es decideix canviar de llenguatge de programació.
  * En canvi, fent servir les tècniques basades en XML sí que es poden reutilitzar les definicions per a diferents llenguatges, sempre que l’eina utilitzada estiga disponible en el llenguatge de programació requerit o bé si el format de les definicions segueix alguna especificació estàndard.

A hores d'ara és difícil dir per on s'encaminaran els estàndards. Això sí, la
major part d’eines utilitzen una sintaxi prou similar per a expressar les
definicions del mapatge, encara que òbviament sempre hi ha diferències que no
els fan del tot compatibles.

**<u>Llenguatges de consulta</u>**

El llenguatge de consulta més utilitzat per la major part d’eines és el
llenguatge anomenat **OQL** (**Object Query Language**) o una variant del
mateix. Es tracta d’un llenguatge especificat per l’ODMG. Presenta molta
similitud amb SQL, ja que que els dos són llenguatges d’interrogació no
procedimental, però l’OQL està totalment orientat a objectes. És a dir, en la
consulta fem servir la sintaxi pròpia dels objectes i els resultats obtinguts
retornen objectes o col·leccions d’objectes.

Es tracta del llenguatge d’interrogació que també utilitzen moltes bases de
dades orientades a objecte, la qual cosa el fa un dels estàndards més populars
i coneguts. Hi ha altres llenguatges de consulta orientats a objectes, però no
tan estesos com OQL.

**<u>Tècniques de sincronització</u>**

La sincronització amb la base de dades és segurament un dels aspectes més
crítics de les eines de mapatge. Solen ser processos molt complexos, on trobem
implicades sofisticades tècniques de programació per a poder descobrir els
canvis que van patint els objectes (i així poder guardar-los), a crear i
inicialitzar les noves instàncies que calga posar en joc dins l’aplicació
d’acord amb les dades guardades o també a extreure la informació dels objectes
per revertir-la a les taules del SGBD.

Aquest és probablement l’aspecte que més diferencia les eines entre elles. Les
principals tècniques utilitzades són la precompilació, la postcompilació i la
reflexió. El llenguatge de programació suportat per l’eina influirà moltíssim
en les solucions adoptades per resoldre la sincronització. Per exemple, C++
admet precompilació, però no reflexió, mentre que Java/Kotlin admet
postcompilació i reflexió.

**JDO** (**Java Data Objects**) és un estàndard de persistència per a Java (i
per tant per a Kotlin) desenvolupat per Apache. El projecte inclou, a més de
l’especificació de l’estàndard, el desenvolupament d’un marc de persistència
basat en la postcompilació. Aquesta tècnica consisteix en realitzar dues
compilacions a les classes que requereixen persistència. La primera,
realitzada pel jdk, generarà els _bytecode_ estàndards (és a dir els
"executables" de Java/Kotlin, els que es poden executar en la màquina virtual
Java). La segona compilació, fent servir les indicacions descrites en els
documents de mapatge, modificarà els _bytecode_ generats i hi afegirà nova
funcionalitat necessària per dur a terme la persistència d’una forma
transparent.

**JPA** (**Java Persistence API**) és també un estàndard de persistència. Es
troba incorporat en el JDK des de la versió 5. Hi ha diverses biblioteques que
el suporten, totes elles basades en el principi de la reflexió. Java és un
llenguatge que en compilar les classes incorpora metadades amb informació
pròpia de la classe, com ara l’estructura de dades interna, els mètodes que
tinga disponibles, els paràmetres necessaris per invocar els mètodes, etc. La
màquina virtual permet fer servir aquestes metadades per accedir a la
informació real guardada en els objectes, per invocar algun dels seus mètodes,
per construir noves instàncies, etc.

La postcompilació, usada per JDO, presenta l’avantatge de ser més eficient, ja
que incrusta codi compilat directament allà on cal, mentre que la tècnica de
la reflexió ha d’anar llegint les metadades i interpretant la manera
d’executar adequadament les ordres desitjades.

Malauradament, JDO no ha acabat de quallar i poques iniciatives comercials
l’han seguit. Per contra JPA, és la tècnica que han utilitzat els dos gegants
fortament implantats (Hibernate i EJB). Tot sembla apuntar que serà l’escollit
per cobrir l’estandardització de la persistència en Java/Kotlin.

En aquestos apunts utilitzarem **Hibernate** , que es basa en JPA.



Llicenciat sota la  [Llicència Creative Commons Reconeixement NoComercial
CompartirIgual 2.5](http://creativecommons.org/licenses/by-nc-sa/2.5/)

