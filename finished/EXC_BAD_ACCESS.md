# EXC_BAD_ACCESS

Skrevet:  7. jul 2009

iNytt hadde en bug som gjorde at applikasjonen krasjet etter en stund, jeg brukte litt tid på å finne feilen da den oppsto rimelig random.

Feilmeldingen var som følger: EXC_BAD_ACCESS.

Denne feilmeldingen oppstår da du har releaset er objekt flere ganger enn du har retainet det. Altså hvis du releaser er objekt som ikke finnes i minne. Det er ofte problemer med å finne feilen da feilmeldingen ikke forteller hvor i koden du har gjort den. Du må derfor gå igjennom din kode for å sjekke, en god mulighet til å gjøre en liten oppvask med andre ord.;)

I mitt tilfelle releaset jeg ett objekt i en dealloc metode, der jeg hadde releaset objektet en gang tidligere.

Et kjapt tips for å unngå denne feilmeldingen er å huske på denne regelen:
<ul>
	<li>Hvis du bruker alloc, copy eller retain på et objekt må du alltid release objektet med release eller autorelease. Så lenge du bare mottar et objekt, har du ikke ansvar for å release det selv.</li>
</ul>
For mer informasjon les <a title="minnehåntering " href="http://developer.apple.com/documentation/Cocoa/Conceptual/MemoryMgmt/MemoryMgmt.html">denne</a> artikkelen om minnehåntering i objektive-c.

Oppdateringen til iNytt er sendt til til Apple, der den venter på godkjennelse.
