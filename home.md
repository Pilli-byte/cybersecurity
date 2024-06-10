# Progetto cybersecurity: LAMP security CTF4

## Preparazione del progetto
Ho utilizzato una macchina virtuale contentente KaliLinux per effettuare l'attacco mentre per il server vittima ho utilizzato il sito web root-me.org che permette di praticare attacchi su macchine vulnerabili.
Threat model: connessione col server.

## Strumenti utilizzati
* Host
* Nmap
* sqlmap
* _jhonn the ripper_ (Non utilizzato in quanto sqlmap aveva una funzione integrata)
* Putty

## Soluzione
### Reconnaissance
Una volta attivata la macchina virtuale mi è stato fornito un indirizzo del server. Per incominciare ho recuperato l'indirizzo ip (anche se non necessario per l'esecuzione dell'esercizio) ed eseguito
uno scan su tutte le porte per vedere quali server erano in esequzione. Ho trovato tre server in ascolto, tra cui un web server e un ssh server. Inoltre nel web server Nmap ho trovato la presenza del file robots.txt, utilizzato principalmente dai motori di ricerca per l'indicizzazione delle pagine, ma può essere utile per trovare pagine non direttamente accessibili da sito.

![UX Design Process/Toolkit](images/1.png)

Nel file robots.txt c'erano cinque pagine, tre richiedevano le credenziali per entrare: mail,admin,restricted. La pagine conf dava un errore di apache mentre sql conteneva le informazioni sulle tabelle del databese del sito tra cui la tabella user con le password.

![UX Design Process/Toolkit](images/2.png)

Una volta scoperto la presenza di un database ho cercato il modo di accederci. Ho notato due cose:
1) Le richieste avvenivano in GET
2) Nella sezione blog quando si entrava per leggere un articolo compariva nell'indirizzo il paramentro id.

Così ho cercato di rompere la query aggiungendo ' per triggerare un errore, in caso affermativo questo implica la possibiltà di attaccare il server tramite sql injection.

![UX Design Process/Toolkit](images/3.png)

Prima di di procedere in questa direzione però ho notato la presenza di una barra di ricerca, inserendo un script all'interno ho notato la presenza della vulnerabilià XSS. Dato che l'obiettivo è quello di entrare nel server ho abbandonato questa strada in quanto l'uso comune degli attacchi XSS prevede come vittima gli altri utenti che usano il server.

![UX Design Process/Toolkit](images/4.png)

## Credential access

Una volta constatato la presenza della vulnerabilità alle sql injection, ho utilizzato sqlmap per cercare di ottenere le credenziali degli utenti che avevo visto nella fase Reconnaissance. I comandi usati sono stati questi:

1) sqlmap -u "ctf04.root-me.org/index.html?page=blog&title=Blog&id=2" --dbs

2) sqlmap -u "ctf04.root-me.org/index.html?page=blog&title=Blog&id=2" -D ehks --tables

3) sqlmap -u "ctf04.root-me.org/index.html?page=blog&title=Blog&id=2" -D ehks -T user --dump

Il primo comando è riuscito a darmi la lista dei database presenti, mentre il secondo mi ha dato le tabelle presenti nel database ehks. Fino ad adesso nulla di nuovo in quanto queste informazioni le avevo già ottenute. Il terzo comando invece mi ha permesso di scaricare il conenuto della tabella user così facendo ho ottenuto guessing material. La continuazione dovrebbe essere di utilizzare jhon the ripper per cercare di indovinare gli hash delle password, ma mi è stata offerta la possibilità tramite sqlmap e così ho fatto.

![UX Design Process/Toolkit](images/5.png)

Una volta ottenute le credenziali dei vari utenti è stato possibili accedere alla pagina admin. Per dimostrarlo ho aggiunto un post al blog

![UX Design Process/Toolkit](images/6.png)

Gli account del sito non permettono di fare altro però potrebbe essere che abbiano usato le stesse credenziali per la mail, così sono riuscito ad entrare all'interno della mail di un account. A parte ottenere informazioni personali non sono riuscito a ottenere altro.

![UX Design Process/Toolkit](images/7.png)

Il report di Nmap indicava la presenza di un server ssh quindi ho provato usando sempre le stesse credenzali per provare a connetrmi al server, ha funzionato. Il Privilege Escalation è stato abbastanza 
facile in quanto l'utente che ho impersonato faceva parte del gruppo degli admin. Quindi mi bastato inserire il comando "sudo su" e ripetere la stessa password usata per l'accesso ssh per ottenre il privilegio di root. Ho usato putty in quanto essendo un server vecchio, il comando da CLI non funzionava a causa di algoritmi di cifratura absoleti.

![UX Design Process/Toolkit](images/8.png)

## Link consulati per lo svolgimento del progetto

https://chousensha.github.io/blog/2016/04/19/pentest-lab-lampsecurity-ctf4/

https://syrion89.wordpress.com/2016/12/20/lamp-security-ctf4/

https://www.youtube.com/watch?v=J2aZMSJeSm0&t=592s


