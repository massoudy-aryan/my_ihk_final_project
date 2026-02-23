### 1. Informationen	Phase
#### 1.1. Analyse der Pipeline-Struktur und Architektur
In dieser phase habe ich mich uber die pipeline informieert, besonders welche schritee gibt es und was machen diese schritte. ausserdem habe ich die eingabe und ausgabe von jede schritte gesehen. Dies ist entscheidend, weil damit kann man besser die aufbau und die strukture von jede pipeline schrittte versthen. momentan gibt es 25 schritte, manche von diesen schritten sind deaktiviert, weil ihre aufgaben wurden zu einem anderen schirtte gegeben. der rest der schritte laufen nacheindander mithilfe von eimen wrapper shell script.
Die pipeline ist mit nodejs geschrieben und läuft per linux betriebsystem. die strukture bestheht aus vier grob phasen, nämlich data feeding, verarbeitung, bulding und am ende pushen zum cloud. jeder schritte wird mit einem nummerische wert bezeichnet die von 100 beginnt um mit 900 bendedt. jeder schritt gehort zu eine von dem veir schritten, gekenzeichnet durch die anfangszahl.

fur eine ubersichtliche zusammenfassung, sehen Sie bitte das folgedne diagaram:

<div align="center">

```plantuml
@startuml name

title Lifecycle & Vulnerability Data Sources


package "Vulnerability" #fef9e7 {
    rectangle "Security Sources" {
        database "NVD" as nvd
        database "RedHat Security" as rh_sec
        database "GitHub Advisories" as gh_adv
        database "Vendor Advisories\n(Palo Alto, Juniper)" as vendor_adv
    }
}

package "Release & End of Life" #eaf2f8 {
    rectangle "External Feeds" {
        file "XML Data" as xml
        file "JSON Data" as json
        cloud "HTML/Web" as html_cloud
        [GitHub Tags/Releases]
        
        storage "Linux Repos\n(RedHat, Ubuntu)" as linux_repos
        
        package "Package Managers\n(WinGet, Chocolatey)" as pkg_mgrs
    }

    rectangle "Internal Feeds" {
        rectangle "Testing Env. " #fff3e0 {
            [Automation Script] <<WinGet Runner>>
        }
        rectangle "Customer Environment Feed" #e8f5e9 {
            [Versio.io Coverage \n product versions]
        }
    }
}

process "Versio.io Pipeline" <<process>> as pipeline {
    component [106-cve_load-raw-cve-nvd]
    component [110-cve_load-raw-cve-redhat]
    component [120-cve_load-raw-cve-github]
    component [130-cve_load-raw-cve-juniper]
    component [217-productVersions_import_generic]
    component [216-productVersions_import-versions-linux-repo]
    component [CSV List]
    component [950-data-quality-checks]
}

rectangle "Versio.io AI Repository" {
    database "productVersions"
    database "cveRepository"
}

nvd --> [106-cve_load-raw-cve-nvd] : Raw CVE Info.
rh_sec --> [110-cve_load-raw-cve-redhat] : Raw CVE Info.
gh_adv --> [120-cve_load-raw-cve-github]
vendor_adv --> [130-cve_load-raw-cve-juniper] : Raw CVE Info.

' Connectivity - Release & EoL
xml --> [217-productVersions_import_generic]
json --> [217-productVersions_import_generic]
html_cloud --> [217-productVersions_import_generic]
[GitHub Tags/Releases] --> [217-productVersions_import_generic]
linux_repos --> [216-productVersions_import-versions-linux-repo]
pkg_mgrs --> [pipeline]

[Automation Script] --> pipeline : Live Data
[Customer Environment Feed] --> pipeline : Unmapped Versions
[Versio.io AI Repository] --> "950-data-quality-checks": Read DB content
[950-data-quality-checks] --> "Versio.io AI Repository": Write Back to DB
pipeline --> "Versio.io AI Repository" : Store 
@enduml

```
</div>

#### 1.2. Analyse der API-Schnittstelle und Anforderungen
Die API verlangt, dass eine ID nummber, displayName, description soll mitgeschickt werden. ausswerdem soll eine api rate hochsten 5000 pro sekunde soll nicht überschreiben werden. die api ist eine private rest api die uber eine API token erreichbar ist. weil es eine rest api ist, implmentiere ich entweder calls mit put oder post methode, das wird entschieden wenn die anforderugnen an das api call in dem lastenheft defineirt sind.


#### 1.3 Festlegung des standardisierten SQL-Formats

Das system soll eingegebene sql datien lesen, verabieteen und an api schiecken. Das bedutet die sql datiten müssen eine bestimete strukture folgen, damit der system sie verabiten konnen. zum beispiel es soll sicher gestellet werden, dass eine productID, name und herstellername vorhanden sein damit.


#### 1.5 Vorbereitung des Projekt-Lastenhefts
##### 1.5.1.Welche Einschränkungen müssen berücksichtigt werden?
Die bestehenden Datenbankoperationen der Plattform und die allgemeine Verfügbarkeit von Versio.io haben zu jeder Zeit Vorrang. Technisch soll sich die Lösung problemlos in die aktuelle NodeJS-Infrastruktur integrieren und direkt in die vorhandene Pipeline-Struktur eingliedern lassen. Außerdem liegt ein besonderer Fokus auf der Sicherheit, insbesondere bei der geschützten Handhabung von API-Token und Datenbank-Zugangsdaten, um die Integrität der gesamten Umgebung nicht zu kompromittieren. Dies muss durch so eine Methode realisiert werden, dass dadurch keine Sensible-Daten im Klartext im Quellcode stehen.

##### 1.5.2.Welche Einschränkungen müssen berücksichtigt werden?

* Automatische Ausführung von SQL-Queries aus SQL-Dateien, die von QA-Mitarbeitern erstellt werden.
* Verarbeitung von Prüfergebnissen (Versio.io API Konform).
* Gesicherte API-Aufrufe an die Versio.io-Plattform.
* Error-Handling für fehlgeschlagene SQL-Queries und API-Aufrufe.
* Modulare Architektur und Aufgabentrennung für einfache Erweiterung und Wartung.
* Nutzung von NodeJS, MariaDB.
* Integration in die bestehende Versio.io-Pipeline-Struktur.






Wenn die api call durch sind. Versio.io zeigt die erhaltenen calls als instancen mit iheren attributen.
![alt text](image.png)

Jeder Instnace wird mit ihren attributen gezeigt, die von dem System berbeitet wurden und von sql erstellet wurden:
![alt text](image-1.png)

für diese instnacen wird eine policy gruppe erstellet:

![alt text](image-2.png)

dann wird die dataset ausgewählt

![alt text](image-3.png)

dann wird die policy und ihre verfizeirende attribute selektiert:
![alt text](image-7.png)
![alt text](image-4.png)

und dann wird in violations, sie gezeigt:

![alt text](image-5.png)

hier ist die liste:
![alt text](image-6.png)


und wird fur jede davon automatisch ein event erzeugt:
![alt text](image-8.png)



fur diese events kann man notifikation erzugen:

![alt text](image-9.png)


![alt text](image-10.png)

![alt text](image-11.png)

hier wird email genommen:

![alt text](image-12.png)

benachritigunung:
![alt text](image-13.png)