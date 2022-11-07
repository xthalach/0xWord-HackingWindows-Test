Autenticación y autorización en Windows

1.- Introducción
Que es autenticación ? Es el proceso de verificar la identidad de un usuario. Este proceso se utiliza para asegurar que el usuario es realmente quien dice ser.
Mas información: https://technet.microsoft.com/en-us/library/dn751047(v=ws.11).aspx

Los sistemas Windows requieren que todo usuario se autentique a la hora de acceder a un recurso tanto local como en red.
Windows necesitara comprobar las credenciales para saber si se tienen los permisos necesarios para el acceso.

Por defecto, los sistemas Windows, las credenciales se validan en local contra la base de datos Security Account Manager (SAM) o contra un controlador de dominio en caso de tratase 
de una maquina unida a un dominio Active Directory. Este proceso se llevará a cabo mediante el servicio Winlogon.

2.- Windows Logon
Logon es el responsable de recoger las credenciales y proporcionar la autenticación de usuario para posteriormente comprobar si se dispone de los permisos necesarios
para realizar la acción que se desea frente a un recurso. (Mas información)[https://technet.microsoft.com/en-us/library/cc780332(v=ws.10).aspx]
