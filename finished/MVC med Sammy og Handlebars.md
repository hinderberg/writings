MVC med Sammy og Handlebars
===

I disse dager hvor server-side er ut og klient-side er kult har det dukket opp mange avanserte JavaScript-rammeverk som flytter kompleksitet fra serveren til nettleseren. [Backbone](http://documentcloud.github.com/backbone/), [Knockout](http://knockoutjs.com/) og [JavaScriptMVC](http://javascriptmvc.com/) er alle gode eksempler som kan brukes til å lage komplekse applikasjoner, uten antikvariske operasjoner som lasting av hele sider eller rendering av HTML på serveren. I denne nye verden tilbyr serveren et REST-API, og all koden for å bruke dette API-et er Javascript som kjøres av brukerens nettleser.

Vi vil gjerne presentere én metode for å strukturere komplekse JS-applikasjoner, sammen med to flotte biblioteker: [Sammy](http://sammyjs.org/) og [Handlebars](http://handlebarsjs.com/).

**Sammy** er ikke like kjent eller kraftig som de tidligere nevnte rammeverkene, men er likevel svært godt til sitt bruksområde og fortjener en vurdering ved valg av rammeverk. Hovedsakelig brukes Sammy til routing, hvor en URL besvares av en JS-funksjon, og til å rendre HTML-en som skal vises på den aktuelle siden. Biblioteket er i seg selv svært lite, og delegerer det meste av funksjonalitet til sin strålende plugin-arkitektur. I dag er Sammy tungt avhengig av jQuery.

**Handlebars** har den siste tiden fått masse oppmerksomhet, og bygger på det allerede kjente template-rammeverket Mustache. Handlebars brukes til å sammenflette data og HTML på en enklere og strukturert måte. Hensikten er å skille HTML- og DOM-maipulasjon fra resten av applikasjonen, og Handlebars inneholder mange hjelpemetoder som gjør dette ganske elegant.

**Model-View-Controller (MVC)** er en type arkitektur som kan brukes til komplekse klient-side JS applikasjoner. I vår (svært) løse definisjon av MVC får vi følgende struktur:

* Datamodellene håndterer entiteter og kommunikasjon med serveren.
* View-modellene står for oppførselen og strukturen til HTML-elementene.
* Controllerene knytter datamodeller og view-modeller sammen mot spesifikke URL-er.

God struktur er vanskelig i komplekse JS-applikasjoner. Språket gir store muligheter og få begrensninger, og ingen forslag til hvordan strukturen bør være. Det hele kan fort bli uoversiktelig hvis man ikke baserer seg på velkjent struktur, som for eksempel MVC. I denne teksten bygger vi deler av en slik applikasjon.

I eksempelet har serveren en liste personer, som vi ønsker å få tak i, instansiere som modeller og vise en liste av i nettleseren. All kode i denne bloggposten kommer fra [en minimal eksempelapplikasjon du finner på Github](https://github.com/hinderberg/sammy-handlebars-blog-example). Last den gjerne ned og prøv resultatet i din nettleser. Koden kan også brukes som refferanse om det er noe som er uklart i teksten.

Datamodeller og API-kall
---

Datamodellene tar seg av kommunikasjon med serveren via Ajax-baserte HTTP-kall mot et REST-API. Det er mange måter dette kan gjøres på, det finnes ingen fasit, men her kommer ett eksempel. Her har vi kodet modellene selv, men merk at på samme måte som Handlebars tar seg av templates, og Sammy tar seg av controller-ene, kunne vi brukt et tredje bibliotek til å ta seg av datamodellene. Men la oss lage noe selv.

Noen vil kanskje reagere på at datamodeller her potensielt defineres to steder — hos klienten og på serveren — men det er viktig å huske at disse modellene har lite felles funksjonalitet. Datamodellene på klientsiden er der utelukkende for å hjelpe view-modellene, ved å abstrahere bort funksjonalitet for kommunikasjon med serveren, formatering av modelldata og lignende.

Det er to *oppgaver* som kan skilles i datamodellene:

* Selve datamodellene som instansieres og har metoder for å jobbe med hver entitet,
og statiske metoder for hver klasse av modeller, som blant annet gjør API-kall.
* Vi har valgt å bruke standard prorotype-baserte JS objekter. I dette eksempelet lager vi en modell kalt “person”, lagt under namespacet “myapp.models”.

```javascript
/* prototype for alle data-modeller. *
	 myapp.model.datamodel = {};
 
/* Metode for å instansiere en modell. */
myapp.model.create = function(prototype, properties) {
    return myapp.model.merge(Object.create(prototype), properties);
};
 
/* Prototype for person-instanser. */
myapp.model.person = myapp.model.create(myapp.model.datamodel, {
    name: function() {
        return this.firstname + " " + this.lastname;
    }
});
 
/* Instansiering. */
var person = myapp.model.create(myapp.model.person, { 
    firstname: “Foo”, 
    lastname: “Bar” 
});
```

Som sagt kan API-kall tilhørende hver modell skilles ut i egne moduler. Dette er flott når man skal teste systemet, og hjelper holde kompleksiteten nede. Her er et enkelt eksempel på API-kall for modellen:

```javascript
/* Metode for å lage mange instanser. */
myapp.model.createAll = function(prototype, propertiesList) {
    return propertiesList.map(function(properties) {
        return myapp.model.create(prototype, properties);
    });
};
 
/* Metoder for å gjøre kall mot person-API. */
myapp.api.person = {
    all: function(callback) {
        /* Hent listen over alle personer fra serveren. */
        $.ajax({ 
	    success: function(serverdata) {
                callback(myapp.model.createAll(
                    myapp.models.person, 
                    serverdata));
            },
            ........
        });
    }
};
 
/* Bruk */
myapp.api.person.all(function(people) {
/* “people” er her et array av person-instanser. */
});
```

Dette kan virke som mye kode for enkle operasjoner, men etterhvert som applikasjonen blir større er det fint å ha alt som har med serverkommunikasjon og datamodeller skilt fra resten av applikasjonen. La oss nå se hvordan disse datamodellene kan kobles sammen med rendring av HTML.

View-modeller med Handlebars
---


View-modellene knytter data sammen med HTML. Disse modellene holder på dataene som trengs for å vise en side, representert av datamodellene. I tillegg legger view-modellene på oppførsel, animasjoner og hendelser på siden-, eller delen av siden de representerer. Dette kan for eksempel være ting som skal skje når brukeren klikker på en link, flytter på elementer eller vil lagre eller endre data.

Poenget med å bruke view-modeller er å abstrahere bort alt som omhandler hvordan HTML skal produseres og oppføre seg. På samme måte som datamodellene innkapsulerer alt som har med entiteter og server-kall, tar view-modellene seg av all JS-kode som manipulerer DOM-en direkte. Målet er som alltid å ende opp med veldefinerte og klart avskillte moduler med hver sine oppgaver. Her er et eksempel på en view-modell:

```javascript
myapp.view.people = myapp.model.create(myapp.model.viewmodel, {
    compile: function() {
        return { people: this.people };
    },
    apply: function() {
        $("#people li").click(function(event) {
            alert($(this).text() + " clicked!");
        });
    }
});
````

En view-modell jobber i to faser og trenger to forskjellige metoder. For det første har vi compile-metoden, som returnerer alle variabler som skal kunne brukes fra template-filen. Deretter har vi apply-metoden, som kalles etter at template filen har blitt renderet.

Den første metoden gjør altså klar variabler som skal brukes, mens den andre tar seg av all DOM-manipuilasjon etter at HTML-en er rendret. *Apply-metodene til view-modellene er altså de eneste stedene i applikasjonen JS-kode jobber direkte med HTML-elementer, noe som gjør det hele svært ryddig*. Se eksempelapplikasjonen på GitHub [g] for mer av koden som jobber med view-modeller.

View-modellen jobber med en eller flere template-filer, som spesifiserer strukturen på markup-en til den aktuelle siden. Her er et eksempel på en slik template-fil:

```javascript
<div id="content">
{{ >search }}
    <ul id="people">
    {{ #each people }}
        <li>{{ name }}</li>
    {{ /each }}
    </ul>
</div>
```

I Handlebars bruker man Handlebar Expressions til å hente ut objekter ved å skrive {{ objektnavn }}, eller {{{ objektnavn }}} når man ønsker å un-escape. people er definert av view-modellen og brukes direkte i template-filen. name er definert av data-modellen for person-entiteten, som også brukes direkte i dette templatet.

I templaten har vi også en partial komponent, kalt search. Partials vil bli brukt der man ønsker å dele opp templates og lage gjenbrukbare deler som kan brukes på tvers av andre templates. Partial-templaten vil også ha direkte tilgang til view-modellen og dens innhold, men vil som oftest tilhøre en egen view-modell.

I tillegg til å kunne hente ut objekter og kjøre funksjoner, kan man bruke såkalte Block Expressions. Block Expressions gjør det mulig å kjøre Block Helpers som er hjelpefunksjoner for å eksempelvis gjøre operasjoner med lister og sjekke verdier med if/else spørringer. Handlebars kommer med ett sett innebygde helpers, men man kan også lage sine egne om dette skulle være nødvendig.

Her er et eksempel på en Block Helper, som kjører en loop inntil et visst maksimalt antall ganger:

```javascript
/* Definering av en egen hjelpemetode kalt “until”. */
Handlebars.registerHelper('until', function(context, block) {
    var max = block.hash.max;
    var ret = "";
    for (var i = 0, j = context.length; i < j; i++) {
        if (i >= max) { return ret; }
        ret = ret + block(context[i]);
    }
    return ret;
});
```

```javascript
/* Bruk av denne hjelpemetoden i en template-fil. */
{{ #until persons max="3" }}
    {{ name }}
{{ /until }}
```

En annen interessant mulighet man har i Handlebars er å gjøre kall mot parent-objektet til det objektet man jobber med. Her er et eksempel hvor vi bruker notasjonen “../” til å aksessere en variabel som tilhører parent-objektet til det vi looper gjennom:

```javascript
/* Variabel definert i view-modellen. */
this.family = { surname: “I skogen”, members: [
    { name: “Hans”, age: “90” },
    { name: “Grete”, age: “100” },
    { name: “Ulven”, age: “120” },
]};
```

```javascript
/* Bruk av variabelen i template-filen. */
{{ #each family.members }}
    {{ name }} {{ ../surname }}
{{ /each }}
```

Dette er selvfølgelig bare noen få av muligheten man har med Handlebars, men de viser hvordan komplekse templates kan spesifiseres i template-filer som jobber direkte mot en view-modell. Dette sørger for praktisk innkapsulering av alt som dreier seg om HTML og DOM-elementer i applikasjonen.

La oss nå se hvordan det hele kobles sammen ved hjelp av controller-ene og Sammy.

Controllere med Sammy
---

Controller-ene er selve limet som holder applikasjonen sammen. Disse beskriver hva som skal skje når en bruker besøker en URL. Controlleren bruker datamodellene til å hente data som trengs på den aktuelle siden, og rendrer HTML ved hjelp av de tidligere nevnte view-modellene og deres representative template filer. Vi trenger både routing og template-rendring, altså er dette en perfekt oppgave for Sammy.

> For å få et inntrykk av hva mer enn dette Sammy kan brukes til er det bare å se på listen over tilgjengelige plugins. Eksempler inkluderer templating med mange forskjellige biblioteker, Form Builder som minner om hvordan Ruby on Rails setter opp skjemaer, Storage som innkapsulerer mange forskjellige lokale lagringsmetoder i et enkelt API, eller OAuth-pluginen, forklart av sitt eget navn.

Vi ønsker å bruke Sammy-spesifikk kode utelukkende i controller-ene, og ikke i andre deler av applikasjonen. På denne måten vil minst mulig av applikasjonen bli avhengig av dette rammeverket, i tilfelle man skulle ønske bytte det ut på et senere tidspunkt. I tillegg blir applikasjonen enklere å teste, spesielt på enhetsnivå, da vi ikke trenger å bry oss om Sammy annet enn i controller-ene.

Her er en kodebit som kjøres når brukeren gjør en GET-request mot URL-en “/#people”:

```javascript
/* Jobb på HTML-elementet “#page”. */
$.sammy("#page", function() {
    /* Bruk Handlebars-plugin. */
    this.use('Handlebars', 'hb');
 
    /* Start routes. */
    myapp.route.person(this);
}).run();
 
 
/* Route for person-listen. */
myapp.route.person = function(app) {
    /* Definer en route. */
    app.get("", function(context) {
        /* Gjør et kall mot serveren. */
        myapp.api.person.all(function(people) {
 
            /* Definer hvilken template-filer som trengs. */
            var template = "template/person-list.hb";
            var partials = { searchbox: "template/shared/searchbox.hb" };
 
            /* Instansierer view-modellene */
            var search = myapp.model.create(myapp.view.shared.searchbox);
            var list = myapp.model.create(myapp.view.people, { people: people });
 
            /* Bytt ut innholdet i "#page" med ny HTML. */
            context.renderAll(template, partials, [list, search]);
        });
    });
};
```

Den første kodebiten sier at Sammy skal jobbe på HTML-elementet med ID-en “page”, definert i den initsielle HTML-filen. Del to tar tak i URL-en ved hjelp av en route-funksjon. En route består av et verb, en sti, og en callback-funksjon. Verbet er et av de velkjente HTTP-metodene POST, PUT, GET eller DELETE. Stien kan enten være en vanlig URL, eller en hash-basert URL som i dette eksempelet.

Hver Sammy-rute tar inn et context-objekt som inneholder mye informasjon om tilstanden til applikasjonen. Dette objektet inneholder også mange nyttige hjelpefunksjoner. For at applikasjonen skal holde seg modulær er det viktig å kunn bruke dette objektet i controllerene. Hvis context-objektet blir med til datamodellene eller view-modellene, vil disse bli avhengig av Sammy-logikk, som igjen vil komplisere testing og vedlikehold.

Det meste skjer i context.renderAll-metoden. Denne rendrer et template, et sett partials og en samling view-modeller ved hjelp av Sammy’s context-render-metode:

```javascript
context.helpers({
    /* En hjelpefunksjon for å rendre sider med mange view-modeller. */
    renderAll: function(tmpl, partials, models) {
       	var context = this;
       	this.render(tmpl, myapp.model.compileAll(models), function(html) {
            context.swap(html);
            myapp.model.applyAll(models);
        }, partials);
    }
});
```

context.render tar først inn stien til template-filen man vil bruke. Det neste argumentet er dataene man vil bruke til å rendre template-filen, i vårt tilfelle representert dataene av en view-modell. Denne view-modellen vil da være tilgjengelig for template-filen når den rendres av Handlebars-pluginen til Sammy.

Callback-argumentet context.swap tar imot ferdig rendret HTML, og bytter ut hva som allerede er i elementet som jobbes på (“#page”) med den nye HTML-en.

Det siste argumentet, “partials”, er et objekt som kan inneholde flere deler som hovedtemplaten er avhengig av, for eksempel en topp, meny og bunn. Render-metoden cacher template-filene, uten å laste de på nytt hver gang, slik at operasjonen blir kjappere andre gang denne kalles.

En fin måte å organisere disse Sammy-routene på er å ha én JS-fil pr side (hvis det gir mening i applikasjonens sidestruktur, vel å merke). Metoden $.sammy kan brukes på tvers av filer uten problemer. Hver side får ofte flere routes for forskjellige operasjoner, for eksempel:

```javascript
/* settings.js: routes for én side. */
 
$.sammy(“#page”, function() {
    // Vise innstillinger
    this.get(“#/settings/profile”,  function(context) { … });
    this.get(“#/settings/friends”,  function(context) { … });
 
    // Lagre innstillinger
    this.put(“#/settings/profile”,  function(context) { … });
    this.post(“#/settings/friends”, function(context) { … });
});
```

Med controllerene på plass er applikasjonens struktur komplett:

* **Datamodellene** innkapsulerer alt som beskriver entiteter og serverkall.
* **View-modellene** innkapsulerer alt som beskriver HTML-en og oppførselen til DOM-elementer, og kobler dette med datamodellene.
* **Controllerene** tar det hele i bruk og bruker datamodellene og view-modellene til å rendre flott HTML når brukerne besøker forskjellige URL-er.

Oppsummering
---

Selv om en struktur som den presentert her kan virke overdrevet i starten av et prosjekt, vil det hjelpe etterhvert som kompleksiteten og kodemengden øker. Det viktigste er å starte med et godt utgangspunkt, og deretter kontinuerlig forbedre og utvikle arkitekturen.

Det er selvfølgeilig ulemper med tunge JS-applikasjoner. De setter store krav til nettleserens JS-motor, og til utviklernes testing på tvers av mange nettlesere. Språkets frie natur gjør det enkelt å skrive lite vedlikeholdbar kode, og det er sjelden klare svar på hvordan noe bør gjøres.

Til syvende å sist krever komplekse JS-applikasjoner mye av disiplin innen struktur og testing, men denne friheten betyr også at JS-programmering er veldig spennende. Med gode biblioteker som Sammy og Handlebars blir det hele enda morsommere.