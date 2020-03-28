## week9-disposition

- Af Sofie Amalie Landt, Amanda Juhl Hansen & Benjamin Kongshaug

### Explain conceptually all the following terms, and how/why they are needed for SSH and TLS/SSL 

#### Symmetric Encryption

-	Cæsar Shift – brugt siden romertiden, og er super simpelt
-	Enigma er en maskine som blev brugt af nazisterne under anden verdenskrig til symmetrisk kryptering
-	Symmetrisk bruger kun 1 key til at kryptere og dekryptere
-	Https bruger symmetrisk kryptering, fordi det er nemmere og derfor hurtigere
-	Bruges derfor ofte til kommunikation, efter at en sikker forbindelse er etableret med asymmetrisk kryptering

#### Asymmetric Encryption

-	RSA kryptering – Anvender primtal som gør det sværere at dekryptere, medmindre man bruger lave primtal
-	Kom første gang på tale i 1870, men der gik ca. 100 år før det blev udviklet
-	Bruger 2 keys (private og public) Public til at kryptere og private til at dekryptere
-	Denne type er tung og tager lang tid, og bruges derfor ikke så tit når først en sikker forbindelse er etableret, men til at skabe en sikker forbindelse ved at udveksle public keys.
-	Bruges bla. Til at etablere SSH-forbindelser
-	Det kan gøres omvendt, men det bruges sjældent, da det giver mere mening at alle kan kryptere, men ikke dekryptere, frem for det modsatte
-	Den omvendte metode anvendes oftest til at lave certifikater

#### Hashing

-	Envejs algoritme – man kan ikke få den originale værdi ud fra en hashværdi
- Generere en værdi eller flere ud fra tekst ve brug af en matematiske funktion
-	Kan bruges til sikkerhed i beskedtransmissionsprocessen, hvis beskeden kun er tiltænkt en specifik modtager
-	Beskytter beskeden mod uønskede ændringer
-	Hashing bruges også som metode til at sortere nøgle værdier i en database tabel
-	Algoritmen skal være tidstagende for at være sikker, og modstå hackere
-	Hashing bruges til at sikker at man har fået den rigtige public key ved SSH-forbindelser

### Explain what it takes to safely log in to an SSH server, without having to provide a password

- Klientens public key skal ligges manuelt ind på serveren i filen ~/.ssh/authorized_keys
-	Første gang klienten så tilgår serveren, vil serverens public key blive sendt til klienten, sammen med den algoritme der bruges til at hashe den
-	Klienten bliver bedt om manuelt at godkende den sendte public key ved at få præsenteret en hashværdi af den, genereret med den sendte algoritme 
-	Denne skal sammenlignes med den hashværdi som serveren har af nøglen
-	Hvis disse stemmer overens, tilføjes public key til en fil der hedder ”known hosts” hos klienten
-	På denne måde kender begge parter hinanden næste gang der logges in, og kan fortage log in uden adgangskode

### Explain the term SSH-tunnel, and provide a practical example for its use
 
- SSH-tunnel vil sige at tilgå serveren gennem en åben SSH port (port 22) til en bruger på serveren som har adgang til de porte som er lukket for udefra kommende forbindelser f.eks. en database forbindelse (port 3306)
-	SSH-tunnels kan derfor bruges til at for adgang til services på tværs af firewalls
-	Praktisk eksempel med forbindelse til databasen (se under ”Connecting to MySQL via an SSH Tunnel”): 
https://docs.google.com/document/d/1L3DTzTv3yR9yM3h843e1LxAmQnHX0p7-A58OJLBBgsA/edit#

### Explain the steps you have to go through to set up a server with MySQL, as secure as possible → 

#### How can we limit the client IP's that can connect

-	Tillad adgang for specifik ip-adresse i firewallen (ufw)
-	sudo ufw allow from <ip address>

#### If set up to allow only localhost and a firewall that deny 3306, can we still connect “safely” from a remote server 

-	Setup af mysql bruger med alle rettigheder:

mysql> CREATE USER 'no_ssl'@'127.0.0.1' identified by 'test';
mysql> GRANT ALL ON example.* TO 'no_ssl'@'127.0.0.1';
mysql> GRANT SELECT ON performance_schema.* TO 'no_ssl'@'127.0.0.1';
mysql>FLUSH PRIVILEGES;

#### Setup server til kun at tillade localhost forbindelse i mysql:

 sudo nano /etc/mysql/my.cnf

Indsæt:
[mysqld]
#Require clients to connect either using SSL
#or through a local socket file
#require_secure_transport = ON
bind-address = 127.0.0.1

-	Luk port 3306:  sudo ufw deny mysql
-	Mysql kan sikkert tilgås gennem en SSH-tunnel via port 22 (SSH) til den oprettede Mysql bruger

#### How to set up an SSL (Secure Socket Layer (https)) connection that anyone can use

sudo nano /etc/mysql/my.cnf  
Change the value for bind-address like this: bind-address = 0.0.0.0
Uncomment this line in the file: require_secure_transport = ON
Restart the server:  sudo systemctl restart mysql
Allow port  3306 in the firewall: sudo ufw allow mysql

-	MySql 8.x generer selv automatisk de nødvendige SSL/TCP-certifikater ved installation
-	Tjek at de er der: sudo find /var/lib/mysql -name '*.pem' -ls

-	Se praktisk eksempel (se under Connecting directly to MySQL with Mandatory SSL)
-	 https://docs.google.com/document/d/1L3DTzTv3yR9yM3h843e1LxAmQnHX0p7-A58OJLBBgsA/edit#

### Demonstrate a client application (Java or whatever you prefer) running on a separate server that access the Database using SSL


