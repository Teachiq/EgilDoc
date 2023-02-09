# EGIL implementationsprofil för Exam.net och Kunskapsmatrisen

Beskrivning av den specialisering inom standarderna SCIM och SS12000 som tagits fram för automatiserad kontoadministration i digitala läromedel och liknande system. Profilen beskriver vilka objekttyper som ska användas, vilka av objektens attribut som skall användas, hur autentisering mellan ändpunkter utförs och hur federationen är utformad.

## Objekt och attribut

<font color=blue>Nedan</blue> följer det urval av tillgängliga objekttyper som finns beskrivna i SCIM core schema<sup>1</sup> och i SS12000 extended schema<sup>2</sup>.

Samtliga objekt som skapas hos klienten för överföring till en server skall alltid ha attributet _**externalId**_ som tilldelas sitt värde av klienten och som är av typen UUID (tex. 6561043b-c636-b247-8487-6561043bc636). När servern tar emot och lagrar objektet skall den alltid ge attributet _**id**_ samma värde som _externalId._

_Valfria_ objekt och attribut innebär att huruvida dessa skall skickas eller inte avtalas mellan varje tjänsteleverantör och huvudman.

## Organisation<sup>2</sup> (_valfritt_)

Ett objekt per skolhuvudman. Objektet innehåller endast organisationens namn och har inga attribut som refererar till andra objekt.

Eftersom klienten alltid kontaktar servern med sitt entityID vet servern redan att alla objekt från ett entityID tillhör samma organisation, därför kan objektet utelämnas.

| **Attribut** | **Dataformat** | **Beskrivning** ||
| --- | --- | --- | --- |
| displayName<sup>2</sup> | String | Organisationens namn | Krävs |

## SchoolUnitGroup<sup>2</sup> (_valfritt_)

Ett objekt som grupperar en eller flera skolenheter (SchoolUnit) till en &quot;skola&quot;. T.ex har större skolor ofta flera rektorer och varje rektor ansvarar för ett rektorsområde eller motsvarande.

SchoolUnitGroup har inga attribut som refererar till andra objekt, inte ens en lista på vilka SchoolUnit som ingår.

| **Attribut** | **Dataformat** | **Beskrivning** | |
| --- | --- | --- | --- |
| displayName<sup>2</sup> | String | Skolans namn | Krävs |

## SchoolUnit<sup>2</sup> (_obligatoriskt_)

I Skolverkets organisationsmodell för skolan har varje skolenhet precis en rektor och varje skolenhet har en unik nationell 8-siffrig skolenhetskod. I verkligheten tänker man ofta på en skola som en byggnad eller flera byggnader som alla tillhör skolan, tex. &quot;Centralskolan&quot;. Som elev eller vårdnadshavare känner man oftast inte till något om den organisatoriska uppdelningen i rektorsområden och skolenheter, utan man &quot;går på Centralskolan&quot;.

En _elev_ kan bara vara inskriven på _en_ skolenhet.

En _lärare_ kan undervisa på _flera_ skolenheter.

| **Attribut** | **Dataformat** | **Beskrivning** | |
| --- | --- | --- | --- |
| displayName<sup>2</sup> | String | Skolenhetens namn | Krävs |
| schoolUnitCode<sup>2</sup> | String | SCB skolenhetskod | Krävs |
| organisation<sup>2</sup> | REF Organisation | Organisationstillhörighet | Valfritt |
| schoolUnitGroup<sup>2</sup> | REF SchoolUnitGroup | Skoltillhörighet | Valfritt |
| schoolTypes<sup>2</sup> | Code\_SchoolType | Skolform | Valfritt |
| municipalityCode<sup>2</sup> | String | Kommunkod | Valfritt |

## Employment<sup>2</sup> (_obligatoriskt_)

Ett abstrakt relationsobjekt som knyter en anställning till en person och en skolenhet. Varje person (User) som är anställd som lärare eller annan personal inom skolan skall ha ett Employmentobjekt. Om personen arbetar på mer än en skolenhet skall den ha ett Employment-objekt för varje skolenhet.

| **Attribut** | **Dataformat** | **Beskrivning** | |
| --- | --- | --- | --- |
| employedAt<sup>2</sup> | REF SchoolUnit | Anställande enhet | Krävs |
| user<sup>2</sup> | REF User | Anställd person | Krävs |
| employmentRole<sup>2</sup> | Code\_EmploymentRole | Befattning | Krävs |
| signature<sup>2</sup> | String | Lärarsignatur | Valfritt |

## Activity<sup>2</sup> (_obligatorisk_)

Ett relationsobjekt som ytterst binder ihop en lärarperson (User) med elevgrupp eller elev. Genom attributet teachers.employment (TeacherAssignment) refererar Activity via noll eller flera Employment-objekt till lärarpersoner (User). Genom attributet GroupAssignment refereras till elevgrupper (StudentGroup).

För tjänster där relationen lärare - elev är betydelsefull för tjänstens funktionalitet är Activity-objektet av central betydelse.

| **Attribut** | **Dataformat** | **Beskrivning** | |
| --- | --- | --- | --- |
| displayName<sup>2</sup> | String | Aktivitetens namn | Krävs |
| activityType<sup>2</sup> | Code\_ActivityType | Aktivitetstyp | Valfri |
| owner<sup>2</sup> | REF SchoolUnit | Kopplad skolenhet | Krävs |
| teachers<sup>2</sup> | REF Employment | Kopplad lärare | Krävs |
| groups<sup>2</sup> | REF StudentGroup | Kopplad elevgrupp | Krävs |
| topic<sup>2</sup> | REF Course/Subject | Kurs eller ämne | Valfri |

## StudentGroup<sup>2</sup> (_obligatorisk_)

Specialiserat gruppobjekt som förutom attributet studentMemberships som refererar till gruppens medlemmar (User) även har attribut som refererar till den skolenhet (SchoolUnit) som elevgruppen tillhör och beskrivande attribut. StudentGroup används både för skolklasser och undervisningsgrupper.

| **Attribut** | **Dataformat** | **Beskrivning** | |
| --- | --- | --- | --- |
| displayName<sup>2</sup> | String | Gruppens namn | Krävs |
| owner<sup>2</sup> | REF SchoolUnit | Gruppens skoltillhörighet | Krävs |
| studentMemberships<sup>2</sup> | REF User | Elev(er) i gruppen | Krävs |
| studentGroupType<sup>2</sup> | Code\_StudentGroupType | Typ av grupp | Valfritt |

## User<sup>1,2</sup> (_obligatorisk_)

Specialisering av User från SCIM core schema med ett antal extra attribut. Beskrivande attribut anpassade för svensk skola har lagts till, tex personnummer. Attributet enrolments innehåller elevinformation som årskurs (schoolYear), skolform (schoolType) mm. Attributet userRelations ger möjlighet att ange relation till ett annat User-objekt, tex vårdnadshavare (används inte i EGIL-profilen).

I grunden är User-objekten representationer av personer utan specificerad roll i organisationen. Rollen/rollerna bestäms genom relationsobjekt eller relationsattribut, tex kan en User ha ett Enrolment-attribut som anger att man är elev vid Vuxenutbildningen och samma User kan via kedjan Activity - TeacherAssignment - Employment vara lärare för en elevgrupp.

| **Attribut** | **Dataformat** | **Beskrivning** | |
| --- | --- | --- | --- |
| userName<sup>1</sup> | String | Användarnamn\* | Krävs |
| displayName<sup>1</sup> | String | Förnamn Efternamn | Krävs |
| name.familyName<sup>1</sup> | String | Efternamn | Krävs |
| name.givenName<sup>1</sup> | String | Förnamn | Krävs |
| emails<sup>1</sup> | String | e-postadress | Valfritt |
| studentGroups<sup>2</sup> | REF StudentGroup | Medlemskap i elevgrupp | Krävs\*\* |
| civicNo | String | Personnummer YYYYMMDDNNNN | Valfritt |
| enrolments<sup>2</sup> | REF SchoolUnit | Inskriven vid skolenhet | Krävs\*\* |
| enrolments.schoolYear<sup>2</sup> | Int | Årskurs\*\*\* | Valfritt |
| enrolments.programCode<sup>2</sup> | String | Studievägskod (Gy/Vux)\*\*\*\* | Valfritt |
| enrolments.schoolType<sup>2</sup> | Code\_schoolType | Skolform | Valfritt |
| userRelation<sup>2</sup> | REF User | Relaterad person | Valfritt |
| userRelation.relationType<sup>2</sup> | Code\_RelationType | Typ av relation | Valfritt |

\* Värdet ska ha samma format som eduPersonPrincipalName i Skolfederations attributprofil, dvs användarId@organisation.

\*\* Krävs för elever, inte för personal

\*\*\* 0 - 10

\*\*\*\* [https://www.skolverket.se/skolutveckling/anordna-och-administrera-utbildning/administrera-utbildning/koder-for-studievagar-yrkespaket-och-sprak](https://www.skolverket.se/skolutveckling/anordna-och-administrera-utbildning/administrera-utbildning/koder-for-studievagar-yrkespaket-och-sprak)

## Course (valfritt)

Ett objekt som beskriver en kurs som är knuten till en aktivitet.

| **Attribut** | **Dataformat** | **Beskrivning** | |
| --- | --- | --- | --- |
| courseCode<sup>2</sup> | String | Kurskod, exempelvis MATMAT01a | Krävs |

## Dataformat i attributen

De dataformat som attributen kan ha inom EGIL-projektet är:

**UUID** : Unik identifierare för en post. Består av 32 hexadecimala tecken (0 - 9, a - f) grupperade enligt 83a44180-6573-554b-0291-83a441806573

**String** : Teckensträng av godtycklig längd som kan innehålla utskrivbara tecken enligt UTF8

**Int** : Heltal av godtycklig längd

**Code\_EmploymentRole** : Datatyp med specificerat värdeförråd (fr SS 12000); _Rektor_, _Lärare_, _Förskollärare_, _Övrig pedagogisk personal_, _Annan personal._

**Code\_SchoolType** : Datatyp med specificerat värdeförråd; _FS_ (förskola), _FSK_ (förskoleklass), _FTH_ (fritidshem), _GR_ (grundskola), _GRS_ (grundsärskola), _SP_ (specialskola), _SAM_ (sameskola), _GY_ (gymnasieskola), _GYS_ (gymnasiesärskola, _VUX_ (vuxenutbildning), _SUV_ (särskild utbildning för vuxna), _YH_ (yrkeshögskola), _FHS_ (folkhögskola), _HS_ (högskola), _AU_ (annan utbildning).

**Code\_RelationType** : Datatyp med specificerat värdeförråd; _Vårdnadshavare_, _Annan ansvarig vuxen_.

**Code\_ActivityType** : Datatyp med specificerat värdeförråd; _Undervisning_, _Elevaktivitet_, _Läraraktivitet_, _Övrigt_.

**REF** (objekttyp): Referens till annat objekt inom en SCIM-domän. Objektreferensen består alltid av:

* _value_ (refererade objektets UUID)
* _$ref_ (refererade objektets URI i serversidans system)
* _display_ (endast vid GET, refererade objektets displayName, i förekommande fall)

## Övriga objekttyper i SS12000

I SS12000 beskrivs även objekttyperna _Subject_, _CalendarEvent_, _Room_, _Resource_, _TeacherException_, _StudentException_ och _StudentGroupException_ **.** I detta skede görs bedömningen att de inte är nödvändiga för att uppnå projektets syfte.

## Objekt med tidsangivelser i attribut

Objekttyperna _SchoolUnit_, _Employment_, _Activity och StudentGroup_ samt relationsattributen _Enrolment_ (User), _TeacherAssignment_ (Activity)_, StudentAssignment_ (Activity), _GroupAssignment_ (Activity), _StudentMembership_ (StudentGroup) och _UserRelation_ (User) har alla subattributen startDate och endDate. Dessa uppgifter är centrala vid överföring av skolschemainformation men används inte i EGIL-profilen eftersom överförda uppgifter alltid avspeglar aktuell situation.

## Obligatorisk delmängd av SCIM

En fungerande server-implementation behöver inte stödja allt från SCIM, inte ens allt som är obligatoriskt enligt SCIM. För att kunna ta emot data måste servern implementera POST (skapa), PUT (uppdatera) och DELETE (ta bort) för alla objektstyper.

GET (för att läsa objekt) måste inte implementeras, dock rekommenderas grundläggande stöd även för GET. Under normal drift läser klienten inte från servern. Om klient och server av någon anledning har hamnat ur synk, eller om man misstänker att så kan vara fallet, har klienten dock bättre möjligheter att &quot;laga&quot; integrationen om servern kan berätta vilka objekt som finns. Det räcker alltså att klienten kan hämta en lista över _alla_ objekt (per objektstyp) och se vilka UUIDs servern har. Det är upp till server-implementationen om man vill använda paginering eller ej när alla objekt returneras. Stöd för filtrering och sortering behöver inte implementeras.

För de metoder som implementeras är det viktigt att returkoder används korrekt enligt SCIM-standarden. Fel kan tyda på att klient och server har hamnat ur synk, genom att tolka returkoderna har klienten möjlighet att i viss mån rätta till sådana problem.

## Autentisering mellan klient och server

Säker kommunikation mellan klienter och servrar implementeras genom TLS med ömsesidig autentisering enligt separat specifikation, [https://github.com/kirei/scim-fed-auth/](https://github.com/kirei/scim-fed-auth/)

Anslutningar från klienter som ej presenterar klientcertifikat skall nekas av servern.

För att en klient ska kunna skicka data till en server krävs följande:

- En certifikatutfärdare (kan göras in-house eller köpas in som tjänst - behöver ej vara betrodd av några operativsystem eller webbläsare)
- En privat och en publik nyckel, och ett klientcertifikat från denna utfärdare
- Metadata om utfärdaren och nyckelns fingeravtryck skall vara publicerat i Skolfederations metadataregister (se nedan)
- En HTTPS-klient som kan använda klientcertifikatet (i testsyfte kan förslagsvis curl användas). Klienten skall också verifiera att serverns certifikat är utfärdat av en av de certifikatutfärdare som den mottagande organisationen har listat i metadataregistret, och att certifikatets nyckel har rätt fingeravtryck.
- Ett avtal med den mottagande organisationen. I normalfallet ska den mottagande servern ha en lista över vilka organisationers data som skall importeras från skolfederations metadataregister, som sedan avgör vilka anslutningar som ska tillåtas.

## Transportskydd

All kommunikation skall skyddas av HTTPS med TLS version 1.2 eller senare, och endast chifferuppsättningar som resulterar framtida sekretesskydd (Perfect Forward Secrecy, PFS), t.ex. ECDHE/DHE, får användas.

## Federation och metadata

Den metadata som krävs för att implementera detta koordineras och hanteras av Skolfederation. Det bör dock påpekas att metadata för EGIL hanteras separat från den metadata som används för t.ex. SAML, och att det inte finns något direkt beroende på Skolfederation.

Innehåll i metadata för varje organisation:

- Entitetens (organisationens) ID, t ex egil.kommunen.se
- Lista över betrodda certifikatutfärdare
  - CA-certifikatet i sin helhet, i PEM-format
- Lista över klienter
  - Namn
  - Den publika nyckelns fingeravtryck
- Lista över servrar
  - Namn
  - URI
  - Den publika nyckelns fingeravtryck

Mer information om metadataregistret, inklusive nyckel för att verifiera metadata finns här:

[https://www.skolfederation.se/teknisk-information/kontosynk/tekniska-miljoer/](https://www.skolfederation.se/teknisk-information/kontosynk/tekniska-miljoer/)

Det metadataregister som användes under PoC-fasen finns publicerat här: [https://fedscim-poc.skolfederation.se/md/skolfederation-fedscim-0\_1.json](https://fedscim-poc.skolfederation.se/md/skolfederation-fedscim-0_1.json)

Registret är i signerat JSON-format, som ska verifieras med hjälp av denna publika nyckel: [https://fedscim-poc.skolfederation.se/jwks](https://fedscim-poc.skolfederation.se/jwks)

Exempeldata för en organisation ser ut på följande vis: [https://github.com/kirei/scim-fed-auth/blob/master/example.json](https://github.com/kirei/scim-fed-auth/blob/master/example.json)
