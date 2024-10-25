# 10 - Consultes d'actualització

Aquesta pregunta no té tanta importància com les altres, ja que
actualitzacions ja sabíem fer amb els mètodes **persist()** , **remove()** i
**mege()** de la sessió (o els seus equivalents ja deprecated **save()** ,
**delete()** i **update()** )

Pràcticament l'única utilitat serà fer actualitzacions massives, que afecten a
més d'una fila (millor dit, més d'un objecte). HQL també permet fer-les.

Lamentablement, aquestes sentències HQL no tornen cap classe, i per tant no
podem posar res vàlid en el segon paràmetre de **createQuery()**. Per tant,
per a aquest tipus de sentència ens veurem **obligats a utilitzar la versió
deprecated de createQuery()** , la que no li posem un segon paràmetre

Mirem la seua sintaxi:

**<u>DELETE</u>**
```
DELETE [FROM] NomEntitat
    [WHERE condició]
```
Esborrarà tots els objecte que acomplesquen la condició. Observeu que:

  * FROM és opcional
  * La clàusula WHERE és opcional. Si no es posa s'esborraran tots els objectes.
  * En el FROM només es pot posar una entitat. No val a posar més d'una ni reunions ni res. Sí que es podran posar subconsultes en la clàusula WHERE, i en les consultes sí que es pot fer el que es vulga (reunions, ...).

Per a executar la consulta s'utilitzarà el mètode
**Query.****executeUpdate()** (en compte de **Query.****list()** per exemple).
Aquest mètode torna un enter que serà el número de files (objectes)
esborrades.

I recordeu que s'ha de confirmar la transacció per a que tinguen efecte els
canvis.

Ací tenim un exemple, al qual li hem aplicat **ROLLBACK** al final, per a no
fer efectiu l'esborrat. Però sí que diu quantes files "s'esborrarien".

El següent exemple "esborraria" els Instituts de la província de Castelló, que
són els que el seu codi comença per 12, i es diu quants serien els afectats,
però després es rebutgen els canvis en fer **rollback()**. Copieu el següent
codi al fitxer **Exemple_51_EsborratMassiu.kt** :

    
    
    package exemples
    
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
    
        val t = sessio.beginTransaction()
    
        val files = sessio.createQuery("delete Institut where codi like '12%'").executeUpdate()
    
        println("S'han esborrat " + files + " files.")
    
        t.rollback()
    
        sessio.close()
    }

Ens dirà que s'han esborrat 52 files, però després no les esborrarà de
veritat.

**<u>UPDATE</u>**
```
UPDATE [FROM] NomEntitat
    SET propietat = valor [, propietat2 = valor2, ...]
        [ WHERE condicio ]
```
Les observacions fetes en l'apartat de DELETE també valen ací. I observeu que
podem modificar més d'un camp, separant-los per comes. Ací tenim un exemple,
en el qual fem també **rollback()** per a deixar les coses com estaven. Copieu
el següent codi al fitxer **Exemple_52_ModificacioMassiva.kt** :

    
    
    package exemples
    
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
        LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
        val sessio = Configuration().configure().buildSessionFactory().openSession()
    
        val t = sessio.beginTransaction()
    
        val files = sessio.createQuery("update Poblacio set poblacio = poblacio *1.05 where poblacio < 200").executeUpdate()
    
        println("S'han modificat " + files + " files.")
    
        t.rollback()
    
        sessio.close()
    }

**<u>INSERT</u>**

```
INSERT INTO NomEntitat (llista_propietats)
    Sentència_select
```
Ara tindrem els següents condicionants:

  * No és vàlida la sentència INSERT INTO ... VALUES ... que servia per a introduir una fila donant-li els valors. Si volem fer això, millor utilitzar **save()**. Per tant només podrem introduir valors procedents d'una altra taula.
  * La sentència SELECT pot ser qualsevol sentència HQL. Només hem de cuidar que els tipus tornats per la consulta siguen els que espera INSERT (és a dir, del mateix tipus que les propietats de la classe associada a la taula).
  * Per a la propietat **id** , que és la que correspon a la clau principal, podem tenir 2 opcions: 
    * es pot especificar en la llista de propietats, i aleshores li haurem de proporcionar un valor.
    * es pot ometre, sempre i quan la propietat estiga definida com **increment** (no **assigned**); això vol dir que es correspon a un camp autoincrementat. Aleshores serà el SGBD qui li assignarà el valor

Aquest seria un exemple. En ell intentem introduir una nova comarca a partir
d'una hipotètica taula de comarques noves (_NovesComarques_). No la podrem
provar perquè no existeix aquesta taula, millor dit, aquesta classe. L'hauríem
de crear, fer-li el mapatge, i posteriorment executar açò. Per tant ho veiem
únicament a mode il·lustratiu:
```
val t = sessio.beginTransaction()
val files = sessio.createQuery("insert into Comarca (nomC, provincia) "  
    \+ " select n.nomC, n.provincia from NovesComarques n").executeUpdate()
println("S'han introduït " + files + " files.")
t.rollback()
```


Llicenciat sota la  [Llicència Creative Commons Reconeixement NoComercial
CompartirIgual 2.5](http://creativecommons.org/licenses/by-nc-sa/2.5/)

