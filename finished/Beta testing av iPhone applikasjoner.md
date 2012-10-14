# Beta testing av iPhone applikasjoner
Skrevet: 23/05, 2010

Under utviklingen av iNytt 4.5 så ønsket jeg at ett utvalg av brukere skulle beta teste applikasjonen før den ble lagt inn på App Store. Men det å lage applikasjonen klar for beta testing var ikke bare bare. Så jeg bestemte meg derfor å skrive ett blogg innlegg om stegene. Som vanlig blir dette presentert i en liste, da jeg er utvikler og elsker lister.;)

<h2>Legg til enheter i Provisioning Portal</h2>
<ol>
	<li>Be de som skal teste applikasjonen din om å innstallere <a title="Ad Hoc Helper" href="http://itunes.apple.com/app/ad-hoc-helper/id285691333?mt=8">Ad Hoc Helper</a>.</li>
	<li>Be de sende en e-post til deg ved hjelp av denne applikasjonen</li>
	<li>Når du har mottatt Device ID fra alle du ønsker gå inn på <a title="Provisioning Portal" href="http://developer.apple.com/iphone/manage/overview/index.action">Provisioning Portal</a></li>
	<li>Trykk på Devices i menyen</li>
	<li>Trykk på Add Devices og legg til enhetene du ønsker</li>
</ol>
<h2>Lag en ny provisioning profile</h2>
<ol>
	<li>Gå inn på <a title="Provisioning Portal" href="http://developer.apple.com/iphone/manage/overview/index.action">Provisioning  Portal</a></li>
	<li>Trykk på Provisioning i menyen</li>
	<li>Trykk så på Distribution tabben</li>
	<li>Trykk på New Profile</li>
	<li>Velg Ad hoc</li>
	<li>Skriv inn profil navnet</li>
	<li>Velg App ID</li>
	<li>Velg mellom enhetene du la til i første steget</li>
	<li>Trykk Submit</li>
	<li>Last ned den nye profilen</li>
	<li>Dobbelt klikk på filen, den legger seg sa inn i Xcode</li>
</ol>
<h2>Gjør applikasjonen klar for å bygges</h2>
<ol>
	<li>Åpne opp prosjektet ditt i Xcode</li>
	<li>Høyre klikk på filen som ligger under Targets (den vil hete det samme som din applikasjon)</li>
	<li>Trykk Get Info</li>
	<li>Trykk på Build tabben -&gt; Configuration drop-down boksen -&gt; Edit Configurations</li>
	<li>Velg Release og trykk Duplicate</li>
	<li>Velg ett passende navn</li>
	<li>Lukk vinduet og gå tilbake til Build tabben</li>
	<li>Velg din nye konfigurasjon fra Configuration drop-down boksen</li>
	<li>Gå til code signing identity -&gt; Any iPhone OS Device og sett den til iPhone Distribtution (den du bruker når du gjør klar applikasjonen for App Store)</li>
	<li>Gjør en Clean Build</li>
	<li>Gå til Build mappen din -&gt; Kan gjøres på følgende måte: Inne i Products folderen i prosjektet finner du applikasjonen din -&gt; Høyre kikk og Reveal in Finder</li>
	<li>Send både applikasjonen og .mobileprovision filen du lagde i sta til de du har fått enhets ID'r fra.</li>
</ol>
<h2>Installering av applikasjonen</h2>
<ol>
	<li>Dra begge filene inn i iTunes</li>
	<li>Slipp de over Library gruppen</li>
	<li>Synkroniser iPhonen din</li>
</ol>