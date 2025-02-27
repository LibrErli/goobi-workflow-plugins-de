# Konfiguration

Nach der Durchführung der Installation des Plugins und der zugehörigen Datenbank muss ebenfalls noch eine Konfiguration des Plugins erfolgen. Diese findet innerhalb der Konfigurationsdatei `plugin_intranda_administration_archive_management.xml` statt. und ist beispielhaft wie folgt aufgebaut:

```markup
<config_plugin>

    <basexUrl>http://localhost:8984/</basexUrl>
    <eadExportFolder>/opt/digiverso/basex/import</eadExportFolder>
    <config>
        <!-- define the name(s) of all archives for the plugin -->
        <archive>*</archive>
        <!-- default title for a new node -->
        <nodeDefaultTitle>Document</nodeDefaultTitle>
        <!-- define metadata fields. All fields are displayed on the UI based on the level and the order within this file.
                - @name: contains the internal name of the field. The value can be used to translate the field in the messages files. The field must start with a letter and can not contain any white spaces.
                - @level: metadata level, allowed values are 1-7:
                    * 1: metadata for Identity Statement Area 
                    * 2: Context Area 
                    * 3: Content and Structure Area
                    * 4: Condition of Access and Use Area
                    * 5: Allied Materials Area
                    * 6: Note Area
                    * 7: Description Control Area
                - @xpath: contains a relative path to the ead value. The root of the xpath is either the <ead> element or the <c> element
                - @xpathType: type of the xpath return value, can be text, attribute, element (default)
                - @repeatable: defines if the field can exist once or multiple times, values can be true/false, default is false
                - @visible: defines if the field is displayed on the UI, values can be true/false, default is true
                - @showField: defines if the field is displayed as input field (true) or badge (false, default), affects only visible metadata
                - @fieldType: defines the type of the input field. Posible values are input (default), textarea, dropdown, multiselect, vocabulary
                - @rulesetName: internal name of the metadata in ruleset. If missing or empty, field is not imported into process metadata
                - @importMetadataInChild: defines if the field is imported or skipped in processes for child elements 
                - @validationType: defines a validation rule, allowed values are unique, required, unique+required, regex, regex+required, regex+unique, regex+unique+required
                - @regularExpression defines a regular expression that gets used for validation type regex
                - validationError: message to display in case of validation errors
                - value: list of possible values for dropdown and multiselect lists
                - vocabulary: name of the vocabulary
                - searchParameter: distinct the vocabulary list by the given condition. Syntax is fieldname=value, field is repeatable
         -->

        <!-- internal fields, not visible on the UI -->
        <metadata xpath="./ead:eadheader[@countryencoding='iso3166-1'][@dateencoding='iso8601'][@langencoding='iso639-2b'][@repositoryencoding='iso15511'][@scriptencoding='iso15924']/ead:eadid/@mainagencycode" xpathType="attribute" name="mainagencycode" level="1" repeatable="false" visible="false"/>
        <metadata xpath="./ead:eadheader/ead:profiledesc/ead:creation/@normal" xpathType="attribute" name="normalcreationdate" level="1" repeatable="false" visible="false"/>
        <metadata xpath="./ead:eadheader/ead:profiledesc/ead:creation" xpathType="element" name="creationdate" level="1" repeatable="false" visible="false"/>
        <metadata xpath="./ead:eadheader/ead:filedesc/ead:titlestmt/ead:titleproper" xpathType="element" name="titlestmt" level="1" repeatable="false" visible="false"/>

        <!--  Identity Statement Area -->
        <metadata xpath="./ead:control/ead:maintenanceagency/ead:agencycode" xpathType="element" name="agencycode" level="1" repeatable="false" fieldType="input"/>
        <metadata xpath="./ead:eadheader/ead:eadid" xpathType="element" name="eadid" level="1" repeatable="false" showField="false" fieldType="input" rulesetName="EADID"/>
        <metadata xpath="./ead:control/ead:recordid" xpathType="element" name="recordid" level="1" repeatable="false" fieldType="input" rulesetName="RecordID"/>
        <metadata xpath="(./ead:archdesc/ead:did/ead:unitid[not(@type)] | ./ead:did/ead:unitid[not(@type)])[1]" xpathType="element" name="unitid" level="1" repeatable="false" showField="false" fieldType="input" rulesetName="UnitID"/>

        <metadata xpath="./ead:did/ead:unitid[@type='Vorl. Nr.']" xpathType="element" name="Number" level="1" repeatable="true" />
        <metadata xpath="./ead:did/ead:unitid[@type='Altsignatur']" xpathType="element" name="Shelfmark" level="1" repeatable="true" rulesetName="shelfmarksource" validationType="unique">
            <validationError>Der Wert wurde an anderer Stelle bereits verwendet</validationError>
        </metadata>
        <metadata xpath="(./ead:archdesc/ead:did/ead:unittitle | ./ead:did/ead:unittitle)[1]" xpathType="element" name="unittitle" level="1" repeatable="false" fieldType="textarea" rulesetName="TitleDocMain" importMetadataInChild="false" />
        <metadata xpath="(./ead:archdesc/ead:did/ead:unitdate | ./ead:did/ead:unitdate)[1]" xpathType="element" name="unitdate" level="1" repeatable="false" rulesetName="PublicationYear" importMetadataInChild="false" regularExpression="\\d{4}" validationType="regex">
            <validationError>Der Wert ist keine vierstellige Jahreszahl</validationError>
        </metadata>
        <metadata xpath="(./ead:archdesc/ead:did/ead:unitdatestructured | ./ead:did/ead:unitdatestructured)[1]" xpathType="element" name="unitdatestructured" level="1" repeatable="false"  rulesetName="DateOfOrigin"/>
        <metadata xpath="(./ead:archdesc/@level | ./@level)[1]" xpathType="attribute" name="descriptionLevel" level="1" repeatable="false" fieldType="dropdown">
            <value>collection</value>
            <value>fonds</value>
            <value>class</value>
            <value>recordgrp</value>
            <value>series</value>
            <value>subfonds</value>
            <value>subgrp</value>
            <value>subseries</value>
            <value>file</value>
            <value>item</value>
            <value>otherlevel</value>
        </metadata>

        <metadata xpath="(./ead:archdesc/ead:did/ead:physdesc | ./ead:did/ead:physdesc)[1]" xpathType="element" name="physdesc" level="1" repeatable="false" rulesetName="SizeSourcePrint" />
        <metadata xpath="(./ead:archdesc/ead:did/ead:physdescstructured | ./ead:did/ead:physdescstructured)[1]" xpathType="element" name="physdescstructured" level="1" repeatable="false" rulesetName="physicalDescriptionExtent" />

        <!-- Context Area -->
        <metadata xpath="(./ead:archdesc/ead:did/ead:origination | ./ead:did/ead:origination)[1]" xpathType="element" name="origination" level="2" repeatable="true" rulesetName="Provenience"/>
        <metadata xpath="(./ead:archdesc/ead:odd/ead:head | ./ead:odd/ead:head)[1]" xpathType="element" name="role" level="2" repeatable="false" fieldType="vocabulary">
            <vocabulary>Rollen</vocabulary>
            <!--<searchParameter>type=visible</searchParameter>-->
        </metadata>
        <metadata xpath="(./ead:archdesc/ead:odd/ead:p | ./ead:odd/ead:p)[1]" xpathType="element" name="person" level="2" repeatable="false"/>

        <metadata xpath="(./ead:archdesc/ead:dsc/ead:bioghist | ./ead:dsc/ead:bioghist)[1]" xpathType="element" name="bioghist" level="2" repeatable="true" fieldType="textarea" rulesetName="BiographicalInformation" />
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:custodhist | ./ead:dsc/ead:custodhist)[1]" xpathType="element" name="custodhist" level="2" repeatable="false" fieldType="textarea" rulesetName="InventoryHistory"/>
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:acqinfo | ./ead:dsc/ead:acqinfo)[1]" xpathType="element" name="acqinfo" level="2" repeatable="false" fieldType="dropdown" rulesetName="AquisitionInformation" >
            <value>value 1</value>
            <value>value 2</value>
            <value>...</value>
        </metadata>

        <!-- Content and Structure Area -->
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:scopecontent | ./ead:dsc/ead:scopecontent)[1]" xpathType="element" name="scopecontent" level="3" repeatable="false" fieldType="textarea" rulesetName="ContentDescription"/>
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:appraisal | ./ead:dsc/ead:appraisal)[1]" xpathType="element" name="appraisal" level="3" repeatable="false" fieldType="textarea" rulesetName="AppraisalInformation"/>
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:accruals | ./ead:dsc/ead:accruals)[1]" xpathType="element" name="accruals" level="3" repeatable="true" fieldType="textarea" rulesetName="Additions"/>
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:arrangement | ./ead:dsc/ead:arrangement)[1]" xpathType="element" name="arrangement" level="3" repeatable="false" fieldType="textarea" rulesetName="Arrangement"/>

        <!-- Condition of Access and Use Area -->
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:accessrestrict | ./ead:dsc/ead:accessrestrict)[1]" xpathType="element" name="accessrestrict" level="4" repeatable="false" fieldType="dropdown" rulesetName="AccessLicenseWork" importMetadataInChild="true">
            <value>open access</value>
            <value>restricted</value>
            <value>required registration</value>
        </metadata>

        <metadata xpath="(./ead:archdesc/ead:dsc/ead:userestrict | ./ead:dsc/ead:userestrict)[1]" xpathType="element" name="userestrict" level="4" repeatable="false" fieldType="dropdown" importMetadataInChild="true" rulesetName="UseRestriction">
            <value>damaged</value>
            <value>good condition</value>
        </metadata>

        <metadata xpath="(./ead:archdesc/ead:did/ead:langmaterial[@label='Language']/ead:language | ./ead:did/ead:langmaterial[@label='Language']/ead:language)[1]" xpathType="element" name="langmaterial" level="4" repeatable="false" fieldType="multiselect" rulesetName="DocLanguage" importMetadataInChild="false">
            <value>eng</value>
            <value>ger</value>
            <value>dut</value>
            <value>fre</value>
            <value>esp</value>
            <value>ita</value>
            <value>lat</value>
            <value>pol</value>
            <value>rus</value>
            <value>swe</value>
        </metadata>

        <metadata xpath="(./ead:archdesc/ead:did/ead:langmaterial[@label='font'] | ./ead:did/ead:langmaterial[@label='font'])[1]" xpathType="element" name="font" level="4" repeatable="false" fieldType="multiselect" rulesetName="FontType" importMetadataInChild="false">
            <value>antiqua</value>
            <value>fracture</value>
            <value>handwritten</value>
            <value>mixed</value>
            <value>no text</value>
        </metadata>

        <metadata xpath="(./ead:archdesc/ead:dsc/ead:phystech | ./ead:dsc/ead:phystech)[1]" xpathType="element" name="phystech" level="4" repeatable="false" fieldType="textarea" rulesetName="PhysTech" />
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:otherfindaid | ./ead:dsc/ead:otherfindaid)[1]" xpathType="element" name="otherfindaid" level="4" repeatable="false" fieldType="textarea" rulesetName="OtherFindAid"/>

        <!-- Allied Materials Area -->
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:originalsloc | ./ead:dsc/ead:originalsloc)[1]" xpathType="element" name="originalsloc" level="5" repeatable="false" fieldType="dropdown" rulesetName="OriginalsLocation">
            <value>value 1</value>
            <value>value 2</value>
        </metadata>
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:altformavail | ./ead:dsc/ead:altformavail)[1]" xpathType="element" name="altformavail" level="5" repeatable="false" fieldType="dropdown" rulesetName="AlternativeFormAvailable">
            <value>value 1</value>
            <value>value 2</value>
        </metadata>
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:relatedmaterial/ead:separatedmaterial | ./ead:dsc/ead:relatedmaterial/ead:separatedmaterial)[1]" xpathType="element" name="separatedmaterial" level="5" repeatable="false" rulesetName="SeparatedMaterial"/>
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:bibliography | ./ead:dsc/ead:bibliography)[1]" xpathType="element" name="bibliography" level="5" repeatable="false" rulesetName="BibliographicCitation"/>

        <!-- Note Area -->
        <metadata xpath="(./ead:archdesc/ead:did/ead:didnote | ./ead:did/ead:didnote)[1]" xpathType="element" name="didnote" level="6" repeatable="false" fieldType="textarea" rulesetName="DidNote"/>
        <metadata xpath="(./ead:archdesc/ead:dsc/ead:odd | ./ead:dsc/ead:odd)[1]" xpathType="element" name="oddnote" level="6" repeatable="false" fieldType="textarea" rulesetName="Odd" />

        <!-- Description Control Area -->
        <metadata xpath="./ead:control/ead:conventiondeclaration" xpathType="element" name="conventiondeclaration" level="7" repeatable="false" fieldType="multiselect" rulesetName="ConventionDeclaration">
            <value>val 1</value>
            <value>val 2</value>
            <value>val 3</value>
            <value>val 4</value>
        </metadata>

        <node name="file" ruleset="File" icon="fa fa-file-text-o" processTemplateId="309544" />
        <node name="folder" ruleset="Folder" icon="fa fa-folder-open-o" processTemplateId="309544"/>
        <node name="image" ruleset="Picture" icon="fa fa-file-image-o" processTemplateId="309544" />
        <node name="audio" ruleset="Audio" icon="fa fa-file-audio-o" processTemplateId="309544"/>
        <node name="video" ruleset="Video" icon="fa fa-file-video-o" processTemplateId="309544"/>
        <node name="other" ruleset="Other" icon="fa fa-file-o" processTemplateId="309544" />
    </config>
</config_plugin>
```

## Allgemeine Konfiguration

Mit den beiden Parametern `<basexUrl>` und `<eadExportFolder>` wird die Anbindung an die BaseX XML Datenbank konfiguriert. Über `basexUrl` wird die URL zur REST API der XML Datenbank konfiguriert, `eadExportFolder` enthält den Ordnernamen, in den das Plugin die EAD Dateien exportiert. Dieser Ordner wird von der XML Datenbank überwacht.

Anschließend folgt ein wiederholbarer `<config>` Block. Über das wiederholbare Element `<archive>` kann festgelegt werden, für welche Dateien der `<config>` Block gelten soll. Soll es einen Default-Block geben, der für alle Dokumente gelten soll, kann `*` genutzt werden.

Mittels `<processTemplateId>` wird festgelegt, auf Basis welcher Produktionsvorlage die erstellten Goobi-Vorgänge erstellt werden sollen.

## Konfiguratioan der Metadatenfelder

Anschließend folgt eine Liste von `<metadata>` Elementen. Darüber wird gesteuert, welche Felder angezeigt werden, importiert werden können, wie sie sich verhalten sollen und ob es Validierungsregeln gibt.

### Pflichtangaben

Jedes Metadatenfeld besteht aus mindestens drei Pflichtangaben:

| Wert | Beschreibung |
| :--- | :--- |
| `name` | Mit diesem Wert wird das Feld identifiziert. Es muss daher eine eindeutige Bezeichnung enthalten. Sofern der Wert nicht noch extra in den messages Dateien konfiguriert wurde, wird er auch als Label des Feldes genutzt. |
| `level` | Hiermit wird definiert, in welchem Bereich das Metadatum eingefügt wird. Dabei sind die Zahlen 1-7 erlaubt: `1. Identifikation`, `2. Kontext`, `3. Inhalt und innere Ordnung`, `4. Zugangs- und Benutzungsbedingungen`, `5. Sachverwandte Unterlagen`, `6. Anmerkungen`, `7. Verzeichnungskontrolle` |
| `xpath` | Definiert einen XPath Ausdruck, der sowohl zum Lesen aus vorhanden EAD Dateien als auch zum Schreiben der EAD Datei genutzt wird. Der Pfad ist im Fall des Hauptelements relativ zum `<ead>` Element, bei allem anderen Knoten immer relativ zum `<c>` Element. |

### Optionale Angaben

Des weiteren gibt es noch eine Reihe weiterer optionaler Angaben:

| Wert | Beschreibung |
| :--- | :--- |
| `@xpathType` | Hiermit wird definiert, ob der XPath Ausdruck ein `element` \(default\), ein `attribute` oder einen `text` zurückliefert. |
| `@visible` | Hierüber kann gesteuert werden, ob das Metadatum in der Maske angezeigt wird oder versteckt ist. Das Feld darf die beiden Werte `true` \(default\) und `false` enthalten. |
| `@repeatable` | Definiert, ob das Feld wiederholbar ist. Das Feld darf die beiden Werte `true` und `false` \(default\) enthalten. |
| `@showField` | Definiert, ob das Feld in der Detailanzeige geöffnet angezeigt wird, auch wenn noch kein Wert vohanden ist. Das Feld darf die beiden Werte `true` und `false` \(default\) enthalten. |
| `@rulesetName` | Hier kann ein Metadatum aus dem Regelsatz angegeben werden. Wenn für den Knoten ein Goobi-Vorgang erstellt wird, wird das konfigurierte Metadatum erstellt. |
| `@importMetadataInChild` | Hierüber kann gestuert werden, ob das Metadatum auch in Goobi-Vorgängen von Kind-Knoten erstellt werden soll. Das Feld darf die beiden Werte `true` und `false` \(default\) enthalten. |
| `@fieldType` | Steuert das Verhalten des Feldes. Möglich sind `input` \(default\) ,  `textarea`,  `dropdown`, `multiselect`, `vocabulary` |
| `value` | Dieses Unterelement wird nur bei den beiden Typen `dropdown` und  `multiselect` genutzt und enthält die möglichen Werte, die zur Auswahl stehen sollen. |
| `vocabulary` | Dieses Unterelement enthält den Namen des zu verwendenden Vokabulars. Es wird nur ausgewertet, wenn `vocabulary` im Typ des Feldes gesetzt ist. Die Auswahlliste enthält jeweils den Hauptwert der Records. |
| `searchParameter` | Mit diesem wiederholbaren Unterfeld können Suchparameter definiert werden, mit denen das Vokabular gefiltert wird, die Syntax ist `fieldname=value`. |
| `@validationType` | Hier kann eingestellt werden, ob das Feld validiert werden soll. Dabei sind drei unterschiedliche Regeln möglich, die sich kombinieren lassen. `unique` prüft, dass der Inhalt des Feldes nicht noch einmal an einer anderen Stelle genutzt wurde, mittels `required` wird sichergestellt, dass das Feld einen Wert enthhält. Mit dem Typ `regex` kann geprüft werden, ob der ausgefüllte Wert einem regulären Ausdruck entspricht. Mit `unique+required`, `regex+required`, `regex+unique` und `regex+unique+required` können mehrere Validierungsegeln angewendet werden. |
| `@regularExpression` | Hier wird der reguläre Ausdruck angegeben, der bei der `regex` Validierung genutzt werden soll. WICHTIG: der Backslash muss dabei durch einen zweiten Backslash maskiert werden. Eine Klasse wird daher nicht durch `\w`, sondern durch `\\w` definiert. |
| `validationError` | Dieses Unterfeld enthält einen Text, der angezeigt wird, wenn der Feldinhalt gegen die Validierungsregeln verstößt. |

## Beispiele für verschiedene Feld-Konfigurationen

### Einfaches Eingabefeld

```markup
<metadata xpath="./ead:control/ead:maintenanceagency/ead:agencycode" xpathType="element" name="agencycode" level="1" repeatable="false" fieldType="input"/>
```

### Textfeld

```markup
<metadata xpath="(./ead:archdesc/ead:did/ead:unittitle | ./ead:did/ead:unittitle)[1]" xpathType="element" name="unittitle" level="1" repeatable="false" fieldType="textarea" rulesetName="TitleDocMain" importMetadataInChild="false" />
```

### Auswahlliste

```markup
<metadata xpath="(./ead:archdesc/@level | ./@level)[1]" xpathType="attribute" name="descriptionLevel" level="1" repeatable="false" fieldType="dropdown">
    <value>collection</value>
    <value>fonds</value>
    <value>class</value>
    <value>recordgrp</value>
    <value>series</value>
    <value>subfonds</value>
    <value>subgrp</value>
    <value>subseries</value>
    <value>file</value>
    <value>item</value>
    <value>otherlevel</value>
</metadata>
```

### Mehrfachauswahl

```markup
<metadata xpath="(./ead:archdesc/ead:did/ead:langmaterial[@label='Language']/ead:language | ./ead:did/ead:langmaterial[@label='Language']/ead:language)[1]" xpathType="element" name="langmaterial" level="4" repeatable="false" fieldType="multiselect" rulesetName="DocLanguage" importMetadataInChild="false">
    <value>eng</value>
    <value>ger</value>
    <value>dut</value>
    <value>fre</value>
    <value>esp</value>
    <value>ita</value>
    <value>lat</value>
    <value>pol</value>
    <value>rus</value>
    <value>swe</value>
</metadata>
```

### Validierung von Datumsangaben

```markup
<metadata xpath="(./ead:archdesc/ead:did/ead:unitdate | ./ead:did/ead:unitdate)[1]" xpathType="element" name="unitdate" level="1" repeatable="false" rulesetName="PublicationYear" importMetadataInChild="false" regularExpression="^([0-9]{4}\\-[0-9]{2}\\-[0-9]{2}|[0-9]{4})(\\s?\\-\s?([0-9]{4}\\-[0-9]{2}\\-[0-9]{2}|[0-9]{4}))?$" validationType="regex">
  <validationError>Der Wert ist keine Datumsangabe. Erlaubte Werte sind entweder Jahreszahlen (YYYY), exakte Datumsangaben (YYYY-MM-DD) oder Zeiträume (YYYY - YYYY, YYYY-MM-DD-YYYY-MM-DD)</validationError>
</metadata>
```

### Anbindung eines kontrollierten Vokabulars

```markup
<metadata xpath="(./ead:archdesc/ead:dsc/ead:acqinfo | ./ead:dsc/ead:acqinfo)[1]" xpathType="element" name="acqinfo" level="2" repeatable="false" fieldType="vocabulary" rulesetName="AquisitionInformation" >
  <vocabulary>Aquisition</vocabulary>
  <searchParameter>type=visible</searchParameter>
  <searchParameter>active=true</searchParameter>
</metadata>
```

