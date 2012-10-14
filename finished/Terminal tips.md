# Terminal tips
Skrevet: 07/10, 2010

Bruker denne posten til å samle tips/triks jeg kommer over til Mac OS X terminalen.
<h2>Kommandoer</h2>
<h3>Open</h3>
Om du skulle ønske å åpne mappen du er i når du bruker terminalen. Kan du bare skrive kommandoen 'open .' Denne kommandoen kan du også åpne filer med ved å for eksempel skrive 'open filnavn.filtype'.

Open kommandoen kan brukes til mye og du kan finne ut mer ved å skrive 'man open'. Her er noen få eksempler:
<ul>
	<li>'open *' åpner alle filer i en gitt mappe</li>
	<li>'open *.png' åpner alle png filer i mappen du står i</li>
</ul>
<h3>Alias</h3>
Om du trenger kjappe snarveier til mapper eller programmer du bruker ofte kan du bruke 'alias' kommandoen for å lage disse. Bare legg de inn i .bashrc. Hvis du for eksempel vil ha en snarvei til dokument mappen din kan du legge inn følgende:
<div id="_mcePaste">
<ul>
	<li>alias doc='cd /Users/YOURUSERNAME/Documents'</li>
</ul>
</div>
For å komme deg til dokument mappen din nå trenger du bare skrive doc og trykke enter.
<h3>Opensnoop</h3>
Opensnoop bruker DTrace for å vise deg hvilke filer som blir aksessert på dit system, en kan også spesifisere enkelt filer. Du kan for eksempel bruke den for å sjekke hvem som bruker passwd fila de ved å skrive: 'sudo opensnoop -f /etc/passwd'
<h3>!!</h3>
'!!' gjentar sist brukte kommando
<h2>Utseende</h2>
For å få ett mer lesbart utseende på terminalen anbefaler jeg artikkelen <a href="http://blog.infinitered.com/entries/show/6">A black OS X Leopard Terminal theme that is actually readable</a>. Dette fungerer bare om du starter terminalen i 32-bit mode etter installasjon.
<h3>Promt</h3>
For å få en annen promt kan du spesifisere 'export PS1='det du ønsker at skal være der' i .bashrc.

Jeg bruker følgende:
<pre><code>parse_git_branch() { git branch 2&gt; /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(git::\1)/' } parse_svn_branch() { parse_svn_url | sed -e 's#^'"$(parse_svn_repository_root)"'##g' | awk -F / '{print "(svn::"$2 ")"}' } parse_svn_url() { svn info 2&gt;/dev/null | grep -e '^URL*' | sed -e 's#^URL: *\(.*\)#\1#g ' } parse_svn_repository_root() { svn info 2&gt;/dev/null | grep -e '^Repository Root:*' | sed -e 's#^Repository Root: *\(.*\)#\1\/#g ' } export PS1='\[\033[1m\]\[\033[36m\]\w\[\033[33m\]$(parse_git_branch)$(parse_svn_branch)\[\033[0m\]\[\033[31m\]:&gt;\[\e[0m\] '</code></pre>
For en bedre forklaring av hvordan PS1 fungerer kan du sjekke ut denne artikkelen <a href="http://beckism.com/2009/02/better_bash_prompt/">A better bash prompt on Mac OS X</a>.