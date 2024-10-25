# 12\. Generació de taules a partir de les classes (voluntari)

**VOLUNTARI**

Durant aquest tema, i també en els exercicis, hem treballat sobre projectes en
els quals a partir de les taules d'una Base de Dades ja creada en qualsevol
Sistema Gestor de Bases de Dades (excepte SQLite) hem generat les classes.
Funciona molt bé aquesta generació de classes amb Hibernate.

També és possible el procés invers, és a dir, **generar les taules** en una
Base de Dades **a partir de les classes** creades en un projecte. Veurem un
exemple molt senzill de com és possible. Tanmateix hem de dir ja que aquesta
manera de funcionar no és tan efectiva com la inversa, i si l'estructura de
classes és complexa, podem tenir problemes per a generar les taules. Per això
l'exemple que posarem és molt senzil.

Aquesta pregunta és totalment voluntària. Si no teniu ganes o temps, no fa
falta que feu l'exemple mostrat a continuació. El que sí que és obligatori és
la generació de classes a partir de les taules, vist anteriorment.

L'exemple que intentarem implementar és una part de l'exemple de la
biblioteca, vist ja en el tema 4. Només treballarem sobre 2 classes
**Editorial** i **Llibre**.

Aquestes són les classes, millor dit les propietats de les classes, com veieu
amb una estructura molt senzilla:

    
    
    class Editorial (var codEd: String, var nom: String, var web: String, var llibres: MutableSet<Llibre>)
    
    class Llibre (var isbn: String, var titol: String, var pagines: Int, var editorial: Editorial)

Com veieu tenim una propietat en la classe **Llibre** que és de tipus
**Editorial** , per a marcar l'editorial del llibre. I en **Editorial** tenim
un **MutableSet** de tipus **Llibre** per a posar tots els llibres de
l'editorial.

La manera de crear les taules és molt senzilla, encara que si comencem de zero
és un poc laboriosa, i fa falta comprendre els fitxers de mapatge **hbm.xml**
en profunditat, ja que els construirem nosaltres. Després serà tan senzill com
guardar dades, fer-les permanents. Incloent una propietat en el fitxer
**hibernate.cfg.xml** es crearan les taules abans de guardar les dades.

Els fitxers de mapatge serien aquestos:

**Editorial.hbm.xml**

    
    
    <hibernate-mapping>
        <class name="biblio.Editorial" table="editorial" schema="public" catalog="biblio_gen">
            <id name="codEd" column="cod_ed"/>
            <property name="nom" column="nom"/>
            <property name="web" column="web"/>
            <set name="llibres" inverse="true">
                <key>
                    <column name="cod_ed" not-null="true"/>
                </key>
                <one-to-many not-found="ignore" class="biblio.Llibre"/>
            </set>
        </class>
    </hibernate-mapping>

**Llibre.hbm.xml**

    
    
    <hibernate-mapping>                                                                  
        <class name="biblio.Llibre" table="llibre" schema="public" catalog="biblio_gen"> 
            <id name="isbn" column="isbn"/>                                              
            <property name="titol" column="titol"/>                                      
            <property name="pagines" column="pagines"/>                                  
            <many-to-one name="editorial" class="biblio.Editorial">                      
                <column name="cod_ed" not-null="true"/>                                  
            </many-to-one>                                                               
        </class>                                                                         
    </hibernate-mapping>                                                                 

Evidentment haurem de fer la connexió amb la Base de Dades i incorporar
llibreries d'Hibernate, igual com hem fet fins el moment.  
  

Per a aquest exemple, utilitzarem un altra Base de Dades, usuari i
contrasenya, també en PostgreSQL. Serà sobre la BD, usuari i contrasenya
**biblio_gen**.

Una vegada feta la connexió a la Base de Dades, utilitzarem aquest
**hibernate.cfg.xml** :

    
    
    <?xml version='1.0' encoding='utf-8'?>
    <!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
    <hibernate-configuration>
      <session-factory>
        <property name="connection.url">jdbc:postgresql://89.36.214.106:5432/biblio_gen</property>
        <property name="connection.driver_class">org.postgresql.Driver</property>
        <property name="connection.username">biblio_gen</property>
        <property name="connection.password">biblio_gen</property>
        <!-- DB schema will be updated if needed -->
        <property name="hibernate.hbm2ddl.auto">create</property>
        <mapping resource="Editorial.hbm.xml"/>
        <mapping resource="Llibre.hbm.xml"/>
      </session-factory>
    </hibernate-configuration>

Aquest fitxer el podríem haver generat mapejant de la connexió, però sense
mapejar les taules, clar, que encara no existeixen. I sobre ell faríem algun
canvi:

  * En la línia 12 llavaríem el comentari sobre aquesta propietat. El que li estem dient és que ha de crear les taules (té l'inconvenient que si ja existien, esborrarà les anteriors)
  * Les línies 13 i 14 les hem incloses per a mapejar Editorial i Llibre amb aquestos fitxers de mapatge

I només faltarà un programeta que intente guardar les dades. Aquest seria un
exemple:

    
    
    package biblio
    import org.hibernate.cfg.Configuration
    import java.util.logging.Level
    import java.util.logging.LogManager
    
    fun main(args: Array<String>) {
            LogManager.getLogManager().getLogger("").setLevel(Level.SEVERE)
            val sessio = Configuration().configure().buildSessionFactory().openSession()
            val t = sessio.beginTransaction()
            val e = Editorial("ed5","Tabarca Llibres","www.tabarcallibres.com", mutableSetOf<Llibre>())
            val ll = Llibre("8480241815","L'ull de la boira",141,e)
            e.llibres.add(ll)
            sessio.persist(e)
            sessio.persist(ll)
            t.commit()
            sessio.close()
    }

En el següent vídeo s'arreplega tot el procés, des de la creació del projecte
fins tenir les taules creades.

**_Nota_**

En Eclipse es pot automatitzar un poc més el procés, ja que és capaç de
generar quasi totes les anotaions necessàries per a poder mapejar les classes
(sense utilitzar els fitxers de mapatge **hbm.xml**). Les classes, en Java,
quedarien així:

**classe Editorial** | **classe Llibre**  
---|---  
**import javax.persistence.Entity;**  
**import javax.persistence.Id;** **@Entity**  
**public class Editorial {**  
** @Id**  
**String codi;**  
**String nom;**  
**String web;**  
****  
**public Editorial() {**  
**super();**  
**}**  
**public Editorial(String codi, String nom, String web) {**  
**super();**  
**this.codi = codi;**  
**this.nom = nom;**  
**this.web = web;**  
**}**  
**public String getCodi() {**  
**return codi;**  
**}**  
**public void setCodi(String codi) {**  
**this.codi = codi;**  
**}**  
**public String getNom() {**  
**return nom;**  
**}**  
**public void setNom(String nom) {**  
**this.nom = nom;**  
**}**  
**public String getWeb() {**  
**return web;**  
**}**  
**public void setWeb(String web) {**  
**this.web = web;**  
**}**  
**}** | **import javax.persistence.Entity;  
import javax.persistence.Id;  
import javax.persistence.ManyToOne;  
  
@Entity  
public class Llibre {  
@Id  
String isbn;  
String titol;  
int pagines;  
@ManyToOne  
Editorial editorial;  
  
public Llibre() {  
super();  
}  
public Llibre(String isbn, String titol, int pagines, Editorial editorial) {  
super();  
this.isbn = isbn;  
this.titol = titol;  
this.pagines = pagines;  
this.editorial = editorial;  
}  
public String getIsbn() {  
return isbn;  
}  
public void setIsbn(String isbn) {  
this.isbn = isbn;  
}  
public String getTitol() {  
return titol;  
}  
public void setTitol(String titol) {  
this.titol = titol;  
}  
public int getPagines() {  
return pagines;  
}  
public void setPagines(int pagines) {  
this.pagines = pagines;  
}  
public Editorial getEditorial() {  
return editorial;  
}  
public void setEditorial(Editorial editorial) {  
this.editorial = editorial;  
}  
}**  
  
I amb aquesta informació es podran generar les sentències SQL que crearan les
taules en la Base de Dades. Aquestes sentències serien:

**taula EDITORIAL** | **taula LLIBRE  
**  
---|---  
******create table Editorial (  
codi varchar(255) not null,  
nom varchar(255),  
web varchar(255),  
primary key (codi)  
);** | ******create table Llibre (  
isbn varchar(255) not null,  
pagines int4 not null,  
titol varchar(255),  
editorial_codi varchar(255),  
primary key (isbn)  
);  
  
alter table Llibre  
add constraint FKap3159swqtnuj1i2q43xm0wk2  
foreign key (editorial_codi)  
references Editorial;**  
  
Observeu com fins i tot es crearia la clau externa en LLIBRE que apunta a
EDITORIAL

Aquest vídeo arreplega tot el procés:

Llicenciat sota la  [Llicència Creative Commons Reconeixement NoComercial
CompartirIgual 2.5](http://creativecommons.org/licenses/by-nc-sa/2.5/)

