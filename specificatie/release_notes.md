# Wat is nieuw ten opzichte van 1.2.0?

## Aanpassingen informatiemodel

### Toevoegingen voor de Waterschappen
Het informatiemodel is uitgebreid zodat de waterschappen ook gebruik kunnen maken van de ORI API (<a href='https://github.com/VNG-Realisatie/ODS-Open-Raadsinformatie/issues/89'>issue #89</a>).

- Enumeratiesoort "functie" is uitgebreid met de volgende waarden:
    * Dijkgraaf
    * Dagelijks bestuurlid
    * Algemeen vestuurslid
    * Secretarisdirecteur

- Enumeratiesoort "rolNaam" is uitgebreid met de volgende waarden:
    * Dagelijks bestuurslid
    * Algemeen bestuurslid

- Enumeratiesoort "vergaderingsType" is uitgebreid met de volgende waarden:
    * Algemene bestuursvergadering

- Enumeratiesoort "naamDagelijksBestuursType" is uitgebreid met de volgende waarden:
    * Dagelijksbestuur

- Aan Objecttypen "Vergadering" en "Fractie" zijn de volgende attributen toegevoegd:
    * Gemeente,
    * Provincie,
    * Waterschap.

### Allerhande
- Alle Objecttypen en Relatieklassen zijn voorzien van een generieke idendifier "ID". Bestaande specifieke identifiers zijn verwijderd.
- Attribuut 'Griffier' toegevoegd aan Enumeratiesoort "rolNaam"" (issue #88).
- Attribuut 'Volledige naam' toegevoegd aan Gegevensgroeptype "Naam" (issue #84).
- Attribuut 'Publicatiedatum' toegevoegd aan Objecttype "Vergadering" (issue #78).
- Objecttype "Betrokkene" hernoemd naar "Indiener".
- Objecttype "Zaak" is gewijzigd in een Gegevensgroeptype.

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
- "is ingediend" tussen Informatieobject en Indiener.
- "heeft als sub-amendement" tussen Amendement en zichzelf (voorheen:“is subAmendement heeft betrekking op”)
- "hoort bij" tussen Amendement en Voorstel.
- "hoort bij" tussen Antwoord en Vraag (voorheen: "behorend bij").
- "leidt tot" tussen Stemming en Besluit (issue #88).
- "heeft betrekking op" tussen Stemming en Agendapunt (issue #87).
- "heeft betrekking op" tussen Stemming en Besluitvormingsstuk (issue #87).


## OAS
- De syntax van de berichten gebaseert op Plain JSON in plaats van HAL.
- Consequente verwijzing en identificatie van resources doormiddel van de attributen "url" en "id".
- Dubbele voorkomens beschrijvingen in OAS verwijderd (issue #86).
- Alle resources voorzien van dezelfde HTTP-operaties (GET, POST, PUT, DELETE).
- PUT-operaties gedefinieerd op op basis van path-paramenter "id".
