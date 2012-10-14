#! Invalid path - git push heroku master
Skrevet: 30. nov 2010 @ 23:18

Etter å ha fiklet litt med Ruby i det siste, tenkte jeg at det var på tide å sjekke ut <a title="Heroku" href="http://heroku.com/">Heroku</a>. Når jeg skulle deploy'e så fikk jeg følgende feilmelding:
<blockquote>
<div>! Invalid path.</div>
<div>! Syntax is: git@heroku.com:&lt;app&gt;.git where &lt;app&gt;  is your app's name</div></blockquote>
Etter litt googling fant jeg ut at denne feilmeldingen kan dukke opp når git repoet enten mangler en remote adresse, eller at det er noe feil med adressen. En kan sjekke om det finnes en remote adresse ved å kjøre følgende kommando.
<pre><code class="terminal">git remote -v</code></pre>
Du skal da se 2 x adresser, (fetch) og (push). Om terminalen ikke skriver ut noe, er ingen adresse satt. Du kan da legge til en ved hjelp av kommandoen:
<pre><code class="terminal">git remote add heroku git@heroku.com:&lt;applikasjonsnavn</code><span style="font-family: monospace;">&gt;.git</span></pre>
I mitt tilfelle var det en feil i adressen og jeg ønsket å redigere den. Dette kan gjøres i filen .git/config. Den vil inneholde noe som ligner på dette:
<pre><code class="terminal">[remote "heroku"] url = git@heroku.com:&lt;applikasjonsnavn&gt;.git fetch = +refs/heads/*:refs/remotes/heroku/*</code></pre>