---

copyright:
  years: 2017
lastupdated: "2017-07-19"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Protokolle herunterladen
{: #downloading_logs}

Sie können Protokolle in eine lokale Datei herunterladen oder Daten über eine Pipe an ein anderes Programm umleiten. Sie laden Protokolle im Kontext einer Sitzung herunter. Eine Sitzung gibt an, welche Protokolle heruntergeladen werden. Wenn der Download der Protokolle unterbrochen wird, ermöglicht die Sitzung die Fortsetzung des Downloads an der Stelle, an der er unterbrochen wurde. Wenn der Download abgeschlossen ist, müssen Sie die Sitzung löschen.
{:shortdesc}

Führen Sie die folgenden Schritte aus, um Protokolldaten aus einem {{site.data.keyword.Bluemix_notm}}-Bereich in eine lokale Datei herunterzuladen:

Bevor Sie beginnen, melden Sie sich unter {{site.data.keyword.Bluemix_notm}} an einer Region, einer Organisation und einem Bereich an. 

Führen Sie zum Beispiel den folgenden Befehl aus, um sich an der Region 'USA (Süden)' anzumelden: 
	
```
cf login -a https://api.ng.bluemix.net
```
{: codeblock}

## Schritt 1: Verfügbare Protokolle ermitteln
{: #step1}

1. Verwenden Sie den Befehl `cf logging status`, um zu ermitteln, welche Protokolle für die letzten beiden Wochen zur Verfügung stehen. Führen Sie den folgenden Befehl aus:

    ```
    $ cf logging status
    ```
    {: codeblock}
    
    Dieser Befehl stellt beispielsweise folgende Ausgabe bereit:
    
    ```
    +------------+--------+-------+--------------------+------------+
    |    DATE    |  COUNT | SIZE  |       TYPES        | SEARCHABLE |
    +------------+--------+-------+--------------------+------------+
    | 2017-05-24 |    16  | 3020  |        log         |   None     |
    +------------+--------+-------+--------------------+------------+
    | 2017-05-25 |   1224 | 76115 | linux_syslog,log   |    All     |
    +------------+--------+-------+--------------------+------------+
    ```
    {: screen}

    **Hinweis:** Der {{site.data.keyword.loganalysisshort}}-Service ist ein globaler Service, der die koordinierte Weltzeit (UTC) verwendet. Tage sind als UTC-Tage definiert. Um Protokolle für einen bestimmten Tag in Ortszeit anzufordern, müssen Sie möglicherweise mehrere UTC-Tage herunterladen.


## Schritt 2: Sitzung erstellen
{: #step2}

Eine Sitzung ist erforderlich, um den Umfang der Protokolldaten zu definieren, die für einen Download verfügbar sind, und um den Status des Downloads zu behalten. 

Verwenden Sie den Befehl [cf logging session create](/docs/services/CloudLogAnalysis/reference/logging_cli.html#session_create) zum Erstellen einer Sitzung. Optional können Sie das Startdatum, das Enddatum und Protokolltypen angeben, wenn Sie eine Sitzung erstellen:  

* Wenn Sie das Startdatum und das Enddatum angeben, ermöglicht die Sitzung den Zugriff auf die Protokolle für diesen Datumsbereich. 
* Wenn Sie den Protokolltyp angeben (**-t**), ermöglicht die Sitzung den Zugriff auf einen bestimmten Protokolltyp. Dies ist ein wichtiges Feature, wenn Sie viele Protokolle verwalten müssen, da Sie den Geltungsbereich auf die Protokolle eingrenzen können, die im betreffenden Fall relevant sind.

Um eine Sitzung zu erstellen, die verwendet wird, um Protokolle des Typs *log* herunterzuladen, führen Sie den folgenden Befehl aus:

```
cf logging session create -t log
```
{: codeblock}

Die Sitzung gibt die folgenden Informationen zurück:

* Den Datumsbereich für die herunterzuladenden Daten. Standardmäßig wird das aktuelle UTC-Datum verwendet.
* Die Protokolldateien, die heruntergeladen werden sollen. Standardmäßig umfasst dies alle Protokolltypen, die für den Zeitraum verfügbar sind, den Sie beim Erstellen der Sitzung angegeben haben. 
* Informationen dazu, ob das gesamte Konto oder nur der aktuelle Bereich berücksichtigt werden soll. Standardmäßig erhalten Sie Protokolle in dem Bereich, in dem Sie angemeldet sind.
* Die Sitzungs-ID, die erforderlich ist, um Protokolle herunterladen.

Beispiel:

```
$ cf logging session create -t log     
+--------------+--------------------------------------+
|     NAME     |                VALUE                 |
+--------------+--------------------------------------+
| Access-Time  | 2017-05-25T18:04:21.743792338Z       |
| Create-Time  | 2017-05-25T18:04:21.743792338Z       |
| Date-Range   | {                                    |
|              |  "End": "2017-05-25T00:00:00Z",      |
|              |  "Start": "2017-05-25T00:00:00Z"     |
|              | }                                    |
| Id           | -jshdjsunelsssr4566722==             |
| Space        | fdgrghg3-b090-4567-vvfg-afbc436902a3 |
| Type-Account | {                                    |
|              |  "Type": "log"                       |
|              | }                                    |
| Username     | xxx@yyy.com                          |
+--------------+--------------------------------------+
```
{: screen}

**Tipp:** Um die Liste aktiver Sitzungen anzuzeigen, führen Sie den Befehl [cf logging session list](/docs/services/CloudLogAnalysis/reference/logging_cli.html#session_list) aus.

## Schritt 3: Protokolldaten in eine Datei herunterladen
{: #step3}

Führen Sie den folgenden Befehl aus, um die Protokolle herunterzuladen, die durch die Sitzungsparameter angegeben sind:

```
cf logging download -o Protokolldateiname Sitzungs-ID
```
{: codeblock}

Dabei gilt:

* 'Protokolldateiname' ist der Name der Ausgabedatei.
* 'Sitzungs-ID' ist die GUID der Sitzung. Sie haben diesen Wert im vorherigen Schritt erhalten.

Beispiel:

```
cf logging download -o helloLogs.gz -jshdjsunelsssr4566722==
 160.00 KB / 380.33 KB [==============>------------------------]  42.07% 20.99 KB/s 10s
```
{: screen}

Der Fortschrittsanzeiger zeigt den jeweils aktuellen Downloadstatus auf einer Skala von 0 bis 100 Prozent.

**Hinweis:** 

* Das Format der heruntergeladenen Daten ist das komprimierte JSON-Format. Wenn Sie die Datei mit der Erweiterung .gz dekomprimieren und öffnen, können Sie die Daten im JSON-Format sehen. 
* Das komprimierte JSON-Format eignet sich für das Einpflegen per Elasticsearch oder Logstash. Wenn -o nicht angegeben wird, werden die Daten in die Standardausgabe gestreamt, sodass Sie sie direkt in Ihren eigenen ELK-Stack umleiten können.
* Sie können die Daten auch mit einem Programm verarbeiten, das JSON-Parsing ermöglicht. 

## Schritt 4: Sitzung löschen

Wenn der Download abgeschlossen ist, müssen Sie die Sitzung mit dem Befehl [cf logging session delete](/docs/services/CloudLogAnalysis/reference/logging_cli.html#session_delete) löschen. 

Führen Sie den folgenden Befehl aus, um eine Sitzung zu löschen:

```
cf logging session delete Sitzungs-ID
```
{: codeblock}

Dabei ist 'Sitzungs-ID' die GUID der Sitzung, die Sie im vorherigen Schritt erstellt haben.

Beispiel:

```
cf logging session delete -jshdjsunelsssr4566722==
+---------+------------------------+
|  NAME   |         VALUE          |
+---------+------------------------+
| Message | Delete session success |
+---------+------------------------+
```
{: screen}




