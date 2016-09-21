---
title: Implementa OpenId Connect, no más passwords!
updated: 2015-09-12 15:56
---

En el mundo web actual el protocolo de autenticación que parece ya se ha establecido en OpenId Connect que vendría a ser una extensión del más conocido OAuth 2.0. OAuth está diseñado para hacer frente a procesos de autorización (...) mientras OpenId Connect es una especificación que extiende OAuth 2.0 para proporcionarle un mecanismo de autenticación y por tanto poder identificar al usuario. 

Por poner un ejemplo para hospedar este blog estoy registrado en el servicio web que me proporciona GitHub y para dotar del apartado de comentarios he optado por darme de alta en el servicio que proporciona Discuss [^1]. Para cada uno tengo un username y un password diferente, porque en este caso estos servicios no optan por implementar este protocolo. Mientras escribo post pongo música molona de fondo con Spotify en el cual me he registrado con la identidad que me proporciona un tercero en este caso facebook. 

[Según esta clafisicación que acabamos de hablar podríamos enumerar diferentes tipos de proveedores de servicios:](#)

1 - Los que directamente te registras creando un usuario y una contraseña (GitHub)

2 - Los que te dejan registrarte mediante un proveedor de identidad o también conocido como social login (Spotify).

3 - Los que te dejan registrarte mediante un proveedor de identidad pero además piden un password a lo largo del proceso de registro (Discuss)

### Desde el punto de vista de un consumidor de servicios (user mode)
En el párrafo anterior daba a entender que como usuario activo de internet en tu día a día te ves obligado a recordar una serie de combinaciones de user/passwords que se suele ir de las manos. 

Otro tema a tener en cuenta es que tendemos a usar servicios desde diferentes dispositivos, como el pc del trabajo, el de casa, el smart phone, la tablet, ...

Algunos usuarios optan por usar siempre el mismo password en todos los sites... No es buena idea, le estás dando información muy valiosa a cientos de proveedores de servicios de internet entre los cuales el 90% serán empresas fiables con sus políticas de seguridad pero otro 10% podría no serlo. 

Otra opción es usar un password diferente en cada sitio y tener un lugar que solo tu conozcas donde guardas estas contraseñas para acordarte de todas.. la variante que yo uso es tener una password complicada y fuerte y en cada site concaternarle palabras delante y detrás según un pattern mental que te creas... mi pattern no es infalible y tengo que resetear passwords de sites más a menudo de lo que me gustaría porque no me acuerdo. 

> Como usuario me gustaría tener una o dos identidades digitales y de este modo confiar únicamente mis credenciales a estas empresas. Suena un poco utópico pero si todos los sites que proveen servicios proporcionasen la posibilidad de usar identidad de terceros se acabaría el problema de tener que recordad N passwords de N sites. 

### Desde el punto de vista de un proveedor de servicios (developer mode)
Como developer conocedor de este tipo de escenarios a día de hoy no se me ocurre ningún inconveniente por el cual no implementaría este tipo de flujo y así permitir a mis usuarios que decidan si quieren usar la identidad que ya provee un tercero. En estos momentos los proveedores de identidad que destacan son las grandes empresas de servicios en internet como Microsoft, Google, Facebook o Twitter. 

Iré un poco más allá, de la clasificación inicial me incomoda especialmente el caso 3. Este site me permite usar mis credenciales de Google al registrarme. A lo largo del proceso soy reenviado a la pantalla de login de mi proveedor de identidad (Google) con el que me autentico y luego el propio flujo de registro me devuelve al site original donde para finalizar el proceso de registro me piden una contraseña. Como usuario no entiendo porque me pides un password si ya lo he escrito en el formulario de autenticación de Google. 

>Como developer entendería que completes la información del usuario en el proceso de registro con información adicional que requieres para poder dar servicio al usuario y que Google no te proporciona pero me molesta que me pidas un password. 

Siguiendo en modo developer que ha visto memberships de empresas sé que no siempre los passwords están encriptados con algoritmos seguros y también que todos los developers de la compañía suelen tener acceso a esa info. Porque me inquieta esto... porque les he dado mi correo y una contraseña... si hubiese optado por la técnica de usar el mismo password en todos los sites que me registro... los programadores de Discuss ya podrían obtener tranquilamente de su base de datos mis credenciales de Google y como las mías las de todos los usuarios que no estén atentos a la jugada. Aclarar que no tengo ni idea de quien forma Discuss y sus políticas de seguridad, ni estoy diciendo que no sea un sitio seguro. Al contrario agradecer este tipo de servicios (sin coste). Simplemente lo tomo como ejemplo en este post como podría haber tomado como ejemplo a 20 sites más de los que estoy registrado. 

### Más info

Este protocolo que da título al post no sólo es útil para el mundo web "público". En una compañia podría darse el caso que los usuarios accedan a lo largo del día a varias aplicaciones web de la compañía [^2] para hacer su trabajo diario y tengan que cambiar de una a otra bastante a menudo. Si cada aplicación establece su propio mecanismo de autenticación cada vez que cambian de aplicación han de logarse de nuevo... y esto les crispa. En nuestro caso implementamos una relación entre Active Directory Local y Cloud (ADFS) y montamos un identity server [^3]. De este modo todas las aplicaciones 'usan' a Active Directory como su proveedor de identidad. Usuarios contentos que ahora ya pueden leer el marca 2 minutos más gracias al tiempo que se ahorran, los de IT lo dan de baja de AD y ya saben que no podrá autenticarse en ninguna aplicación y el director feliz porque de cara a pasar ISOs de calidad era importante tener centralizado en un punto todos los usuarios. 

Te dejo un serie de enlaces por si te interesa el tema y quieres adentrarte en la especificación o en como llevar a cabo la implementación para una api o servicio que estés desarrollando. 

<a href='http://openid.net/connect/'>Official Spec</a>

<a href='https://azure.microsoft.com/en-us/documentation/articles/active-directory-protocols-openid-connect-code/'>Authorize access to web applications using OpenID Connect and Azure Active Directory</a>

<a href='https://azure.microsoft.com/es-es/documentation/articles/active-directory-code-samples/'>Ejemplos .net</a>

<a href='https://ptgmedia.pearsoncmg.com/images/9780735696945/samplepages/9780735696945.pdf'>Modern Authentication with Azure Active Directory for Web Applications (Libro)</a>

[^1]: Discuss si que permite el registro a través de un tercero como Google o Facebook pero pide un password al finalizar el registro. A la práctica necesito recordar ese password que por coherencia debería ser diferente al de mi proveedor de identidad. 
[^2]: Algunas desarrollas en .Net (web forms), otras .net (mvc) y otras en Java.
[^3]: En este tipo de escenarios donde interviene un AD local necesitarás conocimientos de WsFederation. 

