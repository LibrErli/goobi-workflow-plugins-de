---
description: >-
  Dieses Step Plugin erlaubt die Validierung von Bildern unter Nutzung von JHove
  und das Anpassen des Workflows
---

# Tif-Validierung

## Einführung

Diese Plugin dient zur Validierung von Bildern im Format `TIF` innerhalb von definierbaren Verzeichnissen. Die Validierung erfolgt dabei unter Zuhilfenahme der [Open-Source-Softwarebibliothek JHove ](https://jhove.openpreservation.org/)und ist weitreichend konfigurierbar.

| Details |  |
| :--- | :--- |
| Identifier | intranda\_step\_jhove-validation |
| Source code | [https://github.com/intranda/goobi-plugin-step-tif-validation](https://github.com/intranda/goobi-plugin-step-tif-validation) |
| Lizenz | GPL 2.0 oder neuer |
| Kompatibilität | Goobi workflow 2020.07 |
| Dokumentationsdatum | 12.06.2020 |

## Installation

Zur Installation des Plugins muss zunächst die folgende Datei installiert werden:

```bash
/opt/digiverso/goobi/plugins/workflow/plugin_intranda_step_tif-validation.jar
```

Um zu konfigurieren, wie sich das Plugin verhalten soll, können verschiedene Werte in der Konfigurationsdatei angepasst werden. Die zentrale Konfigurationsdatei befindet sich üblicherweise hier:

```bash
/opt/digiverso/goobi/config/plugin_intranda_step_jhove-validation.xml
```

Innerhalb diese Konfigurationsdatei ist unter anderem der Pfad zu der JHove-Konfiguration benannt. Im Falle des unten aufgeführten Beispiels ist dort der folgende Pfad angegeben:

```markup
<jhoveConfiguration>/opt/digiverso/goobi/config/jhove/jhove.conf</jhoveConfiguration>
```

Entsprechend müssen unter diesem Pfad daher auch folgende beiden Dateien installiert werden:

```markup
/opt/digiverso/goobi/config/jhove/jhove.conf
/opt/digiverso/goobi/config/jhove/jhoveConfig.xsd
```

## Konfiguration des Plugins

Die Konfiguration für das Plugin erfolgt innerhalb der zentralen Konfigurationsdatei. Sie sieht beispielhaft wie folgt aus:

```markup
<config_plugin>
    <config>
        <project>*</project>
        <step>*</step>
        <!-- folders to validate, can be multiple one (e.g. master, main etc. -->
        <folder>master</folder>
        <openStepOnError>Scanning</openStepOnError>
        <lockAllStepsBetween>true</lockAllStepsBetween>
        <jhoveConfiguration>/opt/digiverso/goobi/config/jhove/jhove.conf</jhoveConfiguration>
        <namespace uri="http://www.loc.gov/mix/v20" name="mix" />
        <namespace uri="http://hul.harvard.edu/ois/xml/ns/jhove" name="jhove" />
        <!--Counter check -->
        <check>
            <xpath>count(//jhove:repInfo/jhove:format)</xpath>
            <wanted>1.0</wanted>
            <error_message>Check for image format count: Image: "${image}" Wanted value: "${wanted}"\, found value: "${found}".</error_message>
        </check>
        <check>
            <xpath>string(//jhove:repInfo/jhove:format)</xpath>
            <wanted>TIFF</wanted>
            <error_message> Check for image format: Image: "${image}" Wanted value: "${wanted}"\, found value: "${found}".</error_message>
        </check>
        <!--Check if the image is well-formed and valid -->
        <check>
            <xpath>//jhove:repInfo/jhove:status</xpath>
            <wanted>Well-Formed and valid</wanted>
            <error_message> Check for image status: Image: "${image}" Wanted value: "${wanted}"\, found value: "${found}".</error_message>
        </check>
        <!--Check for resolution (number or range) -->
        <integrated_check name="resolution_check">
            <mix_uri>http://www.loc.gov/mix/v20</mix_uri>
            <wanted>100.0-899.23</wanted>
            <error_message> Check for resolution: Image: "${image}" Wanted value: "${wanted}"\, found value: "${found}".</error_message>
        </integrated_check>
    </config>
</config_plugin>
```

Die Parameter innerhalb der zentralen Konfigurationsdatei des Plugins haben folgende Bedeutungen:

| Wert | Beschreibung |
| :--- | :--- |
| `project` | Dieser Parameter legt fest, für welches Projekt der aktuelle Block `<config>` gelten soll. Verwendet wird hierbei der Name des Projektes. Dieser Parameter kann mehrfach pro `<config>` Block vorkommen. |
| `step` | Dieser Parameter steuert, für welche Arbeitsschritte der Block `<config>` gelten soll. Verwendet wird hier der Name des Arbeitsschritts. Dieser Parameter kann mehrfach pro `<config>` Block vorkommen. |
| `folder` | Mit diesem Parameter können Verzeichnisse festgelegt werden, deren Inhalte validiert werden sollen. Dieser Parameter kann wiederholt vorkommen. Mögliche Werte hierfür sind z.B. `master`, `media` oder auch individuelle Ordner wie `photos` und `scans`. |
| `openStepOnError` | Dieser Parameter legt fest, welcher Arbeitsschritt des Workflows erneut geöffnet werden soll, wenn ein Fehler innerhalb der Validierung auftritt. Wird dieser Parameter nicht verwendet, so aktiviert das Plugin stattdessen einfach den vorherigen Arbeitsschritt des Validierungsschritts. |
| `lockAllStepsBetween` | Mit diesem Parameter wird festgelegt, ob die Arbeitsschritte des Workflows zwischen dem Validierungsschritt und demjenigen, der innerhalb des Parameters `openStepOnError` angegeben wurde, wieder auf auf den Status gesperrt gesetzt werden sollen, so dass diese Arbeitsschritte ein erneutes Mal durchlaufen \(`true`\) werden müssen. Wird der Wert hingegen auf `false` gesetzt, so wird der Status der dazwischen liegenden Schritte nicht verändert, so dass die Arbeitsschritte auch nicht noch einmal durchlaufen werden. |
| `jhoveConfiguration` | Mit diesem Parameter wird angegeben, wo sich die Konfigurationsdatei für JHove befindet. |
| `check` | Innerhalb eines jeden Elements check wird festgelegt, was JHove genau validieren soll. Hier wird beispielsweise festgelegt, welches Dateiformat erwartet wird. Zugehörig ist hierbei ebenso, welche Fehlermeldung im Falle einer fehlerhaften Validierung ausgegeben werden soll. |

## Arbeitsweise des Plugins

Das Plugin wird üblicherweise vollautomatisch innerhalb des Workflows ausgeführt. Es ermittelt zunächst, ob sich innerhalb der Konfigurationsdatei ein Block befindet, der für den aktuellen Workflow bzgl. des Projektnamens und Arbeitsschrittes konfiguriert wurde. Wenn dies der Fall ist, werden die weiteren Parameter ausgewertet und die Checks gestartet. Ist einer der konfigurierten Checks nicht erfolgreich, so wird der konfigurierte oder alternativ der vorherige Arbeitsschritt in einen Fehlerstatus versetzt und die Validierungsmeldung in das Vorgangslog geschrieben. Sollen die Arbeitsschritte zwischen dem Validierungsschritt und dem benachrichtigten Schritt im Status auf `geschlossen` gesetzt werden, so sind diese für die Bearbeiter ebenfalls mit der Korrekturmeldung versehen und erlauben so eine Nachvollziehbarkeit des Problemfalls.

## Bedienung des Plugins

Dieses Plugin wird in den Workflow so integriert, dass es automatisch ausgeführt wird. Eine manuelle Interaktion mit dem Plugin ist nicht notwendig. Zur Verwendung innerhalb eines Arbeitsschrittes des Workflows sollte es wie im nachfolgenden Screenshot konfiguriert werden.

![Integration des Plugins in den Workflow](../.gitbook/assets/intranda_step_jhove-validation.png)

