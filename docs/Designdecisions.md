---
layout: page-with-side-nav
title: Ontwikkelversie Open Raads- en Stateninformatie
---

# Design decisions Open Raads- en Stateninformatie

In de onderliggende Design Decisions zijn de inzichten vastgelegd die zijn opgedaan bij het ontwerpen en specificeren van API-specificatie voor Open Raads- en Stateninformatie.
Mogelijkerwijs zijn deze design desicions onderhavig aan wijzigingen vanwege voortschrijdend inzicht. Dat zal gedocumenteerd worden.

In de releasenotes wordt dan aangegeven of de wijziging breaking is of niet (wat ook gevolgen kan hebben voor het versienummer).
Daarmee kunnen nieuwe API's en API's in ontwikkeling optimaal bediend worden.

## Algemene design decisions

### 1 Landelijke API-strategie

Bij het ontwerpen van de API hebben we ons geconformeerd aan de [Designrules van de de Landelijke API-strategie](https://docs.geostandaarden.nl/api/API-Designrules/).
Waar mogelijk en van toepassing hebben we ons ook geconformeerd aan de [Api Designrules Extensions van de de Landelijke API-strategie](https://geonovum.github.io/KP-APIs/API-strategie-extensies/)

### 2 VNG Design rules

We voldoen ook aan de [VNG Designrules](https://github.com/VNG-Realisatie/API-Kennisbank/tree/master/Design%20rules/readme.md)
Hier worden landelijke designrules en/of extensions aangescherpt voor APA-specificaties die onder regie van VNG worden ontworpen.

## Specifieke desin Decisions

### 1 Modelleerkeuze voor resources "Vergadering" en "Agendapunt"

Resources "Vergadering" en "Agendapunt" corresponderen 1 op 1 met objecttypes uit het informatiemodel.

_**Ratio : **_ Aangezien de er geen duidelijke User-stories vanuit gebruikersperspectief zijn opgesteld is ervoor gekozen om de resources vorm te geven op basis van het informatiemodel.

### 2 Modelleerkeuze voor resource "Informatieobject"

De subtypes die in het informatiemodel onderkent zijn van het supertype "Informatieobject" zijn in het technische model gedenormaliseerd. De eigenschappen zijn opgenomen in de resource "Informatieobject" en er ise een property "InformatieObjectType" toegevoegd.

_**Ratio : **_ Deze wijze van technisch ontsluiten sluit beter aan bij het gebruik van deze gegevens dan het maken van allemaal aparte resources per subtype.
