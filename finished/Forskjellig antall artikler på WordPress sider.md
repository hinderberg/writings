#Forskjellig antall artikler på WordPress sider
Skrevet: 21. mai 2009

Jeg har i en periode nå laget nytt design for min WordPress side, der jeg kom over et problem.

Det jeg ville var:
<ol>
	<li>Vise de tre siste artiklene på forsiden, index.php også gi mulighet til å gå til forrige eller neste side</li>
	<li>Vise alle artikler på arkiv siden</li>
	<li>Vise maks 50 artikler i en bestemt kategori eller et bestemt måned også vise forrige eller neste side</li>
</ol>
Problemet jeg kom over var at WordPress har kun en innstilling for antall artikler på en side, som igjen virker globalt på alle forespørsler mot den kjente loopen:

<pre><code class="php">&lt;?php while (have_posts()) : the_post(); ?&gt;</code></pre>

Så når jeg satte 3 artikler i innstillingene (Options - Reading - Show At Most) for å nå første mål så viste både andre og tredje mål også tre artikler.

Løsningen på dette ble følgende:
<ol>
	<li> I templaten archives.php, som må lages først og som ikke blir gått igjennom hvordan her så setter du inn:
<pre><code class="php">&lt;?php query_posts('posts_per_page=-1'); ?&gt;</code></pre>


før den kjente loopen.Denne setningen setter standard verdien til -1 som fører til at alle artiklene blir skrevet ut</li>
	<li> I templaten archive.php så setter du inn:
<pre><code class="php">&lt;?php if(is_archive) query_posts($query_string . &quot;&amp;posts_per_page=50&quot;); ?&gt;</code></pre>

Først sjekkes det for om siden er en arkiv side, altså om du er inne på en måned, år, kategori etc. Deretter legger den til antall artikler den skal vise per side. Sjekken for at det er en arkiv side er nødvendig for at side visningene skal fungere på alt annet, noe som er merkelig spør du meg;)</li>
</ol>
Håper dette sparte deg for noen timers søking på nettet.