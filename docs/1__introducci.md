# 1 - Introducció

Ja hem vist que a través de JDBC podem connectar fàcilment a Bases de Dades
Relacionals. Però tenim el problema del desfasament Objecte-Relacional. Com
que en Java i Kotlin utilitzem objectes, ha d'haver una part important de
programació en la conversió d'aquestos objectes a taules, per a que es puguen
guardar de forma permanent. Això ho hem intentat comprovar en l'**exercici
4.4** , destinat a fer la classe **GestionarRutesBD** de manera que siga
transparent la persistència de les dades a qui utilitza aquesta classe. Sobre
un exemple molt senzill, hem vist que la traducció entre els objectes i
únicament dues taules, ja suposava una miqueta de esforç. Podem veure per tant
que en una aplicació una miqueta més complicada, fer la traducció entre
objectes i taules és una feina que es pot fer, però que costarà prou.

Les eines de **mapatge objecte-relacional** (**ORM** : Object-Relational
Mapping) intenten minimitzar tant com siga possible el desfasament objecte-
relacional, automatitzant el procés de traspàs d’un sistema a l’altre.

En certa manera podríem dir que implementen una Base de Dades Orientada a
Objectes virtual perquè aporten característiques pròpies del paradigma OO,
però el substrat on s’acaben guardant els objectes és un SGBD Relacional.



Llicenciat sota la  [Llicència Creative Commons Reconeixement NoComercial
CompartirIgual 2.5](http://creativecommons.org/licenses/by-nc-sa/2.5/)

