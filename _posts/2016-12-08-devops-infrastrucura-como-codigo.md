---
title: DevOps y Infrastructura cómo código
updated: 2016-10-12 12:00
---

A lo largo de estos dos últimos años se ha puesto de moda el termino DevOps y también el concepto infrastructura como código IAC. 
En este post, hablaré sobre estos conceptos explicándo un poco mis experiéncias a lo largo de más de 10 años involucrado en areas de desarrollo de software. 

Devops es un término que estamos usando para definir esa serie de operaciones que se realizan desde que un software se desarrolla hasta que corre en una máquina 
ya sea un entorno de producción, desarrollo, integración.. 

Por norma general para cubrir estos escenarios hay dos roles claramente separados, los que desarrollan las aplicaciones que correran en las máquinas
y los que mantienen esa infraestructura de máquinas. 

### Mis inicios
En mi caso al principio de desarrollar todo esto era más de estar por casa, en la empresa donde empecé teníamos un servidor de desarrollo que era un PC debajo
de una mesa en el que convivían todo tipo de versiones y pruebas. A medida que la arquitectura de la aplicación paso a ser distribuida intervenían más máquinas
y el sistema del PC deja de escalar. 

### Evolución I - Entornos de integración
Otro tema es que uno va adquiriendo más profesionalidad y se empieza a sacar rendimiento a entornos de integración donde se prueban 
y consolidan las versiones de software. Por tanto la necesidad de máquina crece exponencialmente a medida que en el desarrollo de software evoluciona. 

### Evolución II - Involucramos IT + Virtualización
Con el tiempo la empresa donde estaba pasó a tener area própia de IT. Estos montaron un sistema de virtualización y mi Pc paso a ser un servidor enracado en el CPD de la empresa que mantenía el area de IT, al cual mediante una solicitud o ticket podías pedirle a IT que crease 
una imagen nueva o modificase las características de otra, etc. Esto ya permitía tener más de una máquina. 

### Integración continua
Hasta ahora he explicado como pasamos de un desarrollo campechano o anticuado a uno más enterprise donde surge la necesidad de varios entornos, 
interviene IT y se empiezan a usar sistemas de virtualización.

Con el tiempo crecemos en el desarrollo de software, nos interesamos por ALM y nos proponemos realizar integración continua. Esto implica que 
desde tu herramienta de control de versionado puedes configurar por norma general que se realice una build cada cierto tiempo o desencanado a partir de un 
evento como podría ser un commit de una nueva feature. Esto es interesante para asegurarse que el código que vamos subiendo al repo no 'rompe' la build y es 
posible configurar además que esa versión de tu software se despliegue automáticamente en un entorno o varios o configurarlo según rama... 

En principio lo mínimo recomendable es asegurarse que no se rompe la build al subir código, pero como comentaba es interesante si tenemos equipos de QA
automatizar el despliegue a un entorno de pre o integración donde esta gente pueda probar y validar las nuevas funcionalidades durante el proceso de desarrollo
y aportar feedback al equipo de desarrollo sin tener que esperar al final de un sprint o entrega. 

Ya pensando en el concepto de integración contiua podría darse el caso que nos interese tener más de un entorno para que diferentes equipos de negocio 
o clientes puedan hacer pruebas del software y no se mezclen entre ellas. Como podéis ver la necesidad de máquinas y la agilidad que necesita a la hora de 
gestionarlas un equipo de desarrollo va creciendo exponencialmente a medida que vamos mejorando y profesionalizando el proceso de desarrollar software.  

### ¿Qué es Infrastructure as Code IAC?
Como ya mencionaba, a día de hoy existen herramientas de 'virtualización' muy potentes que nos permiten crear esos recursos que necesitamos sin necesidad de
disponer de una máquina para cada entorno y trabajando con imágenes predefinidas lo cual nos ahorra mucho tiempo. Normalmente un buen sistema de virtualización
permite crear estos recuersos a partir de scripts que pueden almacenarse en ficheros. El concepto de tener estos scripts versionados junto a cada versión de código
de tu proyecto es bastante interesante sin duda... sin más, esto es IAC tal vez hace tiempo que lo practicas y no le habías puesto nombre.

### ¿Qué es DevOps?
La definición de wikipedia, la tenéis al principio del post. Para completar, Devops, devs -> developer | ops-> operaciones. Por tanto se explica como ese subconjunto entre developers, 
qa y IT. Otra explicación más casera sería decir que es el tio o rol que trabaja los scripts y tiene criterio para definir este escenario. A las empresas y equipos que les gustan las etiquetas ya tienen 
una más. Pero como IAC considero que el devops hace mucho años que existe en algunos equipos simplemente ahora le hemos puesto nombre y lo hemos profesionalizado más. 

Hasta aquí el post de hoy, espero os resulte interesante. En siguientes posts dejaré de vender humo y mostaré como implementar este tipo de escenarios con 
herramientas modernas de hoy en día. 



