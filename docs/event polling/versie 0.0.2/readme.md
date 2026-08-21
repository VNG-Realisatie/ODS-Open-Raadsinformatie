---
layout: page-with-side-nav
---

# Event polling — concept 0.0.2

Concept-doorontwikkeling van [versie 0.0.1](../versie%200.0.1/readme.md), bedoeld
als gespreksstuk. Versie 0.0.1 blijft ongewijzigd staan, zodat de twee naast
elkaar te vergelijken zijn:

- [Redoc — 0.0.2](https://redocly.github.io/redoc/?url=https://raw.githubusercontent.com/VNG-Realisatie/ODS-Open-Raadsinformatie/feedback/event-polling-sync-contract/docs/event%20polling/versie%200.0.2/openapi.yaml)
- [Redoc — 0.0.1](https://redocly.github.io/redoc/?url=https://raw.githubusercontent.com/VNG-Realisatie/ODS-Open-Raadsinformatie/master/docs/event%20polling/versie%200.0.1/openapi.yaml)

De opzet van 0.0.1 — een afnemer laten synchroniseren op verschillen in plaats
van op volledige herharvests — is de goede richting. Dit voorstel raakt niet aan
dat uitgangspunt; het werkt uit welke afspraken er in het contract nodig zijn om
die belofte waar te maken.

De aanleiding is praktisch. De ophaalkant van deze keten (raadsinformatie
ontsluiten uit iBabs, NotuBiz, GemeenteOplossingen en Parlaeus) en de
uitleverkant (een publieke feed voor hergebruikers) zijn in
[OpenBesluitvorming](https://openbesluitvorming.nl) — de opvolger van
openraadsinformatie.nl — inmiddels in productie gebouwd. Een aantal keuzes
hieronder komt rechtstreeks uit fouten die we daar eerst zelf hebben gemaakt.

## De belangrijkste wijziging: alleen publiceren, niet aanleveren

Versie 0.0.1 beschrijft twee onverenigbare topologieën in één document. Er is een
`POST /events` waarmee een bronhouder aanlevert *bij Open Overheid*, met PID's in
de namespace `openoverheid.nl` — een centraal model. Maar bij `GET /events` staat
letterlijk: "Vraag opgeslagen events op **van de bronhouder**", en bij
`/events/stream`: "stuurt nieuwe events zodra deze worden ontvangen **door de
bronhouder**" — een federatief model. Beide kunnen niet waar zijn, en het verschil
bepaalt alles: wie kent identifiers toe, wie is aanspreekbaar op beschikbaarheid,
en wat een afnemer moet doen als de schakel in het midden stilstaat.

0.0.2 kiest expliciet: **de bronhouder biedt zijn eigen feed aan, afnemers
aggregeren.** Het `POST`-patroon, de aanleverstatus en de status-webhooks zijn
eruit; die horen in de
[aanlever-ORI-API van KOOP](https://gitlab.com/koop/woo/aanleveren-ori) waar ze
vandaan komen. Wat overblijft is precies wat de mapnaam belooft: *event polling*.

Drie argumenten, in volgorde van gewicht:

1. **Minimale aansluitlast.** Het
   [BIT-advies over PLOOI](https://www.adviescollegeicttoetsing.nl/site/binaries/site-content/collections/documents/2022/11/28/bit-advies-plooi/BIT-advies+Platform+Open+OverheidsInformatie.pdf)
   concludeerde dat de gekozen oplossing leunde op een standaard die overheden
   moeilijk kunnen volgen, en dat een herontwerp nodig was met minimale
   aansluitlast voor overheidsorganisaties. Een feed aanbieden is precies dat: een
   bronhouder ontsluit zijn eigen gegevens één keer, en hoeft geen integratie te
   bouwen per landelijke voorziening. Elke nieuwe afnemer kost hem daarna niets.
2. **Geen enkel punt waarop de keten stilvalt.** In een centraal model is de
   dekking van iedere afnemer gelijk aan de dekking van de schakel in het midden.
   Loopt die achter, dan lopen alle hergebruikers achter, en niemand kan er iets
   aan doen. Bij feeds per bronhouder is een storing lokaal.
3. **Het is het GEMMA-uitgangspunt uit de eigen architectuurnotitie.** In
   [Architectuur.md](../../Architectuur.md) staat "eenmalig vastleggen, meervoudig
   gebruiken", met voor publicatie een verwijzing naar de opslaglocatie in het
   bronregister. Een publicatiefeed bij de bron is de letterlijke uitwerking
   daarvan; een centraal platform dat de inhoud overneemt is dat niet.

Dit sluit een landelijke voorziening niet uit — die wordt een afnemer als alle
andere, en kan precies dat doen waar hij goed in is: **een register van
feedlocaties bijhouden** in plaats van de inhoud kopiëren. Dat is ook wat de
Woo-index al is. Om dat werkbaar te maken voegt 0.0.2 een servicedocument toe
(`GET /`, plus een verwijzing op `/.well-known/ori-feed` van het eigen domein),
waarin een feed zelf vertelt welke organisaties erin zitten, welke resourcetypes
hij ondersteunt en hoe ver de historie teruggaat.

## Rollen: bronhouder, aggregator, register

Een veelgehoord bezwaar tegen federatie is dat een hergebruiker dan bij honderden
losse feeds langs moet. Dat hoeft niet, en het is ook niet hoe het vandaag werkt:
er zijn al aggregatoren. Dit model maakt die rol expliciet in plaats van
impliciet, en gebruikt er geen apart contract voor.

- Een **bronhouder** publiceert de feed van zijn eigen raadsinformatie.
- Een **aggregator** consumeert feeds van meerdere bronhouders en publiceert het
  resultaat als *weer een feed volgens deze specificatie*. Dat werkt omdat
  `sequenceNumber` en `cursor` per feed gelden: een aggregator kent zijn eigen
  reeks toe en hoeft de bovenliggende feeds niet te vertalen.
- Een **register** houdt alleen bij *waar* feeds staan, en kopieert geen inhoud.

Een hergebruiker kiest dus zelf: rechtstreeks bij één gemeente wanneer hij één
gemeente volgt, of in één keer bij een aggregator die er honderden bundelt en
bijvoorbeeld zoeken, tekstextractie of een archieffunctie toevoegt. Het
servicedocument vertelt via `rol` met wie hij praat, en per organisatie via
`laatstBijgewerkt` en `herkomst` hoe actueel het is en waar het vandaan komt.
Bestaande aggregatoren — [OpenBesluitvorming](https://openbesluitvorming.nl), de
opvolger van openraadsinformatie.nl, is er één — blijven in dit model dus werken,
en worden tegelijk aanspreekbaar volgens dezelfde afspraken als de bronnen zelf.

Voor die compositie zijn vier afspraken nodig, zodat een keten van hops dezelfde
garanties houdt als één stap. Ze staan in de spec en zijn hier kort onderbouwd:

1. **Identiteit blijft van de bron.** `resource` (`organisatiecode` +
   `bronidentificatie`) gaat onveranderd door; eigen identifiers mogen ernaast
   staan, niet in de plaats. Alleen zo kan een afnemer gegevens van een
   aggregator en van de bron zelf naast elkaar leggen zonder dubbel te tellen —
   en alleen zo kan hij van route wisselen zonder zijn dataset weg te gooien.
2. **`redact` propageert.** Een intrekking die bij de eerste hop blijft hangen,
   is geen intrekking. Een aggregator die een `redact` ontvangt, verwijdert de
   inhoud uit zijn eigen historie én publiceert zelf een `redact`.
3. **Geen `delete` bij eigen blindheid.** Dit is de fout die iedereen één keer
   maakt: een bron die onbereikbaar is of een leeg antwoord geeft, lijkt op een
   bron waar alles is verwijderd. Een aggregator die dat verschil niet maakt,
   tombstoned een hele gemeente. Publiceer in dat geval niets en meld de
   achterstand via `laatstBijgewerkt`.
4. **Verrijking hoort in de eigen API.** Geëxtraheerde tekst, thumbnails en
   zoekindexen passen niet in dit model en moeten er niet in geperst worden. Ze
   horen naast de feed, met de resource-sleutel als verwijzing.

Dat maakt ook een geleidelijke overgang mogelijk, wat in deze keten geen luxe is:
vier leveranciers, honderden organisaties, en niemand die op één datum omgaat. Een
aggregator kan per organisatie een 0.0.2-feed gebruiken zodra die bestaat, en
zolang dat niet zo is de bestaande leveranciers-API blijven benutten. Dat is
zichtbaar in het servicedocument in plaats van verborgen, zodat een hergebruiker
weet wat hij krijgt. Er is dus geen moment waarop de hele keten tegelijk moet
overstappen — en de eerste bronhouder die een feed aanbiedt, heeft er meteen wat
aan.

## De kern: een feed is een log, geen tabelquery

In 0.0.1 wordt de event-historie bevraagd met `offset`, `fromTimestamp`,
`toTimestamp` en `sortOrder` (default `desc`). Voor een dashboard werkt dat; voor
synchronisatie niet:

- **`offset` schuift.** Terwijl een afnemer pagineert, komen er events bij. Met
  `sortOrder=desc` betekent iedere nieuwe event dat de volgende pagina records
  overslaat of dubbel levert. De afnemer merkt daar niets van: het resultaat is
  een stil gat in zijn kopie.
- **`timestamp` is de klok van het bronsysteem.** Die is niet monotoon, en een
  wijziging die met terugwerkende kracht bekend wordt heeft een tijdstempel in het
  verleden. Een afnemer die pollt met `fromTimestamp=<laatst gezien>` mist die
  permanent — precies het geval dat in deze keten dagelijks voorkomt: een document
  dat achteraf aan een oude vergadering wordt gehangen.
- **Er zijn twee tijdstempels** (`timestamp` en `storedAt`) en de spec zegt niet
  op welke van de twee wordt gefilterd en gesorteerd.

0.0.2 vervangt dit door de vier afspraken die een feed synchroniseerbaar maken:
een `sequenceNumber` dat de bronhouder bij vastlegging toekent, de garantie dat er
nooit iets vóór een al gelezen nummer wordt ingevoegd, uitlevering altijd
oplopend, en gaten in de reeks uitsluitend als expliciet `redact`-record. Verder
een opake `cursor` in plaats van `offset`, plus `head` (hoe ver loop ik achter?)
en `hasMore: false` als "je bent bij"-signaal.

Eén subtiliteit die makkelijk fout gaat: de cursor duidt een positie in de
**volledige** feed aan en filters worden daarná toegepast. Anders is een cursor
alleen geldig bij exact dezelfde filtercombinatie, en dat is een val waar
afnemers in lopen zodra ze hun filter aanpassen.

Deze bouwstenen zijn niet ORI-specifiek. Ze zijn parallel aan dit voorstel
ingediend als generiek patroon `patterns/sync-feed` in het
[VNG API lab](https://github.com/VNG-Realisatie/vng-api-lab), met
ADR-0007 als onderbouwing (open data: publicatie via pull-feed, als afbakening
van de CloudEvents-push uit ADR-0004). Landt dat patroon, dan kan deze
specificatie ernaar verwijzen in plaats van de definities te herhalen.

## Initiële synchronisatie ontbrak

Zonder snapshot moet een nieuwe afnemer de volledige historie herspelen, met
`limit` maximaal 100. Voor 120+ organisaties en jaren archief is dat niet
haalbaar. 0.0.2 voegt `GET /snapshot` toe: de huidige stand, met op de eerste
pagina de `feedCursor` waarmee je zonder gat op de feed overstapt.

Dat die cursor wordt vastgesteld **vóórdat** de snapshotrijen worden gelezen, is
geen detail: doe je het erna, dan vallen wijzigingen die tijdens het doorlopen van
de snapshot binnenkomen tussen wal en schip.

`limit` gaat naar maximaal 1000 (default 500). Voor machineconsumenten is 100 te
laag: het maakt een initiële sync onnodig traag en de rate-limits onnodig scherp.

## Identificatie zonder centrale uitgifte

In 0.0.1 heeft `InformatieObject` een `id` ("kenmerk van het object binnen het
bestuursorgaan zelf"), maar `Vergadering` en `Agendapunt` hebben dat niet. Verder
bestaan er drie identifiers voor één ding — `eventId`, `resourceUrl` en
`resourcePid` — zonder gedefinieerde relatie; in de voorbeelden is `resourceUrl`
zelfs opgebouwd uit het `eventId`, waardoor twee wijzigingen op dezelfde
vergadering twee verschillende `resourceUrl`s krijgen.

Zonder centrale PID-uitgifte is de oplossing eenvoudiger dan met:
**`organisatiecode` + `bronidentificatie`** is de identiteit. Landelijk uniek
(TOOI), en houdbaar als een organisatie van leverancier wisselt of een feed van
domein verhuist. Een aggregator die zelf identifiers uitgeeft, leidt die
deterministisch af uit (`organisatiecode`, `resourceType`, `bronidentificatie`) en
houdt ze daarmee stabiel over herindexaties heen. De URL waar een resource te
vinden is (`url`, `resourceUrl`) is een adres, geen identiteit — dat staat nu ook
expliciet in de spec.

Dat wijzigende identifiers hier een reëel risico zijn, is geen theorie: het was de
best-gedocumenteerde klacht van hergebruikers van de bestaande ORI-API. Sinds we
ID-stabiliteit als expliciete garantie zijn gaan documenteren, is het een van de
sterkste argumenten van de nieuwe API geworden.

Hetzelfde probleem zat in `VerwijzingNaarResource`, dat `id` én `url` verplicht
stelde — beide waarden kende de bronhouder bij aanlevering nog niet. In 0.0.2
verwijst hij via `bronidentificatie`, met `url` als `readOnly` aanvulling.
Daardoor zijn ook `agendapunten`, `subagendapunten` en `deelvergaderingen`
uitdrukbaar; in 0.0.1 stond `agendapunten` alleen op de PID-variant.

En omdat er geen centraal toegekende PID meer is, kan het paar
`X`/`XZonderPid` per resource verdwijnen: één schema per resource volstaat. Dat
lost ook de tegenspraak op waarin de beschrijving bij `data` sprak over "het
volledige object met PID" terwijl de `oneOf` alleen de `ZonderPid`-varianten
aanbood.

## CREATE/UPDATE/DELETE → upsert/delete

De bronsystemen kunnen dit onderscheid grotendeels niet leveren. Uit onze eigen
adapters:

| Leverancier | Wijzigingssignaal | Verwijdersignaal |
|---|---|---|
| NotuBiz | `last_modified`, alleen op documenten | geen |
| iBabs | `MutationDate`, `GetMeetingsChangedSince` | `GetMeetingsDeletedSince` (bij ons nog niet in productie geverifieerd) |
| GemeenteOplossingen | geen | geen |
| Parlaeus | geen | geen |

Eén van de vier heeft een verwijdersignaal. Waar een RIS de wijzigingen zelf
publiceert is dat probleem kleiner — het systeem weet wat er in zijn eigen
database gebeurt — maar het onderscheid CREATE/UPDATE blijft zinloos voor de
afnemer: hij moet beide gevallen behandelen als "vervang de toestand die ik had".
Daarom kent 0.0.2 `upsert` en `delete`.

Daarbij hoort `contentHash`, om twee redenen. Ten eerste als rem: een
herindexatie of modelwijziging aan de bronkant produceert anders een storm van
no-op-records, en dan haalt iedere afnemer bij elke herverwerking de hele dataset
opnieuw op — dat was de grootste kostenpost van de bestaande ORI-API. Ten tweede
als ordeningssleutel per resource, die in 0.0.1 ontbreekt: twee `UPDATE`s op
dezelfde resource zijn met alleen een bron-`timestamp` niet te ordenen.

## Verwijderen, intrekken en redactie

In 0.0.1 is een `DELETE` een `pid` plus een vrij tekstveld `metadata.reason` dat
volgens de beschrijving verplicht is maar in het schema niet `required` staat. Voor
een Woo-publicatie is dat te dun: er gaan drie verschillende gebeurtenissen onder
één noemer (het bestuursorgaan trekt in / valt onder een Woo-uitzonderingsgrond /
onrechtmatig openbaar gemaakte persoonsgegevens), terwijl een afnemer bij de
laatste twee meer moet doen dan de resource verwijderen — ook uit zijn zoekindex,
caches en afgeleide producten.

Belangrijker is de spanning die eronder zit. Een permanente historie waarin `data`
het volledige object bevat, is een onuitwisbaar archief van precies die gegevens
die je soms moet kunnen intrekken. Wij hebben dit als concreet dossier op ons bord
gehad (persoonsgegevens in bijlagen die uit alle kopieën moesten verdwijnen), en
het is achteraf niet toe te voegen zonder het cursorcontract te breken.

0.0.2 doet daarom drie dingen: `verwijderreden` is verplicht en gecodeerd; er is
een `redact`-operatie die aangeeft dat eerdere inhoud uit de feedhistorie is
verwijderd en welke `sequenceNumber`s dat betreft; en feedrecords bevatten
metadata en verwijzingen in plaats van bijlage-inhoud, met het bestand achter
`bestandsurl`.

## Kleinere correcties

| Onderwerp | 0.0.1 | 0.0.2 |
|---|---|---|
| Beveiliging | `security: []` op documentniveau, géén `securitySchemes`, wel overal `401` gedocumenteerd — formeel is de aanlever-POST dus publiek | de feed ontsluit openbare gegevens en vereist niets; er is dus ook geen `401` meer, en met de POST-kant vervallen de authenticatievragen in deze spec |
| `GET /webhook-subscriptions` | geeft álle registraties terug, ongeauthenticeerd: een publieke lijst van interne webhook-URL's van gemeenten | vervallen met de aanleverkant |
| Foutantwoorden | eigen `ErrorResponse {titel, status, detail}` | RFC 9457 `application/problem+json` met `invalidParams`, conform de API-strategie-extensies |
| `nullable: true` | OpenAPI 3.0-syntax in een 3.1-document (ongeldig; elders wordt wél `type: [string, "null"]` gebruikt) | consequent `type: [..., "null"]` |
| `data.oneOf` | geen discriminator, `additionalProperties` open: een payload die aan twee `...ZonderPid`-schema's voldoet maakt `oneOf` ongeldig | `discriminator` op `dossiertype`; daarvoor is `dossiertype` toegevoegd aan `InformatieObject` |
| `organisatieCode` | voorbeeld `GM0363` in de query, `gm0363` in het schema; case-gevoeligheid ongedefinieerd | `Organisatiecode` met TOOI-patroon, kleine letters normatief, normalisatie gedocumenteerd |
| Collecties | `/agendapunten` e.d. zonder enige parameter, en een `404` op een collectie | onder `/organisaties/{organisatiecode}/…`, met `cursor` en `limit`; geen `404` op een lege collectie |
| Vindbaarheid | niet benoemd | servicedocument op `GET /`, plus `/.well-known/ori-feed` als aanbeveling |
| `total` | verplicht op elke pagina; duur en weinig zinvol bij een groeiend log | vervallen; `head` geeft de achterstand |
| Versie in URI | alleen `/ori-mock` als server | majorversie in de URI conform de Landelijke API-strategie |
| Rate limiting | niet benoemd | `429`/`503` met `Retry-After`, `Cache-Control` en `Link` op de feed |
| SSE | responseschema als `type: object` met een `data`-property (zo werkt `text/event-stream` niet); `fromTimestamp` met default "nu", dus een gegarandeerd gat bij reconnect | `id:` = `sequenceNumber` met `Last-Event-ID`-hervatting, expliciet zonder synchronisatiegarantie |
| `InformatieObject` | alleen `webpaginalink` (een HTML-pagina) | plus `bestandsurl`, `omvang`, `checksum`, `gewijzigdop` |

Ter controle: `redocly lint --extends=minimal` geeft op 0.0.1 twee errors (de
`nullable`-regels) en drie warnings (de SSE-voorbeelden valideren niet tegen hun
eigen schema); 0.0.2 valideert schoon.

## Open punten

Bewust niet in dit voorstel verwerkt, omdat ze een aparte afweging verdienen:

1. **CloudEvents als recordvorm.** De velden in `FeedRecord` mappen vrijwel
   1-op-1 op de CNCF-standaard: `sequenceNumber`/`resource` → `id`+`subject`,
   `resourceType` → `type`, `storedAt` → `time`, `data` → `data`. CloudEvents
   heeft een HTTP-binding, een batchformaat en bibliotheken in elke taal — een
   lagere drempel voor leveranciers. Wij produceren intern al CloudEvents. Voor
   1.0 het serieus overwegen waard; voor dit concept was het te veel churn naast
   de inhoudelijke wijzigingen.
2. **Onveranderlijke, cachebare archiefpagina's.** Voor een publieke feed is het
   patroon uit RFC 5005 (afgesloten pagina's met een vaste URL en
   `Cache-Control: immutable`, plus één levende koppagina) operationeel goedkoper
   dan cursor-endpoints: alles behalve de kop is CDN-cachebaar en de bronhouder
   houdt geen sessiestate bij. Voor een gemeentelijke feed met veel afnemers is
   dat het verschil tussen "een bestand serveren" en "een API draaien". Te
   combineren met het cursorcontract hierboven.
3. **`Organisatie` samenvouwen** tot `{organisatiecode, naam}` — de TOOI-code
   drukt het onderscheid gemeente/provincie/waterschap al uit, en dat sluit aan op
   `ResourceSleutel`. Niet gedaan, omdat het het model raakt dat met KOOP wordt
   gedeeld.
4. **`verwijderreden` als TOOI-waardelijst** in plaats van een enum in de OAS.
5. **Hoort SSE in de standaard?** Een langlevende stroom naar een onbekend aantal
   anonieme afnemers is een operationele verplichting (fan-out, geen backpressure)
   voor iets waarvan de meerwaarde beperkt is als pollen goedkoop is. In dit
   voorstel staat de operatie erin, gerepareerd; wij zouden hem zelf niet als
   eerste bouwen — en voor een gemeente die dit moet aanbieden, is het de duurste
   eis in de hele specificatie.
6. **Conditionele verplichtingen formaliseren.** "`data` aanwezig bij `upsert`",
   "`verwijderreden` bij `delete`" staan nu in de beschrijving. In OpenAPI 3.1
   zijn ze met JSON Schema `if`/`then` ook machineleesbaar te maken; hier
   weggelaten omdat de tooling-ondersteuning wisselt.
7. **Retentie en aankondigingen.** Het servicedocument benoemt retentie, maar de
   standaard zou een ondergrens moeten stellen: hoe lang moet een feedhistorie
   minimaal terugreiken voordat een afnemer via de snapshot moet herstarten?
   Daarnaast vroegen hergebruikers van de bestaande ORI-API al in 2023 om
   versionering en aankondiging van breaking changes; machineleesbare
   `Deprecation`/`Sunset`-headers horen in het contract.
8. **Taalconsistentie.** Dit voorstel legt één grens vast (envelope Engels,
   domeinmodel Nederlands) in plaats van de gemengde naamgeving van 0.0.1. Als de
   voorkeur uitgaat naar volledig Nederlands is dat een prima aparte, mechanische
   wijziging.
