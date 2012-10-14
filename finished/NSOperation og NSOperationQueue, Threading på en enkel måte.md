# NSOperation og NSOperationQueue, Threading på en enkel måte
Skrevet: 08/01, 2010

Alle som har jobbet med threading vet om alle problemene som kan oppstå og hvor vanskelig det kan være å feilsøke. Når en programmerer en iPhone applikasjon i Objective C vil en ofte få brukt for enkel threading for å unngå at GUI låser seg ved utførelse av diverse oppgaver. Det er her det er fint å vite at det finnes to objekter som heter NSOperation og NSOperationQueue.

Å lære seg hvordan disse to fungerer er utrolig enkelt. En kan for eksempel se på koden i ett eksempel hos Apple <a href="http://developer.apple.com/iphone/library/samplecode/TopSongs/index.html#//apple_ref/doc/uid/DTS40008925">TopSongs</a> eller lese klasse dokumentasjonen <a href="http://developer.apple.com/iphone/library/documentation/Cocoa/Reference/NSOperation_class/Reference/Reference.html">NSOperation</a>. Personlig liker jeg å lese dokumentasjon å prøve ut selv, da dette gir en bedre forståelse av hvordan klassene fungerer.
<h3>Et lite eksempel</h3>
Bruker her NSInvocationOperation som er en sub klasse av NSOperation

<pre><code class="objc">@interface OperationViewController : UITableViewController {
	NSOperationQueue *operationQueue;
	NSInvocationOperation *operation;
}

@property (nonatomic, retain) NSOperationQueue *operationQueue;
@property (nonatomic, retain) NSInvocationOperation *operation;
</code></pre>

<pre><code class="objc">#import &quot;OperationViewController.h&quot;

@implementation OperationViewController
@synthesize operationQueue;
@synthesize operation;

- (void)viewDidLoad {
	[super viewDidLoad];
	self.title = @&quot;Operation&quot;;
	[operationQueue setMaxConcurrentOperationCount:1];
	operationQueue = [[NSOperationQueue alloc] init];
}
</code></pre>

Om du nå skulle ønske å kjøre en operasjon i bakgrunn trenger du bare å legge til en NSInvocationOperation i NSOperationQueue

<pre><code class="objc">operation = [[[NSInvocationOperation alloc] initWithTarget:self selector:@selector(runInBackground) object:nil] autorelease];

[self.operationQueue addOperation:operation];
</code></pre>