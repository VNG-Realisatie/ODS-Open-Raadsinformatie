---
layout: page-with-side-nav
title: Wat is nieuw ten opzichte van 1.2.0?
---

# Wat is nieuw ten opzichte van 1.2.0?

## Aanpassingen informatiemodel

### Toevoegingen voor de Waterschappen

Het informatiemodel is uitgebreid zodat de waterschappen ook gebruik kunnen maken van de ORI API ([issue #89](https://github.com/VNG-Realisatie/ODS-Open-Raadsinformatie/issues/89)).

- Enumeratiesoort "functie" is uitgebreid met de volgende waarden:
  - Dijkgraaf
  - Dagelijks bestuurslid
  - Algemeen bestuurslid
  - Secretarisdirecteur

- Enumeratiesoort "rolNaam" is uitgebreid met de volgende waarden:
  - Dagelijks bestuurslid
  - Algemeen bestuurslid

- Enumeratiesoort "vergaderingsType" is uitgebreid met de volgende waarden:
  - Algemene bestuursvergadering

- Enumeratiesoort "naamDagelijksBestuursType" is uitgebreid met de volgende waarden:
  - Dagelijksbestuur

- Aan Objecttypen "Vergadering" en "Fractie" zijn de volgende attributen toegevoegd:
  - Gemeente,
  - Provincie,
  - Waterschap.
 
### Wijziging rondom modellering relatie "is ingediend"

- Objecttype "Betrokkene" hernoemd naar "Indiener".
- De relatie tussen de objecttypen Indiener en Informatieobject ("is ingediend door") is omgedraaid en hernoemd naar "heeft ingediend". De kardinaliteit van die relatie is aan de zijde van het objecttype Informatieobject op 1 gezet.
- De kardinaliteit van de relatie tussen de objecttypen Indiener en Natuurlijk Persoon ("is") is aan de zijde van het objecttype Natuurlijk Persoon op 0..1 gezet.
- De kardinaliteit van de relatie tussen de objecttypen Indiener en Organisatorische Eenheid ("is") is aan de zijde van het objecttype Organisatorische Eenheid op 0..1 gezet.

### Allerhande

- Alle Objecttypen en Relatieklassen zijn voorzien van een generieke identifier "ID". Bestaande specifieke identifiers zijn verwijderd.
- Attribuut 'Griffier' toegevoegd aan Enumeratiesoort "rolNaam"" (issue #88).
- Attribuut 'Volledige naam' toegevoegd aan Gegevensgroeptype "Naam" (issue #84).
- Attribuut 'Publicatiedatum' toegevoegd aan Objecttype "Vergadering" (issue #78).
- Objecttype "Zaak" is gewijzigd in een Gegevensgroeptype.
- De attribuutsoort 'Link' is verwijderd uit het objecttype Enkelvoudig Informatieobject, deze was immers als aanwezig in het objecttype Informatieobject.
- De naam van het enumeratiesoort 'naamDagelijksBestuursType' is gewijzigd in 'dagelijksBestuurstype'.

## Volledig dekking informatiemodel

De OAS dekt nu het volledige informatiemodel. De volgende klassen uit het informatiemodel zijn toegevoegd aan de OAS:

- Fractielidmaatschap.
- Fractie.
- Natuurlijk Persoon.
- Stemresultaat Per Fractie.
- Organisatorische Eenheid.
- Indiener (voorheen: Betrokkene)
- Zaak (voorheen: Objecttype, nu is het een Gegevensgroeptype).

De volgende relaties uit het informatiemodel zijn toegevoegd aan de OAS:

- “is genotuleerd in” tussen Agendapunt en Vergadering.
- “is vastgelegd middels” tussen Vergadering en Mediabron.
- “is mede ondertekend door” tussen Natuurlijk Persoon en Besluitvormingsstuk.
- "heeft ingediend" tussen Indiener en Informatieobject.
- "heeft als sub-amendement" tussen Amendement en zichzelf (voorheen:“is subAmendement heeft betrekking op”)
- "hoort bij" tussen Amendement en Voorstel.
- "hoort bij" tussen Antwoord en Vraag (voorheen: "behorend bij").
- "leidt tot" tussen Stemming en Besluit (issue #88).
- "heeft betrekking op" tussen Stemming en Agendapunt (issue #87).
- "heeft betrekking op" tussen Stemming en Besluitvormingsstuk (issue #87).

## OAS

- De syntax van de berichten is gebaseerd op Plain JSON in plaats van HAL.
- Consequente verwijzing en identificatie van resources door middel van de attributen "url" en "id".
- Dubbele voorkomens beschrijvingen in OAS verwijderd (issue #86).
- Alle resources voorzien van dezelfde HTTP-operaties (GET, POST, PUT, DELETE).
- PUT-operaties gedefinieerd op basis van path-parameter "id".
