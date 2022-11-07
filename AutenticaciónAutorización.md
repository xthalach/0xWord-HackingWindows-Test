Autenticación y autorización en Windows

1.- Introducción
Que es autenticación ? Es el proceso de verificar la identidad de un usuario. Este proceso se utiliza para asegurar que el usuario es realmente quien dice ser.
[Mas información](https://technet.microsoft.com/en-us/library/dn751047(v=ws.11).aspx)

Los sistemas Windows requieren que todo usuario se autentique a la hora de acceder a un recurso tanto local como en red.
Windows necesitara comprobar las credenciales para saber si se tienen los permisos necesarios para el acceso.

Por defecto, los sistemas Windows, las credenciales se validan en local contra la base de datos Security Account Manager (SAM) o contra un controlador de dominio en caso de tratase 
de una maquina unida a un dominio Active Directory. Este proceso se llevará a cabo mediante el servicio Winlogon.

2.- Windows Logon
Logon es el responsable de recoger las credenciales y proporcionar la autenticación de usuario para posteriormente comprobar si se dispone de los permisos necesarios
para realizar la acción que se desea frente a un recurso. [Mas información](https://technet.microsoft.com/en-us/library/cc780332(v=ws.10).aspx)

Distintos escenarios de inicio de sesión:

  - Inicio de sesión interactivo: Este es el proceso llevado acabo cuando un usuario introduce sus credenciales en ventana de diálogo del tipo "Iniciar sesión en Windows" al inicio del equipo.
  - Inicio de sesión de red: Con las credenciales ya validadas, el inicio de sesión en red se lleva a cabo cuando el usuario o servicio desea acceder a algún recurso en red y se necesita validar si dispone de privilegios para tal. Generalmente ocurre de manera transparente al usuario. Para poder llevar a cabo este proceso, Windows necesita incluir diversos mecanismos de autenticación tales como Kerberos, certificados de clave publica, Secure Socket Layer/Transport Layer Security (SSL/TLS), Digest o NTLM.
  - Inicio de sesión mediante tarjetas inteligentes o Smart card logon: Se debe disponer de una tarjeta inteligente y su PIN asociado, asi como un lector.
  - Inicio de sesión biométrico: Se acepta este tipo de autenticación donde una característica biométrica como la huella dactilar.

3.- Autenticación y procesamiento de credenciales
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




