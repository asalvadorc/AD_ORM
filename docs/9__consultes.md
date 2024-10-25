# 9 - Consultes

Hem vist com podem accedir molt còmodament a les classes que equivalen a una
taula. Però en moltes ocasions ens farà falta accedir no exactament a una
taula sinó a una combinació de taules, o en definitiva voldrem informació més
elaborada. Per a poder "interrogar" a la Base de Dades amb consultes més
complexes, Hibernate suporta un llenguatge de consulta Orientat a Objectes
anomenat **HQL** (_**Hibernate Query Language**_), molt paregut a SQL ja que
és una extensió Orientada a Objectes d'aquest.

En aquest sentit s'ha intentat fer un estàndar de llenguatge de consulta
anomenat **OQL** (_**Object Query Language**_), desenvolupat per un grup amb
ànim de crear un estàndar **ODMG** (Object Data Management Group). Aquest
estàndar no l'ha implementat al 100% cap producte comercial, i en realitat
tenim subconjunts d'aquest estàndar.

Utilitzarem la classe **Query** , i invocarem al mètode **createQuery()** de
**Session** , que justament torna una Query. Aquest seria un exemple. Observeu
que com a primer paràmetre se li passa la sentència, i com a segon paràmetre
el tipus que torna; com que les classes no les hem traduïdes a Kotlin, hem
d'especificar que són de Java:
```
val q = sessio.createQuery("from Comarca", Comarca::class.java)
```

<div style="background-color: #d6eaf8; color: black; padding: 5px;">
<b>Nota</b><br>
Abans no calia el segon paràmetre, però des de la versió d'<b>Hibernate 6</b> ,
marca com a <b>deprecated</b> el <b>createQuery</b> si no es posa aquest segon
paràmetre
</div><p></p>
Per a recuperar les dades tenim dues possibilitats, utilitzant dos mètodes de
la query:

  * Mètode **list()** : torna tots els resultats de la consulta en una col·lecció (**List)**. Aquest mètode fa una crida única al SGBD i es duran totes les dades. Haurà d'haver, per tant, memòria suficient per a que càpiguen tots els resultats. Si és una quantitat gran de resultats, tardarà molt en executar-se.
  * Mètode **iterate()** : torna un iterador per a poder recórrer els resultats de la consulta. Hibernate executa la sentència, però només torna els identificadors de les files del resultat, i cada vegada que fem **next()** del iterador, s'executa realment la consulta tornant la següent fila. Per tant fa falta molta menys memòria. Per contra es fan mots més accessos a la Base de Dades, encara que cadascun tardarà molt poc, però en total tardarà més. Es pot fins i tot fixar la quantitat de files a tornar amb el mètode **setFetchSize()**.

En el següents dos exemples, que són equivalents, es trau una llista de totes
les comarques. El primer utilitza el mètode **list()** i el segon el mètode
**iterate()**. I en aquesta ocasió utilitzem un iterador per a recórrer la
llista (List) en compte de **foreach** , per veure més d'una manera, i per
similitud amb el segon exemple. Poseu a aquest primer exemple el nom de
**Exemple_21_AccesAmbList.kt** :

    
    
    package exemples
    
    import classes.Comarca
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
        val q = sessio.createQuery ("from Comarca", Comarca::class.java)
    
        val llista = q.list ()
    
        val it = llista.iterator ()
        while (it.hasNext()) {
            val com = it.next() // no fa falta fer un casting perquè ja sap que és Comarca
            println(com.nomC + " - " + com.provincia)
        }
        sessio.close()
    }

Aquest serà el resultat:
```
    Safor - València  
    Horta Sud - València  
    Foia de Bunyol - València  
    Plana Baixa - Castelló  
    Horta Nord - València  
    Racó - València  
    Plana d'Utiel - València  
    Vall de Cofrents - València  
    Ribera Baixa - València  
    Ribera Alta - València  
    Marina Alta - Alacant  
    Serrans - València  
    València - València  
    Baix Maestrat - Castelló  
    Marina Baixa - Alacant  
    Vall d'Albaida - València  
    Canal de Navarrés - València  
    Horta Oest - València  
    Camp de Túria - València  
    Alt Millars - Castelló  
    Baix Vinalopó - Alacant  
    Comtat - Alacant  
    Alt Palància - Castelló  
    Plana Alta - Castelló  
    Alacantí - Alacant  
    Camp de Morvedre - València  
    Alt Vinalopó - Alacant  
    Costera - València  
    Alcalatén - Castelló  
    Alcoià - Alacant  
    Vinalopó Mitjà - Alacant  
    Alt Maestrat - Castelló  
    Baix Segura - Alacant  
    Ports - Castelló
```
Encara que la manera més curta i potser més clara és utilitzant un bucle
**foreach** amb el mètode **list()**. Només haurem d'anar amb compte de fer un
_**cast**_ per a que sàpiga quin tipus d'element és. Copieu aquest tercer
exemple al fitxer **Exemple_22_AccesAmbListForEach.kt** :

    
    
    package exemples
    
    import classes.Comarca
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
        val q = sessio.createQuery ("from Comarca", Comarca::class.java)
    
        for (c in q.list()) {
            println(c.nomC + " --- " + c.provincia)
        }
    
        sessio.close()
    }

Si sabem que la consulta tornarà únicament una fila, podem assignar aquesta
fila a un objecte de la classe de la taula afectada, posant el mètode
**uniqueResult()** en la creació de la query, així ens estalviem passos.
Copieu el següent programa al fitxer kotlin
**Exemple_23_AccesAmbUniqueResult.kt** :

    
    
    package exemples
    
    import classes.Comarca
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
    
        val d = sessio.createQuery ("from Comarca where nomC='Alcalatén'",Comarca::class.java).uniqueResult()
    
        println(d.nomC + " - " + d.provincia)
    
        sessio.close()
    }

Però en realitat en aquest exemple poca cosa hem guanyat, perquè per a agafar
l'objecte corresponent a una única fila d'una taula, ja ho féiem amb
**session.get()**. Més endavant veurem consultes més complicades on trobarem
la utilitat.

## 9.1 - Paràmetres en les consultes

**createQuery()** admet també la utilització de paràmetres, igual que feia les
sentències **PreparedStatement** del tema anterior. La manera de posar-les és
prou similar al vist en aquella ocasió. Ara, però, ho ampliarem un poc.

La manera de posar valor als paràmetres serà utilitzant el mètodes
**setParameter()** que tindrà 2 paràmetres, el primer per a assenyalar el
paràmetre, i el segon per a indicar el valor. De moment tot igual que en el
**PreparedStatement**.

Però ara veurem 2 maneres de posar paràmetres en la consulta:

  * Amb **?i** , utilitzant el mètode **setParameter(int,_valor_)** i indicant el número **el mateix que després de la interrogant (la i)**.
  * Amb **:_nom_ **, utilitzant el mètode **setParameter(string,_valor_)** i indicant el nom del paràmetre

Ho veurem molt més clar en un exemple, millor dit, en dues versions del mateix
exemple. Intentarem traure les poblacions d'una determinada comarca, que tenen
una altura determinada o major. I aquestos dos valors els posarem com a
paràmetres, i així podrem practicar les dues formes. Els valors que posarem
per a la comarca serà **Alcalatén** , i l'altura **500**

En aquesta primera versió assenyalem els paràmetres amb **?i** a l'estil de
**JDBC**. Copieu el següent codi al fitxer
**Exemple_31_ConsultesAmbParametres1.kt**

    
    
    package exemples
    
    import classes.Poblacio
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
    
        val q = sessio.createQuery("from Poblacio where altura>=?1 and comarca.nomC=?2", Poblacio::class.java)
        q.setParameter(1, 500)
        q.setParameter(2, "Alcalatén")
    
        for (p in q.list()) {
            p as Poblacio
            println(p.nom + " - " + p.altura)
        }
    
        sessio.close()
    }


<div style="background-color: #d6eaf8; color: black; padding: 5px;">
<b>Nota</b></br>
En versions anteriors a banda de <b>setParameter()</b> s'utilitzaven els mètodes
<b>setInteger()</b> , <b>setString()</b>... però des de la versió 6 d'Hibernate ja
no existeixen
</div><p></p>

En la segona posem els paràmetres de l'altra manera. Copieu el següent codi al
fitxer **Exemple_32_ConsultesAmbParametres2.kt**

    
    
    package exemples
    
    import classes.Poblacio
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
    
        val q = sessio.createQuery("from Poblacio where altura>=:alt and comarca.nomC=:com", Poblacio::class.java)
        q.setParameter("alt",500)
        q.setParameter("com","Alcalatén")
        for (p in q.list()) {
            p as Poblacio
            println(p.nom + " - " + p.altura)
        }
    
        sessio.close()
    }

## 9.2 - Consultes diferents a una classe ja definida

De moment en les consultes sempre hem tornat una fila o més però sempre**d'una
única taula** , o millor dit, **una classe** equivalent a una taula.

Però hi haurà moltes ocasions en què voldrem fer una consulta diferent, on
intervinga més d'una taula (o classe), que torne funcions de grup (SUM, AVG,
...), en definitiva que no siga una sentència equivalent a **SELECT * FROM
...**

Per tant ara no tornarà un objecte o una col·lecció d'objectes dels que ja
tenim definits. Podem tenir més d'un cas, i a continuació els intentarem
tractar:

Consultes que tornen més d'un objecte dels definits

Són consultes en les quals hi haurà una reunió de taules (millor dit, de les
classes equivalents), i es tornen tots els camps (totes les propietats).

Podríem definir-nos una classe que englobara totes les propietats de tots els
objectes, però hi ha una solució més ràpida. Els resultats s'obtenen en un
**array d'objectes** , on el primer objecte serà l'equivalent de la primera
taula, el segon l'equivalent de la segona, etc.

Per tant, la classe que posem com a segon paràmetre del **createQuery** sera
**Array::class.java**.

Aquest exemple, on traurem el nom de la comarca i la província, és només
il·lustratiu. Està clar que es pot fer molt més curt i senzill agafant
únicament les Poblacions i a partir d'elles traure la comarca. Copieu el
següent codi al fitxer **Exemple_41_ConsultaComplexa1.kt** :

    
    
    package exemples
    
    import classes.Poblacio
    import classes.Comarca
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
    
        val q = sessio.createQuery("from Poblacio p, Comarca c where p.comarca.nomC=c.nomC order by p.nom", Array::class.java)
    
        for(tot in q.list()) {
            val p = tot[0] as Poblacio
            val c = tot[1] as Comarca
            println(p.nom + " (" + c.nomC + ". " + c.provincia + ")")
        }
    
        sessio.close()
    }

Com veiem, un poc "rotllo"...

**<u>Consultes que tornen un únic valor</u>**

Aquest és el cas més senzill, ja que el valor tornat el podem agafar del tipus
bàsic del valor tornat. Normalment és després d'utilitzar una funció de grup.
Recordem que ha de tornar un únic valor (una única fila i una única columna,
per dir-lo així).

La classe que posem com a segon paràmetre del **createQuery** harà de ser la
del valor tornat. En aquest cas que calculem la mitjana, a classe oportuna
serà **Double**. Si comptàrem, per exemple, seria **Long**.

Haurem d'utilitzar el mètode **uniqueResult()** ja comentat amb anterioritat.
L'exemple serà per a calcular l'altura mitjana de totes les poblacions. Copieu
el següent codi al fitxer Kotlin **Exemple_42_ConsultaComplexa2.kt** :

    
    
    package exemples
    
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
    
        val q = sessio.createQuery("select avg(altura) from Poblacio",Double::class.java)
    
        val mitjana = q.uniqueResult ()
    
        println("Altura mitjana: " + mitjana)
    
        sessio.close()
    }

**<u>Consultes amb qualsevol resultat</u>**

Una de les solucions seria crear una classe amb totes les propietats que torna
la consulta. No veurem aquesta solució per tediosa.

Farem com en el primer apartat d'aquesta pregunta, gestionant l'**array
d'objectes** que s'obtenen, però en aquesta ocasió cada objecte de l'array
serà un tipus simple. Copieu el següent codi al fitxer
**Exemple_43_ConsultaComplexa3.kt** :

    
    
    package exemples
    
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
    
        val q = sessio.createQuery("select c.nomC,count(p.codM),avg(p.altura) "
                + "from Comarca c , Poblacio p "
                + "where c.nomC=p.comarca.nomC "
                + "group by c.nomC "
                + "order by c.nomC", Array::class.java);
        for (tot in q.list()){
            println("Comarca: " + tot[0] + ". Núm. pobles: " + tot[1] + ". Altura mitjana: " + tot[2]);
        }
        sessio.close()
    }

Només hem d'anar amb compte amb posar com a segon paràmetre de **createQuery**
el tipus **Array::class.java**.    

Llicenciat sota la  [Llicència Creative Commons Reconeixement NoComercial
CompartirIgual 2.5](http://creativecommons.org/licenses/by-nc-sa/2.5/)

