---
layout: page-with-side-nav
title: mini-ORI API
---

# De mini-ORI API

In de mini-ORI API is de functionaliteit van de ORI API beperkt tot de volgende drie resources:

- Agendapunten,
- Informatieobjecten en
- Vergaderingen.

Bovendien zijn de HTTP-operaties beperkt tot bevragingen en zijn alleen de GET-operaties opgenomen.

Met deze kleine maar representatieve subset van de ORI API kunnen de RIS-leveranciers eenvoudig en snel beginnen met het implementeren en beproeven van de API. Gaandeweg zal de set incrementeel uitgebreid worden tot het volledige model.

## Wat is nieuw?

In de mini-ORI API zijn alvast een aantal verbeteringen doorgevoerd:

- Voor het refereren naar gemeenten, provincies en waterschappen wordt nu gebruik gemaakt van de waardlijsten van TOOI (zie [issue #93](https://github.com/VNG-Realisatie/ODS-Open-Raadsinformatie/issues/93)).

- Het attribuut "webpaginaLink" is toegevoegd aan de resources van de API (zie [issue #92](https://github.com/VNG-Realisatie/ODS-Open-Raadsinformatie/issues/92)).

## DE OAS-specificatie

De OAS-specificatie van de mini-ORI API is op drie manieren te bekijken:

- [Redoc](https://redocly.github.io/redoc/?url=https://raw.githubusercontent.com/VNG-Realisatie/ODS-Open-Raadsinformatie/master/docs/mini-ORI-API/openapi.yaml),
- [Swagger](https://petstore.swagger.io/?url=https://raw.githubusercontent.com/VNG-Realisatie/ODS-Open-Raadsinformatie/master/docs/mini-ORI-API/openapi.yaml) en
- [Yaml](https://raw.githubusercontent.com/VNG-Realisatie/ODS-Open-Raadsinformatie/master/docs/mini-ORI-API/openapi.yaml).




