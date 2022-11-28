# Autenticación y autorización en Windows

## 1.- Introducción
Que es autenticación ? Es el proceso de verificar la identidad de un usuario. Este proceso se utiliza para asegurar que el usuario es realmente quien dice ser.
[Mas información](https://technet.microsoft.com/en-us/library/dn751047(v=ws.11).aspx)

Los sistemas Windows requieren que todo usuario se autentique a la hora de acceder a un recurso tanto local como en red.
Windows necesitara comprobar las credenciales para saber si se tienen los permisos necesarios para el acceso.

Por defecto, los sistemas Windows, las credenciales se validan en local contra la base de datos Security Account Manager (SAM) o contra un controlador de dominio en caso de tratase 
de una maquina unida a un dominio Active Directory. Este proceso se llevará a cabo mediante el servicio Winlogon.

## 2.- Windows Logon
Logon es el responsable de recoger las credenciales y proporcionar la autenticación de usuario para posteriormente comprobar si se dispone de los permisos necesarios
para realizar la acción que se desea frente a un recurso. [Mas información](https://technet.microsoft.com/en-us/library/cc780332(v=ws.10).aspx)

Distintos escenarios de inicio de sesión:

  - Inicio de sesión interactivo: Este es el proceso llevado acabo cuando un usuario introduce sus credenciales en ventana de diálogo del tipo "Iniciar sesión en Windows" al inicio del equipo.
  - Inicio de sesión de red: Con las credenciales ya validadas, el inicio de sesión en red se lleva a cabo cuando el usuario o servicio desea acceder a algún recurso en red y se necesita validar si dispone de privilegios para tal. Generalmente ocurre de manera transparente al usuario. Para poder llevar a cabo este proceso, Windows necesita incluir diversos mecanismos de autenticación tales como Kerberos, certificados de clave publica, Secure Socket Layer/Transport Layer Security (SSL/TLS), Digest o NTLM.
  - Inicio de sesión mediante tarjetas inteligentes o Smart card logon: Se debe disponer de una tarjeta inteligente y su PIN asociado, asi como un lector.
  - Inicio de sesión biométrico: Se acepta este tipo de autenticación donde una característica biométrica como la huella dactilar.

## 3.- Autenticación y procesamiento de credenciales
Para hacer el proceso de autenticación posible, es necesario que la parte servidor disponga de las claves criptograficas correctas de cada usuario para poder realizar las correspondientes comprobaciones.

¿ Por que Windows no pide al usuario constantemente que introduzca las credenciales cada vez que accede a cualquier recurso en red ? Esto es posible gracias a un concepto llamado Single Sign-On.

Security Support Provider Interface o SSPI, es una API proporcionada por Microsoft para facilitar los servicios de autenticación, integridad de mensajes, privacidad, etc.

SSPI proporciona un mecanismo mediante el cual dos equipos que deseen comunicarse no necesitarán explícitamente especificar el protocolo de autenticación a utilizar. El protocolo a utilizar es negociado antes de empezar la autenticación. Windows Server, desde su versión 2003, prefiere el uso de Kerberos siempre y cuando sea posible, es decir, el cliente también sea compatible con dicho protocolo.

Se listan los principales SPP que interactúan con SSPI por defecto. Cada uno de estos se utiliza de manera distinta para llevar a cabo la autenticación en un contexto de red.

  - Kerberos: Este SSP utiliza la implementación de Microsoft de la versión 5 de Kerberos y es el método preferido a utilizar para la autenticación en red. Este protocolo utiliza una contraseña o tarjeta inteligente para realizarel inicio de sesión. Ubicación: %windir%\Windows\System32\kerberos.dll
  - NTLM: Implementa el famoso protocolo de desafio/respuesta desarrollado por Microsoft y es aún mantenido para proporcionar compatibilidad con versiones anteriores de Windows. El paquete SSP NTLM incluye la implementación tanto de NTLMv1 como NTLMv2. Ubicación: %windir%\Windows\System32\mvs1_0.dll
  - Digest: Es un mecanismo de desafio/respuesta. Este protocolo ha sido uno de los métodos usados en servidores web para negociar credenciales, tales como nombre de usuario y contraseña, desde el navegador web. También puede usarse en LDAP,HTTP o SASL. Las credenciales se envian por la red en forma de hash MD5 que puede ser descifrada ya que la calve de cifrado fue publicada por Microsoft.
  - Schannel: Schannel implementa los protocolos Secure Socket Layer (SSL) y Transport Layer Security (TLS) utilizados para acceder a una web de modo seguro. Ubicación: %windir%\Windows\System32\schannel.dll
  - Negotiate: Se utiliza para la negociar el protocolo de autenticación a utilizar. En concreto, Negotiate con Kerberos y NTLM como opciones y seleccionara Kerberos por defecto siempre y cuando sea posible. Ubicación: %windir%\Windows\System32\lsasrv.dll

Especificando un unico protocolo de autenticación:

  1. El cliente solicita acceso a un recurso.
  2. El servidor responde con el protocolo que necesita utilizarse y un reto de autenticación.
  3. El cliente examina la respuesta y determina si soporta dicho protocolo. Si es así, el proceso de autenticacón continua. De no ser soportado, la autenticación falla con independencia de que el cliente tenga autorización para el recurso.

Negociar el protocolo:

  1. El cliente solicita acceso a un recurso.
  2. El servidor contesta con una lista de protocolos de autenticación que soporta y un reto de autenticación basado en el protocolo a utilizar por preferencia.
  3. El cliente examina la respuesta y determina si soporta alguno de los protocolos listados por el servidor.
   - Si el cliente soporta el protocolo preferido, el proceso de autenticaicón continúa.
   - Si el cliente no soporta el protocolo sugerido por el servidor, pero sí otro de los soportados, se lo comunica al servidor y la autenticación continúa.
   - Si el cliente no soporta ninguno de los protocolos, la autenticaicón falla.


Signle Sign-on(SSO): Los usuarios sólo necesitan suministrar sus credenciales una vez para consecutivamente acceder a los distintos recursos en red que necesiten autenticación sin necesidad introducir dichas credenciales una y otra vez.

### Local Security Authority: Windows guarda de manera local en memoria dichas credenciales en el subsistema Local Security Authority (LSA).

  - Administrar la politica de seguridad local.
  - Proporciona los servicios para la autenticación o inicio de sesión interactivo.
  - Generar tokens de acceso. 
  - Administrar las politicas de auditoria.

Hay dos formas en las que LAS hace la comprovación de dichas credenciales:
  - Si las credenciales son locales. (SAM)
  - Si las credenciales pertenecen a un dominio contactará con el controlador del dominio para verificar que son correctas.

El proceso Local Security Authority Subsystem Service (LSASS) es el responsable de imponer las politicas de seguridad, administrar los cambios de contraseña y crear access tokens. Se encarga de mantener en memoria un registro de las politicas de seguridad de las contraseñas y sus credenciales de los usuarios cuyas sesiones Windows están activas. 

### LSASS almacenamiento de credenciales:
  - Tickets Kerberos.
  - Hashes NT.
  - Hashes LAN Manager o LM.
  - En plano.

Ocasiones en donde se guardan las credenciales con LSASS:
  - Iniciar sesión en la maquina de manera local.
  - Iniciar sesión en la maquina de manera remota.
  - Ejecutando alguna acción con el comando runas.
  - Ejecutando un servicio Windows que requiere autenticación.
  - Ejecutar una tarea programada que requiere de autenticación.
  - Ejecutar una tarea en la máquina local mediante una herramienta de administración remota.
 
 [Recomendaciones para mejorar LSASS](https://technet.microsoft.com/en-us/library/dn408187(v=ws.11).aspx) - [Mas información](https://technet.microsoft.com/en-us/itpro/windows/whats-new/security)
 
### Almacenamiento de las credenciales

¿ Donde almacena Windows las credenciales de manera permanente o temporal ?
  - SAM:
  - Las contraseñas no son almacenadas en plano. En su lugar se almacenan su hashes NT se encuentra en dicho archivo.
  - LASSAS: Almacena las credenciales para aquellos usuarios con sesiones activas para acceder a los distintos recursos de red sin necesidad de reintroducir sus credenciales constantemente.
  - LSA Secret: En algunas ocasiones, LSA guarda ciertas credenciales en disco de manera cifrada.
      - Contraseña de la cuenta de equipo de Active Directory. (Esto se utilizara cuando el controlador de dominio este fuera de servició)
      - Contraseñas de las cuentas de servicios Windows configurados en la máquina.
      - Contraseñas de las cuentas para las tareas programadas.
      - Contraseñas de applicaciones IIS, cuentas Microsoft, etc.
  - Base de datos AD DS. La base de datos NTDS.dit contiene las credenciales de todas las cuentas de usuarios y equipos del dominio Active Directory.
  - Administrador de credenciales o Credential Manager store: Permite a los usuarios almacenar credenciales de los navegadores soportados y otras aplicaciones Windows.

[Mas informacion](https://technet.microsoft.com/en-us/library/hh994565(v=ws.11).aspx)

## 4.- Access Tokens

Acess token o token de acceso es un objeto o structura de datos que describe el contexto de seguridad de un proceso o hilo. Cuando un usuario inicia sesión en un sistema Windows, el sistema comprueba que las credenciales son correctas y LSA genera un access token asociado a dicho usuario. Por lo tanto en Windows todo proceso o hilo tiene un access token asignado.

Esta estructura de datos contiene información tal como el SID de usuario, el SID de los grupos a los que el usuario pertenece, los privilegios asignados al usuario o sus grupos, referencia a su sesión de logon asociada para realizar Single Sign-On, etc.

[Información sobre access token](https://msdn.microsoft.com/en-us/library/windows/desktop/aa374909(v=vs.85).aspx)

Este token representará la identidad del usuario que está corriendo dicho proceso, así como sus privilegios, cuando se interactúe con otros objetos asegurados de manera local o en red; dicta entonces lo que se puede hacer y lo que no desde el proceso. Windows compara los idetificadores SID contenidos en la lista de control de acceso o ACL, del objeto asegurado con los identificadores SID en el access token del usuario.

Ejercicio practico:

Comprobar el SID de usuario, grupos y permisos contenidos en un determinado access token asociado a un proceso cmd.exe. [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer)

Este hecho hace ya entrever que cuando un usuario administrador local inicia sesión en una máquina, dos access token separados se crearán para este usuario: uno estandar y otro de administrador. (UAC)

Los access token forman parte del proceso de autenticación automática contra otros sistemas en la red, cada access token contiene la referencia a la sesión logon asociada, la cual se almacena en LASSS y contiene las credenciales necesarias para realizar el inicio automático de sesion.

Ejercicio practico: 

Se dispone de un servidor Active Directory con un cliente Windows 7 con un usuario de dominio llamado "empleado1" el cual no dispone de privilegios especiales. Se intentara acceder al recurso compartido "c$" en el controlador de dominio "DC1".

Que esta pasando? Como podríamos acceder al recurso compartido ? Como podemos listar las sesiones logon disponibles en cada momento ? [logonsessions](https://learn.microsoft.com/en-us/sysinternals/downloads/logonsessions)

Que pasa cuando ejecutamos el comando runas? Se han generado dos nuevas sesiones generadas para los propositos de Single Sign-On del usuario "admin-dominio1".

Que pasa si ejecutamos el comando runas pero con la opción /netonly ? Este parametro indica que la información del nuevo usuario con la que se correrá el comando será utilizada sólo y únicamente para acceder a un objeto remoto en la red y no de manera local. 

Se podrá acceder al recurso c$ ? Que usuario de volvora el comando whomai? 

Dicho proceso se está ejecutando como el usuario "empleado1" y sólo y únicamente utilizará las credenciales de "admin-dominio1" cuando sea necesario interactuar con un objeto de la red en remoto.

Este último ejemplo muestra claramente que en un access token, el usuario referenciado por su campo SID de usuario no necesariamente debe concidir con el usuario al que pertence la sesión logon referenciada en el mismo. (Posible pregunta)

### Robo y suplantación de tokens

¿ Por que suplantar a otro usuario puede ser útil ?
  - Ayudar a un atacante a encubrir sus acciones al hacerse pasar por otro usuario.
  - Permitir una posible escalada de privilegios local y/o en el dominio.

[Información de la herramienta para el robo de tokens Incognito](https://labs.mwrinfosecurity.com/tools/incognito/)
[Incognito Github](https://github.com/FSecureLABS/incognito)

## 5.- Control de Cuentas de Usuario (UAC)

UAC del inglés User Account Control, es una tecnología desarrollada para seguridad de los sistemas Windows. UAC se creó con la idea de aplicar de una manera sencilla el concepto de mínimo privilegio posible. Windows limita los permisos y las acciones que un usuario o proceso puede llevar a cabo hasta que un administrador autoriza una elevación. 

Sin permisos administrativos, los usuarios no pueden modificar configuraciones críticas del sistema, como obtener una shell privilegiada, modificar la configuración del cortafuegos, combiar el comportamiento del antivirus, obtener la base de datos SAM, etc.

De este modo, se puede incluso estar trabajando con una cuenta con privilegios de administrador y, sin embargo, las aplicaciones que corren bajo esta cuenta no heredan dichos privilegios adminsitravos a no ser que así sea autorizado explícitamente.

Esto se consigue al tener dos tips de access tokens. El usuario administrador puede ejecutar procesos que requieran permisos adminstrativos realizando una elevación de UAC, por ejemplo, al lanzar el proceso haciendo uso de la opción "Ejecutar como administrador".

Comando whoami /priv para ver los permisos de los access tokens.

¿ Cómo consigue Windows diferenciar y aislar los diferentes procesos con diferentes permisos ejecutando bajo el mismo usuario ?

UAC hace uso del control de integridad o MIC, del término inglés Mandatory Integrity Control.

Cada proceso pose un nivel de integridad asociado. De este modo, un objeto del sistema con un nivel de integridad menor no podrá acceder a otro objeto con un nivel de integridad mayor.


En algunas ocasiones, y dependiendo de la configuración de UAC establecida , los programas del sistema se elevan automáticamente si el usuario pertenece al grupo administrador. Estos binarios se pueden identificar observando su archivo Manifest. Sí éste tiene la opción autoElevate con valor igual a True, significa que no hay necesidad de pedir la elevación de privilegios, se elevará a cabo la elevación sin solicitar la confirmación.

## 6.- Bypass UAC

Las técnicas de bypass UAC permiten a un atacante saltarse el proceso de elevación de UAC y conseguir elevar un proceso sin permisos de adminstrador. La idea fundamental en la que se basa la gran mayoria de estas técnicas es la de encontrar binarios  que tenga activado autoElevate y aprovechar estos privilegios para ejecutar programas con integridad alta.

[Mas información sobre los binariso](https://technet.microsoft.com/en-us/library/2009.07.uac.aspx)

### Bypass UAC mediante CompMgmtLauncher 

https://xthalach.github.io/blog/BypassUACCompMgmtLauncher/

### Bypass UAC mediante sdclt.exe

https://xthalach.github.io/blog/BypassUACAppPaths

### Bypass UAC mediante eventvwr.exe 

https://xthalach.github.io/blog/BypassUACEventViwer






