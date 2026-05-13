# Guia: Unió de Zorin OS a un domini Active Directory

## 1. Crear la unitat organitzativa "Linux" al servidor AD
Al Windows Server, dins de l'AD, creem una OU anomenada "Linux" per tenir els equips Linux separats.

![Descripció de la foto](/UD9/IMG/1.png)

Marcam "Protect from accidental deletion" per no esborrar-la sense voler. Això ajuda a organitzar millor el domini.

## 2. Comprovar que l'OU i els usuaris estan creats
Veiem que l'OU "Linux" apareix a l'AD i que tenim usuaris com linux_user.

![Descripció de la foto](/UD9/IMG/2.png)

Sense aquesta estructura, després no podríem unir el client ni gestionar permisos.

## 3. Configurar la xarxa al client Zorin
Posam l'IP i el DNS manualment o comprovam que apunti al DC.

![Descripció de la foto](/UD9/IMG/3.png)

El DNS ha de ser 10.0.2.15 (IP del controlador de domini). Si no, el client no trobarà el domini.

## 4. Canviar el nom de l'equip
Assignam un nom complet al client perquè es vegi dins del domini.

```bash
sudo hostnamectl set-hostname foodlogistic13.test
hostname -f
```

![Descripció de la foto](/UD9/IMG/4.png)

Això surt foodlogistic13.test. El nom ha de ser únic.

## 5. Sincronitzar data i hora
La data ha de coincidir amb la del DC, perquè l'autenticació Kerberos falla si hi ha diferència.

```bash
date
```

![Descripció de la foto](/UD9/IMG/5.png)

Veim que la data està correcta.

## 6. Revisar la zona horària
Ens assegurarem que la zona horària és Madrid/París.

![Descripció de la foto](/UD9/IMG/6.png)

Així l'horari d'estiu no causa problemes.

## 7. Descobrir el domini des del client
Abans d'unir-nos, comprovem que el client veu el domini.

```bash
realm discover foodlogistic13.test
```

![Descripció de la foto](/UD9/IMG/7.png)

Ens surt informació com el tipus Kerberos, que el servidor és Active Directory i quins paquets necessitem.

## 8. Unir el client al domini
Ara sí, ens unim amb l'usuari administrador del domini.

```bash
sudo realm join foodlogistic13.test
```
(Introdueixo la contrasenya de l'Administrator)

![Descripció de la foto](/UD9/IMG/8.png)

Si no dona errors, ja som dins del domini.

## 9. Verificar la unió des del servidor AD
Al Windows Server, veiem que l'equip USUARI apareix amb el nom complet usuari.foodlogistic13.test.

![Descripció de la foto](/UD9/IMG/9.png)

Això confirma que el client està registrat al domini.

## 10. Activar la creació automàtica del home directory
Quan un usuari del domini es connecti, volem que se li creï automàticament la carpeta personal.

```bash
sudo pam-auth-update --enable mkhomedir
```

![Descripció de la foto](/UD9/IMG/10.png)

Sense això, l'usuari no tindria carpeta personal.

## 11. Comprovar l'usuari del domini i la seva carpeta personal
Iniciam sessió com a usuari del domini (linux@foodlogistic13.test) i veiem on som.

```bash
id
pwd
```

![Descripció de la foto](/UD9/IMG/11.png)

L'usuari té UID 481401106 i la seva carpeta personal és /home/linux@foodlogistic13.test. Perfecte.

## 12. Donar permisos sudo als administradors del domini
Editem /etc/sudoers.d/domainadmins per permetre que Linuxadmins pugui fer sudo.

```bash
sudo nano /etc/sudoers.d/domainadmins
```

![Descripció de la foto](/UD9/IMG/12.png)

Afegim la línia %Linuxadmins@foodlogistic13.test ALL=(ALL) ALL. Així els membres d'aquest grup tenen privilegis.

## 13. Provar que funciona sudo
Ens convertim en root amb un usuari del grup Linuxadmins.

```bash
sudo su
```

![Descripció de la foto](/UD9/IMG/13.png)

Si demana contrasenya i després canvia a root, està ben configurat.

## 14. Accedir a recursos compartits per SMB
Obrim l'explorador d'arxius i anem a "Other Locations". Connectem amb smb://dc13/.

![Descripció de la foto](/UD9/IMG/14.png)

(Sé que la captura 14 té un error de commanda, però el que importa és que després funciona).

## 15. Instal·lar llibreria python3-smbc (opcional però útil)
Per si volem fer scripts amb Python per accedir a SMB.

```bash
sudo apt update && sudo apt install python3-smbc
```

![Descripció de la foto](/UD9/IMG/15.png)

Això actualitza els repositoris i instal·la el paquet.

## 16. Veure la carpeta compartida "compartida" del DC
Un cop autenticats amb usuari del domini, podem accedir al recurs compartit.

![Descripció de la foto](/UD9/IMG/16.png)

Aquí ja veiem contingut dins de compartida on dc13.
