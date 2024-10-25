# 11 - Resum del llenguatge HQL

Veurem d'una manera molt ràpida el llenguatge HQL. Sobretot insistirem en les
diferències amb SQL, al qual se sembla molt.

  * La clàusula **SELECT** és opcional. Si no la posem equival a agafar totes les propietats (és a dir, equivalent a SELECT * de SQL). És una operació molt habitual, ja que d'aquesta manera la llista d'objectes tornats és de la classe marcada en el from.
  * Es poden utilitzar àlies de les classes, posant **AS** després de la classe i abans de l'àlies. També podem prescindir del AS. Si utilitzem àlies, per a fer referència a la classe haurem d'utilitzar sempre l'àlies. Per exemple **FROM Comarques AS c** , o senzillament **FROM Comarques c**.
  * La clàusula **WHERE** és opcional. Si no la posem equival a agafar de totes les instàncies de la classe associada (és a dir, de totes les files de la taula).
  * S'admeten les clàusules **GROUP BY** , **HAVING** i **ORDER BY** , que funcionen igual que en SQL.
  * S'admeten **subconsultes** , si és que el SGBD les admet, clar.

Ací teniu uns quants exemples:

>```from Comarca```

Torna totes les instàncies de Comarques

>```from Comarca order by nomC```

Torna totes les instàncies de Comarques ordenades per nom de comarca

>```from Comarca where provincia='Castelló'```

Torna totes les comarques de la província de Castelló

>```from Comarca where provincia in ('Castelló','Alacant')```

Torna totes les comarques de les províncies de Castelló i Alacant

>```from Poblacio where altura between 700 and 750```

Torna totes les poblacions amb una altura entre 700 i 750 m,

>```from Institut where codpostal is null```

Torna els instituts que no tenen codi postal (no hi ha ningun)

>```from Comarca where nomC like 'A%'```

Torna les comarques el nom de les quals comença per A

>```select provincia, count(*) from Comarca group by provincia```

Torna les províncies i el número de comarques en cadascuna

>```select poblacio.nom , count(*) from Institut group by poblacio.nom```

Torna el nom de la població i el número d'instituts que té (només d'aquelles
que tenen algun institut)

>```select comarca.nomC , avg(altura) from Poblacio group by comarca.nomC having avg(altura) > 700 
```

Torna el nom de la comarca i l'altura mitjana d'aquelles comarques que tenen
una altura mitjana més gran que 700

>```from Poblacio where altura > (select avg(altura) from Poblacio)```

Torna les poblacions que tenen una altura major que la mitjana

>```from Poblacio p1 where p1.altura > (select avg(altura) from Poblacio p2 where p2.comarca=p1.comarca)```

Torna les poblacions que tenen una altura major que la mitjana de la seua
comarca

HQL també admet reunions en les classes. Per a fer una reunió normal, podem
posar la condició en el where i ja està, però si volem fer una reunió externa
(un **left join** per exemple) no ens valdrà aquesta opció, i haurem de fer la
reunió.

La sintaxi de la reunió és diferent en HQL.

En **SQL** posàvem la condició en l'apartat **ON** , després d'haver posat les
taules, i posàvem una condició d'igualtat entre els camps d'una i altra taula


>```SELECT ... FROM POBLACIO LEFT JOIN INSTITUT ON POBLACIO.cod_m=INSTITUT.cod_m```

En canvi en **HQL** després de les taules no posem **ON** , sinó que posarem
l'enllaç entre les dues taules. Aquest enllaç coincideix amb el nom de la
col·lecció d'elements de l'altre taula. En el cas de l'exemple que estem
veient és **institutses**. Per tant una reunió entre **Poblacions** i
**Instituts** es definiria així

>```from Poblacio p left join p.instituts```

Vejam un exemple un poc més elaborat, on traurem el nom de la població amb el
número d'instituts que té, fins i tot d'aquells que no tenen institut

>```select p.nom, count(i.codi) from Poblacio p left join p.instituts i group by p.nom order by p.nom```



Llicenciat sota la  [Llicència Creative Commons Reconeixement NoComercial
CompartirIgual 2.5](http://creativecommons.org/licenses/by-nc-sa/2.5/)

