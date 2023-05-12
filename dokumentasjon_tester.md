# Vedlegg A - systemkonfigurasjon

**Innhold:**
1. [Generelt](#1-generelt)  

2. [Test 1](#test1)  
2.1 [Docker Desktop installering](#windows_uten_nett_docker)  
2.2 [Overføre Windows-image](#windows_uten_nett_image)  
2.3 [Overføre Linux-image](#linux-image)  

3. [Test 2](#test2)  
3.1 [Oppsett av de ulike miljøene](#miljø)  
3.2 [Caddy konteinere](#caddy-konteinere)  
3.3 [Endring av konteiner-miljø](#miljø-endring)  
3.4 [Caddy applikasjon](#caddy_app)  
3.5 [Gjennomføring og resultater](#resultater)   

[Dokumentasjon:](#dokumentasjon)  

4. [Oppsett av Linux](#linux)  
4.1 [Sudo rettigheter til Linux brukeren](#sudo-rettigheter)  
4.2 [Oppsett av Docker på Linux](#oppsett-av-docker-på-linux)  

5. [Installere Docker Desktop på Windows](#docker_installasjon)  
6. [Bytte OS-miljø på Docker](#bytte_os)  
7. [Laste ned image](#laste_image)  
8. [Overføre image](#overføre_image)   
9. [Kjøre konteinere](#run)  



<br><br>

# 1. Generelt

Alle maskiner kjører på vSphere, men prosessen med å sette opp virtuelle maskiner er ikke beskrevet da det ikke er fokuset på oppgaven og det ikke skal ha noe å si på forholdene til resultatene. 

<!--  > **MERK:** _I dette labboppsettet vil "vert" og "maskin" bli brukt om maskinene som Docker installeres på._
-->

## Alle maskinene har følgende maskinvare:
RAM: 8 GB \
Hard Disk: 128 (*256) GB \
CPU: 2 \
Kjerner per CPU: 2 \
Alle verts-maskinene har følgende instilling påskrudd: "Exposure hardware assisted virtualization to the guest OS" i vSphere Client, også kalt "Virtualize Intel VT-x/EPT or AMD-V/RVI" I VMware Workstation.

\* Linux-maskinen og Windows-Server har 256 GB, mens alle de andre Windows-maskinene har 128 GB.

<br>
 





<br> <br>

# 2. Test 1 <a id=test1> </a>
Maksinen som ikke er tilkoblet Internett har Windows 10 Pro versjon 22H2 (operativsystembygg 19045.2846).

## 2.1 Docker Desktop installering <a id=windows_uten_nett_docker> </a>
Laster ned Docker Desktop for Windows [(https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe)](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe) fra nettsiden til Docker [(https://docs.docker.com/desktop/install/windows-install/)](https://docs.docker.com/desktop/install/windows-install/). 

Kjører Docker Desktop programmet, og får bedskjed om å oppdatere WSL. Følger WSL guide fra Microsoft [(https://learn.microsoft.com/en-us/windows/wsl/install-manual)](https://learn.microsoft.com/en-us/windows/wsl/install-manual):
1. Laster ned WSL2 [(https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) fra guiden til Microsoft [(https://learn.microsoft.com/en-us/windows/wsl/install-manual)](https://learn.microsoft.com/en-us/windows/wsl/install-manual) på en maskin med Internett.

2. Overfører filen til Windows-maskinen uten Internett med minnepinne, og kjører filen. 

3. Setter WSL 2 som standardversjon av WSL gjennom å kjøre kommandoen:
```shell
wsl --set-default-version 2
```
<br> 

## 2.2 Overføre Windows-image <a id=windows_uten_nett_image> </a>
En maskin med Internett-tilgang vil bli brukt til å laste ned det imaget vi ønsker å overføre til Windows-maskinen uten Internett: 

1. På begge maskinene må man [bytte til](#bytte_os) **Windows-kontinere** for Docker Desktop.
2. På Windows-maskinen med Internett-tilgang [laster man ned](#laste_image) Windows-imaget **hello-world:nanoserver-1809**. 
3. Deretter [overfører](#overføre_image) man imaget til Windows-maskinen uten Internett.

<br>

## 2.3 Overføre Linux-image <a id=linux-image> </a>
En maskin med Internett-tilgang vil bli brukt til å laste ned det imaget vi ønsker å overføre til Windows-maskinen uten Internett: 

1. På begge maskinene må man [bytte til](#bytte_os) **Linux-kontinere** for Docker Desktop.
2. På Windows-maskinen med Internett-tilgang [laster man ned](#laste_image) Windows-imaget **hello-world:linux**. 
3. Deretter [overfører](#overføre_image) man imaget til Windows-maskinen uten Internett.

<br><br>



# 3. Test 2 <a id=test2> </a>

På maskinene som er med i test 2 er det lastet ned en 100 MB binær fil med lenken [https://speed.hetzner.de/100MB.bin](https://speed.hetzner.de/100MB.bin) fra [https://testfiledownload.com/](https://testfiledownload.com/). Denne filen vil fra nå av bli referert til som **100MB.bin**.

Windows-maskinene har [Docker Desktop installert](#docker_installasjon) og [installert](#laste_image) **caddy** imaget med versjon **2.6.4-alpine**. 

Når testene for test 2 ble gjennomført fikk hver maskin en IP i subnettet 10.0.0.0/24 uten Internettilkobling og alle brannmurer var skrudd av.

> **MERK:** _Maskinen som curlet nettsidene var en helt ren Windows-maskin og inneholder bare det som følger med OS-et (Windows 10 PRO versjon 22H2)._ 

<br>

## 3.1 Oppsett av de ulike miljøene <a id=miljø> </a>

Det ble laget ulike testmiljøer for å teste ulike typer konteinere samt en tradisjonell applikasjon. Miljøene og variablene er listet under:


| Miljø             | 1                               | 2                                   | 3                                  | 4                                  | 5                                  | 6                                |
|-------------------|---------------------------------|-------------------------------------|------------------------------------|------------------------------------|------------------------------------|----------------------------------|
| **Applikasjons-form** | Linux-konteiner                 | Linux-konteiner                     | Linux-konteiner                    | Windows-applikasjon                | Windows-konteiner                  | Windows-konteiner                |
| **VertsOS**           | Linux                       | Windows                         | Windows                        | Windows                        | Windows                        | Windows Server               |
| **Virtualisering**    |                -                | Hyper-V                             | WSL 2                              |                  -                 | Hyper-V                            | Hyper-V                          |
| **OS versjon**        | Debian GNU/ Linux 11 (bullseye) | Windows 10 Enterprise versjon 22H2  | Windows 10 Enterprise versjon 22H2 | Windows 10 Enterprise versjon 22H2 | Windows 10 Enterprise versjon 22H2 | Windows Server 2019 versjon 1809 |
| **Docker versjon**    | Versjon 23.0.3                  | Versjon 23.0.5                      | Versjon 23.0.5                     | Versjon 23.0.5                     | Versjon 23.0.5                     | Versjon 23.0.5                   |
| **Image-størrelse**   | 151MB                           | 150.86MB                            | 150.86MB                           | 39,345MB*                          | 4,81GB                             | 4,81GB                           |
| **IP-adresse**        | http://10.0.0.4:8080/100MB.bin  | http://10.0.0.2:8080/100MB.bin      | http://10.0.0.2:8080/100MB.bin     | http://10.0.0.1:8080               | http://10.0.0.2:8080/100MB.bin     | http://10.0.0.5:8080/100MB.bin   |



\* Applikasjon, ikke image

Mange av de resterende underkapittlene i dette kapittelet er oppsett av miljø på de forskjellige maskinene. 


<br>

## 3.2 Caddy konteinere <a id=caddy-konteinere> </a>

Miljø 1, 2, 3, 5 og 6 (alle konteinermiljøene) følger instruksene under for gjennomføringen av test 2:

1. Lager en Dockerfile med følgene innhold:

```shell
FROM caddy:latest \
COPY ./100MB.bin /usr/share/caddy/ 
```

- FROM kommandoen vil hente (PULL) den siste versjonen av caddy fra Docker Hub.
- COPY kommandoen vil legge til 100MB.bin filen inn i imaget i filstrukturen /usr/share/caddy/, som er filplasseringen for filer endebrukeren henter, slik som index.html. Ettersom testene skal curle filen, er det en fornuftig filplassering. 

Kilde: [https://www.docker.com/blog/how-to-use-the-official-nginx-docker-image/](https://www.docker.com/blog/how-to-use-the-official-nginx-docker-image/)

Vi velger å ikke bruke mount volume da det kobler opp konteineren til en filplassering lokalt på maskinen. Dette vil gjøre at konteinerene er mindre portable og kunne skape forsinkelser og gi feilkilder.

2. Vi går inn i samme mappe som Dockerfilen og 100MB.bin ligger, og kjører følgene Docker kommando:
```shell
$ docker build -t webserver .
```
Denne kommandoen bygger en webserver basert på Dockerfilen som ble laget tidligere, altså en Caddy webserver som har 100MB.bin binærfilen i sin filstruktur.

3. Kommando for å starte webserveren:
```shell
$ docker run -d --ip http://10.0.0.x -p 8080:80 webserver
```
Denne kommandoen starter en konteiner basert på imaget "webserver" som ble laget i forrige steg:
- Docker run - Starter konteineren
- -d - kjører konteineren i bakgrunnen
- -p 8080:80 - Setter port 8080 til verten og port 80 til konteineren
- --ip setter statisk IP til webserveren og fjerner TLS slik at det kun blir http


<br>

## 3.3 Endring av konteiner-miljø <a id=miljø-endring> </a>

Under er dokumentasjon for å endre mellom backend moduser som Hyper-V/ WSL 2, isolasjonsmodus og link til vår dokumentasjon på å bytte mellom Windows- og Linux-konteinere.

<br>

### **3.3.1 Hyper-V backend-modus**

Windows- og Linux-konteinere på Windows

1. I instillinger på Docker Desktop under Generelt skru av "Use WSL 2 based engine". 

<!--
https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v
-->

2. Åpne powershell som administrator.
3. Kjør følgene kommando:
```shell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

<br>

#### **3.3.2 Bytting av isolasjonsmodus**
Ved å bruke følgene kommandoer ved kjøring av en konteiner kan man bytte mellom process- og hyper-V-isolation.
```shell
--isolation=process
```
```shell
--isolation=hyperv
```
<!-- 
https://web.archive.org/web/20220922004918/https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container
-->


NB: Windows-server konteinere bruker standard process isolation, og Windows 10 konteinere bruker standard Hyper-V isolation. Fra og med Windows 10 oktober 2018-oppdateringen kan brukere som kjører en Windows 10 Pro eller Enterprise-vert kjøre en Windows-konteiner med process isolation. Brukere må direkte be om process isolation ved å bruke --isolation=process.

<br>

### **3.3.3 WSL2 backend-modus**

Linux-konteinere på Windows

1. I instillinger på Docker Desktop under Generelt skru på "Use WSL 2 based engine"

<br>

### **3.3.4 Hvordan sjekke backend- og isolation-modus**

Følgene kommando viser backend modus: \
```shell
docker info | Select-String -Pattern "OSType"
```
- Windows - Hyper-V
- Linux - WSL 2

Følgene kommando viser isolasjonsmodus: \
```shell
docker info | Select-String -Pattern "Isolation"
```

<br>

### **3.3.5 Switche mellom Windows- og Linux-konteinere**
Se [vår dokumentasjon på bytte mellom konteinertyper](#bytte_os).

<br>

### **3.3.6 Linux-maskinen**
Linux maskinen ble satt opp slik [vår dokumentasjon](#linux) beskriver. 

<!---
5. For å sjekke tiden det tar å hente ned filen 100MB.bin fra Caddy webserveren brukes kommandoen:

(NB: Linux kommando)

$ time curl localhost:8080/100MB.bin --output fil.txt

- time - Tar tiden på henting av 100MB.bin
- --output fil.txt - Legger outputet fra 100MB.bin filen som blir curlet inn i en tekstfil, da det kan bli problemer i terminalvinduet ved printing av binære verdier.

Powershellkommando:

Measure-Command {cmd /c 'curl.exe http://localhost:12345/100MB.bin -o -'}

7. Flere gjenomføringer og resultat i fil - Linux

fileName="results.txt"
for i in {1..10}
do
  { time curl localhost:12345/100MB.bin --output /dev/null 2>&1 ; } 2>> $fileName
done

cat results.txt | grep real | awk '{print substr($2, 3, length($2)-3)}' > final.txt


'''
Containers feature is disabled. Enable it using the PowerShell script (in an administrative PowerShell) and restart your computer before using Docker Desktop: 

Enable-WindowsOptionalFeature -Online -FeatureName $("Microsoft-Hyper-V", "Containers") -All
'''

--->

<br><br>

## 3.4 Caddy applikasjon <a id=caddy_app> </a>
1. Laster ned Caddy for Windows amd64[(https://caddyserver.com/api/download?os=windows&arch=amd64&idempotency=59971736216496)](https://caddyserver.com/api/download?os=windows&arch=amd64&idempotency=59971736216496) fra Caddy sin nettside [(https://caddyserver.com/download)](https://caddyserver.com/download).
- Lager en mappe **C:\caddy**.
- Lagrer nedlastningen av Caddy som **caddy.exe** og flytter den til mappen **C:\caddy**.

1. Setter **caddy** inn som systemvariabel i PATH:
- Høyreklikker på start menyen og velger "System".

- Velger "Avanserte systeminstillinger", går til "Avanset" og trykker på "Miljøvariabler".

- Velger systemvariabelen "Path" og trykker "Rediger". Velger "Ny" og skriver inn **C:\caddy** som er filbanen til caddy.exe. Trykker "ok" til det ikke er flere "ok" å trykke på.

Fra nå av vil kommandoen **caddy** være tilgjengelig i en terminal.

<br>

1. Gjør klar Caddy-serveren:
- Plasserer 100MB.bin filen i mappen **C:\caddy**. 

- Inne i mappen **C:\caddy** legger vi ved en fil **CaddyFile**. I den skriver vi:
```shell
localhost:2017 {
  respond "Hello, world!"
}

http://10.0.0.1:2016 {
  respond "Goodbye, world!"
}

http://10.0.0.1:8080 {
  file_server {
    index 100MB.bin
  }
}
```
> **MERK:** _Det er kun http://10.0.0.1:8080 kode-blokken som er nødvendig å ha i CaddyFile for å gjenomføre testen, men vi hadde med de andre kode-blokkene for å lettere kunne feilsøke ved behov._ 


<br>

Kjører web-serveren ved å kjøre kommandoen:
```shell
caddy run
```

Nå er 100MB.bin filen mulig å laste ned fra http://10.0.0.x:8080 fra en maskin på samme subnett som web-serveren, dersom brannmuren skrus av for maskinene.  

<br>

Dersom CaddyFile endres på, og man ønsker å oppdatere web-serveren etter dette, stopper man først serveren og starter den opp igjen ved å kjøre kommandoene:
```shell
caddy stop
caddy run
```


<br>

## 3.5 Gjennomføring og resultater <a id=resultater> </a>
Maskinen som curlet nettsidene var en helt ren Windows-maskin og inneholder bare det som følger med OS-et (Windows 10 PRO versjon 22H2).

For å kjøre gjennomføre curl 100 antall ganger, samt måle tid den bruker på å curle i en fil, brukes følgene powershellscript:
```shell
$fileName = "resultat.txt" \
$numberOfRuns = 100

for ($i = 1; $i -le $numberOfRuns; $i++) {
    $timeSpan = Measure-Command {cmd /c 'curl.exe http://10.0.0.x/100MB.bin -o -'} \
    $totalSeconds = $timeSpan.TotalSeconds \
    Add-Content -Path $fileName -Value $totalSeconds
}
```
- $fileName - Lager en fil kalt resultat.txt
- $numberOfRuns - Antall ganger den skal curle
- $timeSpan - En variabel for curl og tidsmåling koammdno
- Measure-Coammdnd - Tar tiden det tar å curle
- $totalSeconds - Variabel for hvor mange sekunder curlen tar
- Legger til tiden det tar å curle i filen results.txt

> **MERK:** _For å crule nettsiden til webserveren kjørt fra en caddy **applikasjon** ble url-en **http://10.0.0.x** benyttet i scriptet ovenfor._ 




<br><br>

# Dokumentasjon: <a id=dokumentasjon> </a>

Under er det dokumentasjon som blir referert til i kap. 2 og 3. 

# 4. Oppsett av Linux-maskinen <a id=linux> </a>

> **MERK:** _I den påfølgende teksten til kapittel 2 vil vi bruke "\$"- og "\#"-symbolet for å vise en kommando og hvilken rettighet kommandoen benytter. Disse symbolene skal altså ikke skrives inn som en del av kommandoene. "\$"-symbolet representerer kommandoer skrevet av jb brukeren, mens "\#"-symbolet representerer kommandoer skrevet av root._

Vi kjører Debian GNU/Linux 11.6.0 x86_64 med standardinstillinger.



| Bruker | Privilegie |
| ----------- | ----------- |
| root | super-user |
| jb | super-user |

<br>

## 4.1 Sudo rettigheter til Linux brukeren <a id=sudo-rettigheter> </a>
Av praktiske årsaker gir vi brukeren jb på Linux sudo rettigheter. Under finnes en guide på dette: 

1. I en terminal, kjør koden:
```shell
  $ su
  # sudo visudo
```
Under root i "# User privilege specification" legg til linjen:
```
jb  ALL=(ALL:ALL) ALL
```
og lagre filen.

<br>

Deretter tester man om brukeren har fått sudo rettigheter:
```shell
  # exit
  $ sudo ls -la /root
```
Dersom siste linje printer ut en liste har brukeren sudo rettigheter.

<br>

## 4.2 Oppsett av Docker på Linux <a id=oppsett-av-docker-på-linux> </a>
Installer Docker gjennom et script fra [https://get.docker.com/](https://get.docker.com/) ved å åpne en terminal på Linux-maskinen og kjøre kommandoene: 
```shell
$ sudo apt update && sudo apt upgrade -y && sudo apt install curl -y
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh ./get-docker.sh 
```
Nå er Docker version 23.0.3, build 3e7cbfd installert. Dette sjekkes gjennom å skrive kommandoen:
```shell
$ docker --version
```

> **MERK:** _Skal testen etterprøves, bør samme versjon av Docker lastes ned. Dette er for å skape et likt testmiljø. Sjekk [https://docs.docker.com/engine/install/debian/#install-docker-engine](https://docs.docker.com/engine/install/debian/#install-docker-engine) for guide til dette._

[Laster ned](#laste_image) imaget  **caddy** med versjon **2.6.4-alpine**.


<br><br>

# 5. Installere Docker Desktop på Windows <a id=docker_installasjon> </a>
For installasjon av Docker ble installasjonsguiden til Docker Inc fulgt [(https://docs.docker.com/desktop/install/windows-install/)](https://docs.docker.com/desktop/install/windows-install/). 

Skrur på WSL 2 ved å kjøre kommandoen ```wsl --install``` i Powershell.

Åpner REGEDIT og endrer **HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LxssManager\Start** til **2** og starter maskinen på nytt. 

Åpner Powershell som administrator og kjører ```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux```


Vi installerte Docker Desktop ved å laste ned og kjøre "Docker Desktop Installer.exe" fra 
installasjonsguiden til Docker [(https://docs.docker.com/desktop/install/windows-install/)](https://docs.docker.com/desktop/install/windows-install/).



<br><br>

# 6. Bytte OS-miljø på Docker <a id=bytte_os> </a>
For å administrere images og konteinere som er bygd på forskjellige OS må man bytte til riktig type Docker Daemon. Docker kan kun kjøre en Daemon av gangen, noe som gjør at man blir nødt til å bytte mellom Windows og Linux Daemons for å administrere forskjellige type OS-konteinere. Konteinere som kjører og all data som er lagret vil ikke forsvinne når man bytter Daemon, men vil ikke være mulig å aksessere gjennom Docker Desktop før man bytter til tilhørende Daemon. 

Standard for Docker Desktop på Windows er å kjøre konteinere på WSL, noe som kun Linux-kontinere kan benytte. For å laste opp Windows-images og starte Windows-konteinere på Docker Desktop på en Windows-maskin må man bytte til Windows-konteinere på Docker Desktop. 

<br>

> **MERK:** _Den første gangen man skal bytte til Windows-konteinere er man først nødt til å kjøre kommandoen under i PowerShell med administratorrettigheter og så restarte maskinen:_
```shell
Enable-WindowsOptionalFeature -Online -FeatureName $("Microsoft-Hyper-V", "Containers") –All
```

<br>

For å bytte mellom Windows- og Linux-konteinere finnes det en GUI og en CLI metode:

## GUI:
Høyreklikke på Docker Desktop "tray-ikonet" i oppgavelinjen og trykker på "Switch to Windows containers..." for å bytte fra Linux- til Windows-konteinere eller "Switch to Linux containers..." å bytte fra Windows- til Linux-kontinere. 

<br>

## CLI:
Kjøre kommandoen under i PowerShell med administratorrettigheter: 
```shell
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon
```

<br> <br>

# 7. Laste ned image <a id=laste_image> </a>
For å hente et offentlig image fra Docker Hub på Internett må maskinen være koblet til Internett og ha Docker [installert](#docker_installasjon). 

> **MERK:** _I kommandoene under er \<image> et eksempel på et image navn, og dette ordet må byttes ut med navnet på det imaget du ønsker å hente. \<versjon> må byttes med den spesifikke versjonen til imaget._ 

For å hente nyeste versjon av et image, her kaldt \<image>, kjører man kommandoen under i terminal med administratorrettigheter:
```shell
docker pull <image>
```
eller 
```shell
docker pull <image>:latest
```
<br>

Best practice er å hente nyeste versjon av et image, men dersom man skal kjøre et spesifikt image må man legge til versjon på følgende måte:
```shell
docker pull <image>:<versjon>
```


<br> <br>

# 8. Overføre image <a id=overføre_image> </a>
For å overføre et image fra Internett til en maskin uten Internett må man først hente imaget fra Internett. Dette har vi gjort gjennom en en maskin på Internett. Deretter har vi overført dette imaget til en minnepinne, som vi bruker for å overføre imaget til maskinen uten Internett-tilgang.

Det er viktig at begge maskinene har samme [OS-konteiner miljø](#bytte_os) på Docker for å kunne overføre imaget. 

1. Først må man ha [lastet ned](#laste_image) imaget man ønsker å overføre. 
2. På maskinen imaget er lastet ned på, overfører man imaget til en USB-minnepinne på følgende måte:
- Går til plasseringen vi ønsker å lagre imaget, på USB-minnepinnen, og lagrer imaget med disse kommandoene:
```shell
cd E:
docker save -o <image>.tar <image>
```
Hvor **\<image>** er navnet på imaget du skal lagre. Du ser navnet på images du har ved å kjøre kommandoen ```docker images```.

<br>

3. For å overføre imaget fra minnepinnen til Windows-maskinen uten Internett, kjører man kommandoen under på **Windows-maskinen uten Internett**. **\<filbane>** refererer til filbanen til minnepinnen:
```shell
docker load -i <filbane>\<image>.tar
```

<br><br>

# 9. Kjøre konteinere <a id=run> </a>
Man er nødt til å ha imaget for å kunne kjøre en konteiner basert på imaget. Dette kan gjøres gjennom [Internett](#laste_image) eller ved å [overføre fra en annen maskin](#overføre_image). 

<br>

For å starte en konteiner basert på en image kan man kjøre kommandoen:
```shell
docker run <image>:<versjon>
```
Noen konteinere trenger å starte med noen parametere. 
Det finnes mange parametere i kommandoen, og man får en oversikt over de forskjellige ved å skrive ```docker run --help```.

Dersom noen av konteinerene i testene våres krever parametere så vil det bli spesifisert i oppsettet til testen. 
