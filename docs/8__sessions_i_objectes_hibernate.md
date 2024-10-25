# 8 - Sessions i objectes hibernate

En aquesta secció comentarem el comportament de les sessions i dels objectes
Hibernate (les classes creades per Hibernate).

Recordem que el procés per a arribar a una sessió era **Configuration - >
SessionFactory -> Session**. Les sentències serien aquestes:
```
val cfg = Configuration().configure()
val sf = cfg.buildSessionFactory()
val sessio = sf.openSession()
```
La primera sentència carrega el fitxer de configuració **hibernate.cfg.xml** ,
i inicialitza l'entorn d'Hibernate. A partir d'aquest moment podem utilitzar
l'objecte Configuration per crear el **SessionFactory**.

## 8.1 - Transaccions

El primer que hauríem de fer notar és que si el SGBD admet transaccions, les
possibles modificacions que fem sobre una Base de Dades a través d'Hibernate
no es guardaran si no posem en marxa una transacció. Per tant, si volem fer
actualitzacions (insercions, esborrats i/o modificacions), el primer que
haurem de fer en la sessió serà començar una transacció. Al final podrem
confirmar amb **commit** o rebutjar totes les actualitzacions amb
**rollback**. En el següent codi inserim una nova comarca. Utilitzem el mètode
**save** de la sessió, però ja el veurem més avant. Ara només volem il·lustrar
una transacció. No farem aquest exemple, ja que tots ens estem connectant com
el mateix usuari a la mateixa Base de Dades, aleshores només el primer ho
podria fer, i a la reste ens donaria error perquè ja existiria la fila
corresponent a aquesta nova comarca (restricció de clau principal)

    
    
    1	val sessio = Configuration().configure().buildSessionFactory().openSession()
    2	val t = sessio.beginTransaction()
    3
    4	val com = Comarca()
    5	com.nomC = "Columbretes"
    6	com.provincia = "Castelló"
    7	sessio.save(com)
    8
    9	t.commit()
    10	sessio.close()
    

En cas de no haver posat les dues sentències de les línies 2 i 9, no hauria
hagut cap modificació. El mètode **beginTransaction()** ha fet començar la
transacció, i el mètode **commit()** l'ha confirmada definitivament. Si en
aquest moment vulguérem començar una altra transacció seria suficient amb
**t.begin()**.

Per a no haver d'estar sempre pendents de transaccions, es podria posar en
mode **autocommit** , és a dir, que automàticament faça un commit després de
cada actualització (després de cada **save()** , **delete()** o **update()**
). Seria canviar en el fitxer **hibernate.cfg.xml** , posant la següent
propietat entre les del **< session-factory>**:
```
<property name="hibernate.connection.autocommit">true</property>
```
Tanmateix no és tan fiable, i us recomane que utilitzeu transaccions, és a
dir, tenir el autocommit a false com estava per defecte i gestionar les
transaccions manualment.

## 8.2 - Estats d'un objecte Hibernate

Els objectes de les classes que Hibernate ens ha ajudat a crear poden estar en
més d'un estat, si mirem la seua sincronització amb la Base de Dades:

  * **Transitori** : quan l'objecte s'ha creat, però no s'ha associat a cap sessió. És a dir, ja s'ha creat amb **la creació de l'objecte** i potser estiga buit o no (ja se li ha posat algun valor a alguna propietat), però encara no s'ha associat a cap sessió. Es podrà associar per exemple amb **save()** per a guardar l'objecte com hem vist en l'exemple de l'apartat anterior, o per exemple amb **load()** , com havíem vist en exemples anteriors per a agafar des de la Base de Dades.
  * **Persistent** : quan l'objecte ja s'ha associat amb una sessió i per tant té una representació en la Base de Dades i un valor identificador. Es pot haver guardat o carregat. Hibernate detectarà qualsevol canvi fet sobre un objecte persistent i sincronitzarà l'estat quan es complete la unitat de treball. I per a això ja s'encarregarà ell de fer els INSERT, DELETE o UPDATE en la Base de Dades. Nosaltres no ens haurem de preocupar.
  * **Separat** : és un objecte persistent, però que la seua sessió ha finalitzat. Es pot utilitzar, i fins i tot canviar l'estat, però no tindrà efecte aquestos canvies la Base de Dades, a no ser que es torne a associar una altra vegada amb una sessió, és a dir, si es torna a fer persistent. De fet, és una tècnica de treball, utilitzar objectes separats quan l'usuari puga estar molt de temps sense interactuar, i quan torne a interactuar, tornar a fer-lo persistent.

Anem a veure un exemple on es vegen els tres tipus d'estat. Tampoc cal que el
feu vosaltres, està únicament a nivell il·lustratiu:

    
    
    	val sessio = Configuration().configure().buildSessionFactory().openSession()
    	val t = sessio.beginTransaction()
    
    	val com = Comarca()				// A partir d'ací l'objecte és transitori
    	com.nomC = "Columbretes"
    	com.provincia = "Castelló"
    	sessio.save(com)				// A partir d'ací és persistent
    
    	t.commit()
    	sessio.close()					// A partir d'ací és separat, igual es pot utilitzar,
                                        // però no estarà sincronitzat
    	
    	println(com.nomC + " (" + com.provincia + ")")

## 8.3 - Càrrega d'objectes: mètodes load() i get()

La càrrega d'objectes s'aconsegueix amb els mètodes **load()** i **get()**.
Són similars excepte en una qüestió: el primer dóna un error si no es troba la
fila, mentre que el segon contindrà null. A més a partir d'Hibernate 6.0
**load** està **deprecated** , així que millor uilitzar **get** .**  
**

Tant si utilitzem un com l'altre, haurem d'especificar 2 paràmetres: la
**classe** que volem buscar, i el **valor** de la clau principal de qui volem
trobar. **Aquest valor s'ha de passar amb el mateix tipus que la propietat
corresponent a la clau principal** , i en moltes ocasions haurem de canviar de
tipus (**cast**). En el següent exemple, molt paregut al ja utilitzat
anteriorment en **SegonAcces** , trobem la comarca **Alt Maestrat** (valor de
**nom_c** , que correspon a la clau principal). En aquesta ocasió no cal
canviar de tipus, ja que és string:

    
    
    1	val sessio = Configuration().configure().buildSessionFactory().openSession()
    2	
    3	val com = sessio.get("classes.Comarca","Alt Maestrat") as Comarca	// càrrega amb  mètode load()
    4	
    5	println("Comarca " + com.nomC + ": " + com.provincia)
    6	sessio.close()

Com comentàvem, si no existira la comarca Alt Maestrat, donaria un error. Ací
tenim el mateix exemple, però utilitzant el mètode **get()** i comprovant
després si **com** és null, i així esquivem l'error:

    
    
    1	val sessio = Configuration().configure().buildSessionFactory().openSession()
    2	
    3	val com = sessio.get("classes.Comarca","Al Maestrat") as Comarca?	// càrrega amb mètode get()
    4	
    5	 if (com==null)
    6           println("No existeix la comarca");
    7        else {
    8            println("Comarca " + com.nomC + ": " + com.provincia)
    9	 }
    10	sessio.close()
    

Només hem tingut la dificultat d'haver de posar **Comarca?** per si és null el
resultat de la consulta (justament el que volem comprovar)

Anem a mirar la potencialitat que ens proporciona la manera que té Hibernate
de construir les classes, amb el conjunt (o col·lecció) de poblacions que té
un objecte de la classe Comarca.

Els **Set** es poden recórrer de més d'una manera. Una molt estesa és
utilitzar un **iterator** , utilitzant els mètodes **hasNext()** que ens diu
si hi ha més elements en la col·lecció, i **next()** que torna el següent
element de la col·lecció. En el següent programa anem a visualitzar informació
de la comarca de l'**Alcalatén** , i també els seus pobles. Estan marcades les
línies de la utilització del **Iterator**. Copieu-lo en un fitxer Kotlin
anomenat **Exemple_03_TercerAcces.kt**:

    
    
    package exemples
    
    import classes.Comarca
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
        val com = sessio.get("classes.Comarca", "Alcalatén") as Comarca
        print("La comarca " + com.nomC)
        print(" (província de " + com.provincia + ") ")
        println("té " + com.poblacions.size + " pobles")
        println()
        println("Llista de pobles")
        println("-----------------")
    
        val it = com.poblacions.iterator()!!
    
        while (it.hasNext()) {
            val p = it.next()
            System.out.println(p.nom + " (" + p.poblacio + " habitants)")
        }
    
        sessio.close()
    }

Però més senzilla encara és la utilització del bucle **foreach** , que fa el
recorregut ell sol, tenint disponible cada element. La sintaxi en Kotlin pot
ser de dues maneres. Posem com queda l'exemple anterior de cadascuna de les
dues maneres

  * **for (****e in** _**conjunt**_**)** i **e** anirà agafant tots els valors de la col·lecció.

    
    
    for (p in com.poblacions!!) {
    		println(p.nom + " (" + p.poblacio + " habitants)")
    }

  * **_conjunt._ forEach()** i dins dels parèntesis utilitzem cada element com **it**

    
    
    	com.poblacions!!.forEach { 
    		println(it.nom + " (" + it.poblacio + " habitants)")
    	}
    

Utilitzarem la primera forma, que és la més còmoda, i quedarà de la següent
manera. Guardeu el següent programa en fitxer Kotlin anomenat
**Exemple_04_QuartAcces.kt**:

    
    
    package exemples
    
    import classes.Comarca
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
        val com = sessio.get("classes.Comarca", "Alcalatén") as Comarca
        print("La comarca " + com.nomC)
        print(" (província de " + com.provincia + ") ")
        println("té " + com.poblacions.size + " pobles")
        println()
        println("Llista de pobles")
        println("-----------------")
    
        for (p in com.poblacions) {
            println(p.nom + " (" + p.poblacio + " habitants)")
        }
    
        sessio.close()
    }

En ambdós, tant en **TercerAcces** com en **QuartAcces** casos el resultat
haurà estat:
```
La comarca Alcalatén (província de Castelló) té 9 pobles

Llista de pobles  
-----------------  
Llucena (1417 habitants)  
Useres, les (992 habitants)  
Figueroles (549 habitants)  
Costur (562 habitants)  
Xodos (126 habitants)  
Alcora, l' (10672 habitants)  
Atzeneta del Maestrat (1321 habitants)  
Vistabella del Maestrat (384 habitants)  
Benafigos (156 habitants)
```
Com veieu apareixen tots els pobles de la comarca que hem llegit, però com és
un conjunt (Set) no podem assegurar l'ordre. De fet, podria ser que a cadascú
de nosaltres li apareguen els pobles en ordre distint.

Estaria bé que aparegueren ordenats. Sempre ho podríem fer per mig d'una
consulta i ordenar pel nom de la població com veurem més avant. Però si ens
interessa que els pobles apareguen sempre ordenats, podem fer una altra cosa:
modificant el fitxer de mapatge, li podem dir que ens apareguen sempre
ordenats per un camp. Ho havíem comentat en la pregunta 7, i ara anem a
aplicar-lo.

Per mig d'una senzilla indicació en el fitxer de mapatge **Comarca.hbm.xml** ,
podem fer que les dades ens vinguen ordenades, és a dir, que les poblacions
d'una comarca ens vinguen per ordre alfabètic. Senzillament serà posar
**order-by="nom"** en la definició del **set****poblacions**.

És a dir hauríem de substituir la línia
```
<set name="poblacions" inverse="true">
```
per
```
<set name="poblacions" inverse="true" order-by="nom">
```
I així li estem dient que les poblacions vinguen ordenades per nom, que és el
camp de la taula **POBLACIO** pel qual ens interessa ordenar.

Només fent aquesta modificació, el resultat tant del programa **TercerAcces**
com el de **QuartAccess** serà:
```
La comarca Alcalatén (província de Castelló) té 9 pobles  
  
Llista de pobles  
-----------------  
Alcora, l' (10672 habitants)  
Atzeneta del Maestrat (1321 habitants)  
Benafigos (156 habitants)  
Costur (562 habitants)  
Figueroles (549 habitants)  
Llucena (1417 habitants)  
Useres, les (992 habitants)  
Vistabella del Maestrat (384 habitants)  
Xodos (126 habitants)
```

## 8.4 - Inserció, modificació i esborrat d'objectes

En totes aquestes operacions d'actualització, recordem que farà falta posar-
les en una **transacció** per a que tinguen efecte.

**<u>Inserció</u>**


<div style="background-color: #d6eaf8; color: black; padding: 5px;">
<b>Nota</b><br>
 A partir de la versió 6.0 d'Hibernate, <b>save()</b> està deprecated, en favor de
<b>persist()</b>
</div><br>

Es fa amb els mètodes **save()** o **persist()** de la sessió, passant-li
l'objecte que volem guardar. Posem el mateix exemple de l'apartat 8.1, el
d'inserir una comarca. Copieu l'exemple en un fitxer anomenat
**Exemple_11_Insercio.kt** :

    
    
    package exemples
    
    import classes.Comarca
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
        val t = sessio.beginTransaction ()
        val com = Comarca()
    
        com.nomC = "Columbretes"
        com.provincia = "Castelló"
    
        sessio.persist(com)
    
        t.commit()
        sessio.close()
    }

Podem comprovar en la taula com apareix la comarca que acabem d'introduir:

![](T5_8_4_1.png)

I evidentment ho podríem comprovar des de **DBeaver**

**<u>Esborrat</u>**


<div style="background-color: #d6eaf8; color: black; padding: 5px;">
<b>Nota</b><br>
A partir de la versió 6.0 d'Hibernate, <b>delete()</b> està deprecated, en favor
de <b>remove()</b>
</div><p></p>

Amb els mètodes **delete** o **remove** , passant-li com a paràmetre
l'objecte. Prèviament s'ha d'haver carregar l'objecte amb els mètodes
**load()** o **get()**. Copieu el següent exemple en el fitxer
**Exemple_12_Esborrat.kt** :

    
    
    package exemples
    
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
        val t = sessio.beginTransaction()
        val com = sessio.get("classes.Comarca", "Columbretes")
    
        sessio.remove(com)
    
        t.commit()
        sessio.close()
    }

**<u>Modificació</u>**


<div style="background-color: #d6eaf8; color: black; padding: 5px;">
<b>Nota</b><br>
A partir de la versió 6.0 d'Hibernate, <b>update</b> està deprecated, en favor
de <b>merge()</b>
</div><p></p>

Amb el mètode **update()** o **merge()** , passant-li com a paràmetre
l'objecte. Prèviament haurà d'haver estat carregat amb els mètodes **load()**
o **get()** , igual que en el cas d'esborrar.

En el següent exemple estem canviant la província del Camp de Morvedre. Per a
que no tinga efecte aquest canvi, i no modificar les dades que està utilitzant
la resta de companys, observeu com al final estem fent un **rollback**. Copieu
el següent en el fitxer **Exemple_13_M****odificacio.kt** :

    
    
    package exemples
    
    import classes.Comarca
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
        val t = sessio.beginTransaction()
        val com = sessio.get("classes.Comarca", "Camp de Morvedre") as Comarca
        com.provincia = "Castelló"
    
        sessio.merge(com)
    
        t.rollback()
    
        sessio.close()
    }

En realitat tant el mètode **persist()** com el mètode **merge()** serveixen
tant per a actualitzar un objecte ja existent (que ja era persistent) com per
a crear-ne un nou si no existia.

Llicenciat sota la  [Llicència Creative Commons Reconeixement NoComercial
CompartirIgual 2.5](http://creativecommons.org/licenses/by-nc-sa/2.5/)

