# Site Suicide Guide

Episode 1 
Site Suicide Guide: How I met your bad guy

To start with I’d like you to know this is not another ironic text for us to laugh and go. This is the distilled experience, flavoured with failure and painful mistakes, very personal but still well scaled for you to apply and maybe avoid. Or just have your own. Or make your site go through all the points and die. We’ll see. 
In fact, I'm a kind man, feeling happy every time my projects succeed. Feeling pissed off when having troubles with them. In my opinion, there is hardly a coder who does not care.  
That’s why I was badsurpeized by meeting one-typed and easily avoidable problems in different web projects. The only common point is about “lack of time to rewrite”. Rapid development spread its evil karma to the code quality on these projects. The same rapid development that made the projects so popular, fast growing and profitable. 
To tell the truth, I was ready to believe all the issues being caused within some "bad guy" big plan. The one, who does enjoy working with a sick project. Such a team-House MD. Sounds a bit paranoic? Hell yeah!
We all understand there is no "bad guy". The project incorporates a lot of still working temporary architectural solutions, ready to bring problems in future or are already somehow doing this. While the the huge flow of new requirements and tasks leaves no time to refactor the infrastructure, making the code quality to be the least priority at all.
At this very situation we do need that "bad guy" for a nice fight. Someone sitting on your shoulder and whispering what to do next. As we do not play chess here, black are the first to start. And win in the end.

What I want is to guide you through the obvious (or not) points to kill your project. To make your site commit a suicide with the team innocently watching.  Wanna know how to torture the project to death? So, let me show you in my next post. Stay tuned! 

Episode 2
Site Suicide Guide: May the Database be with you
In my intro post I invite you into a marvellous story with an interactive scenario. It’s only you who decide whether the project die in the end of it. So let’s proceed to the first scene. The Database raping.
We all know, to slow down any web project bad guys usually operate with database. DNS-balancing or upstream directive in the nginx-configuration are well-known for easy scaling up almost any other thing, except db. And bad guys are aware of that as well. Want to ask about the clusterization? You’re loading the gun for them with your own hands. First thing bad guys do to any problematic huge db is the clusterization itself. They will rape database to the condition when no single server can operate with it and here is the weapon:
- No needed indices. At all. Let server die under tonnes of unindexed data.
- Unused indices. Let server build exceed indices in the name of entrophy.
- Too heavy rows bc of non-optimized field types and exceed denormalized fields.
- Creating centralized "table of God", that should be heavy enough to make ALTER TABLE impossible;
- Kill bothering db monitoring tool or just mute it's notifications, so really nobody cares about it’s painful screaming.
I personally observe almost ruined database cluster three times: twice for MySQL and once MongoDB. No indices set, no tables’ make up, while the expensive servers are being grouped in the cluster to make some extra work together.
Database suicide situation is commonly spread within the teams who separate backend-developers from the admins/DevOps/NOC. So, bad guys here are programmers themselves, that do not have to care about performance. And admins know nothing about app's business logic, so they have lack of information needed for db optimization. Deaf and blind go hand-in-hand on a traffic road. Could be a funny anecdote, but it’s not. As it’s really dangerous to keep your data in such a moldy state as you are in one step of losing it all. Your orders, clients, SEO page rank may disappear in a sec.
In case you are ready to kill your project in this episode, please, do optimize your code. Genius bad guys can start project optimization without database tuning. This is a perfect way to lead the team on a way to abyss. In fact it is a fake job. 
And this is just the beginning. As there are still ways ahead bad guys usually drive to end up with your project. The most thrilling one (actually n+1) comes in the next episode! 

Episode 3
Site Suicide Guide: Desperate «n+1» Issue
I keep telling you the ways you may harakiri your project. Let’s talk about the database one more time. It appeared to be two major and powerful scenarios of database ruining. Have you ever heard about the “n+1 Issue”? Me either. Until last month. And I swear I could hardly imagine to meet it in a commercial production. But here it is like that I show you in  pseudo-code:
list = db.query('SELECT * FROM products;')
for (item in list){
      orders = db.query('SELECT * FROM orders WHERE product_id = ?;', {product_id: product.id});
      ...
}
This issue could be detected by taking full web server log access and database query log for one period and comparing the size. See 50MB access log turning into 200GB query log - you do have the “n+1 Issue”. What’s next? You’ll have to modify the code, and you’ll definitely meet a lot of work to be done there. This is what a good guy would tell you to. The bad one will just watch your database cluster grow and crush with popcorn and IMAX glasses as it would definitely be a show. 
Here I may quit. Do you want to revive the project? Ok than. Remember a simple axiom: webdev is nothing but pure users’ to database requests transformation and results showing. 
“n+1” issue is rather common for go-programmers with their go-routines and is rarely met in ActiveRecord adepts, php- и js-coders’ projects.
I told you about 2 scenarios. Here is the second one. “n+1” in it’s litteral meaning. 
SELECT p.*, ( SELECT count(*) FROM comments WHERE product_id = p.id) comment_count FROM products p WHERE author_id = ?;
What you see here is NOT one request, this is «n+1» as well. With no LIMIT. Pa ba ba baaa. The thriller in it’s best, is not it? 
You may change this into JOIN (with GROUP BY sub-request) — but it will not solve the issue, although with Index set it may stay alive. In case you’d like to. Oh, the indices has not been discussed yet? So stay tuned, it comes next.


Episode 4
Site Suicide Guide: Big Bang Index Issue 

Having coped with previously described issues you might have thought I have no ideas left? Bad guys have plenty! Here we go with the Indexing issue. 
In fact, it may be easily detected by downloading this script - dev.mysql.com/doc/mysql-utilities/1.5/en/mysqlindexcheck.html or this one www.percona.com/doc/percona-toolkit/2.1/pt-duplicate-key-checker.html, running it and watching the list of unused index. Facing the life-long one, congrats, you have a shabby base and need to check the requests asap or the suicide bell is already ringing. 
If there was an empty list, try to switch on the unindexed query log, just google the way for your db, they differ, and analyze the results. You may get appropriate results and conclude the indexing to be applied accurately for now. Unfortunately almost always the rabbit is exactly here. So roll up sleeves if you’ve got any suspicious numbers. 
Index everything and modify code at a time. 
Another side of medal is over-indexation. Sometimes spare index is far more dangerous than the missing one. Don’t want to kill the site at this point? Ok, ok. If you have the highloaded tables, your #1 task is to minimize the index list. As bad guys are ready to supply some of your with the incredible pain, especially the ones who have RDB table used for queue organizing. It’s the price for ignoring the obvious rule: don’t build the queue on RDB tables. It is perfectly built with known and proven mechanisms, e.g. with PostgreSQL’s “SELECT FOR UPDATE ... SKIP LOCKED;”

Episode 5
Site Suicide Guide: Big Little Tables Issue 

Unused index re-counting and unindexed data makeup accompanied with immense tables lead us to a black hole of nothing.  This will be the sad but true motto for this episode. Unless you’ve already quit the project using the scenarios vividly described in previous ones. 
This very black hole is well treated by the desire of keeping outdated data forever. And the scariest part any screenwriter would adore is that there are plenty of projects who suffered a lot because of their forever-keeping. They do really live with no ability to apply ALTER TABLE in the centralized high-loaded tables of God. If you are ready to deal with your site for some comfortable time, please create an additional base with zip-prefix in sub-tables, use dumps and backups. It’s free of charge.
While bad guys are still sunbathing on your project, you’d better join the good ones and keep an eye on your developer in case the request to 10 tables at a time takes place in production. Unions, Joins and sub-requests with 3d level nesting is nothing but boasting (here the bad guys are dancing lambada). 
Take the fact that your db driver knows nothing about your app logic. It may attempt only to understand how to give you the wanted results from dozens of requested tables. And you are the one who need to know that for sure. So, being the chosen one, do please excite the good guys and try to split your hard queries into a less harmful something. In case you fail, there is always a good guy ready to help you with the db un-crashing. 
When you are in the middle of getting your millions with some wonderful highloaded popular web project - man, you are in trouble. It has already been turned into a complicated asynchronous system you hardly know how to deal with. You might have missed, but your  db tables became the resources, like hardware or money. Just because some operation types are blocked while performing the others. Index re-counting blocks this index and will definitely block one of the re-counts and so on.
For example, any table UPDATE-request will lead to index re-count, but until finished, all the SELECT using this very index will wait. Using immutable tuples in PostgreSQL and will lead to terrible index re-counting even on simple UPDATE-request.
MySQL is not a silver bullet here at all, good guys do know that. Even a well-set high-loaded table will bring one or two of major index. And most of the updates will end in re-counting them all anyway. So, do you still need one table for all the product types? Time for you to think is over. As we’ve prepared something really special for you in our next episode.

Episode 6
Site Suicide Guide: The Raise of the Code 

Any type of code is ugly unless written by you. Even then it’s still ugly or will be in few months.
In fact,the Code world is balanced. If some really beautiful code has appeared, the other one has failed at this very moment. Good news - the ugliest code is not capable to kill the project. But will no doubt make it fat, sick and tired.
This is not what we want for yours? Right? So here are the good guys with CodeSniffer in addition to code review on the first level of code quality. These tools will definitely help you in Code vs shitCode war. But bad guys are also here to show you over-pattern-usage and excess abstraction layers, which are somehow not cut on code reviewing stage. Any additional abstraction layer in your code will raise its supporting cost, add deployment headache and will for sure increase amount of bugs and problems.
Have you heard about the Gang of fours? Be sure they are the reason why you do have several redundant classes in your architecture. Forget the patterns in web. Good guys need nothing but strong SOLID here.
Or singleton, as a vivid example. Do you know the rules? Singleton is usually used for resource axess limitation, when this very axess has already been organized within the non-unique class sample. In case singleton is created from scratch in the majority of cases you’ll get the anti-pattern. For example, Dependency Injection pattern allows you to put mockups while unit-testing and/or build the app with configuration file like ZF 1.x or Laravel 5.x. In other words - being an anti-pattern.
Talking about Repository + Entity-driven db access, you should be aware of it converting into a piece of useless legacy code in static functions. The ActiveRecord pattern on the contrary gives you a clear image of what is to be a state from the very beginning.
Ok, we’ve promised you this all to be an interactive movie, so you may really turn this thriller into an indie film or even romantic story. Follow simple and really helpful SOLID rules and OOP definition encapsulation good guys supply you with, and enjoy the show while KISSing clean DRY code.
While writing the whole new scenario for your super project, just keep in mind there are no bug-less projects. All the tools you have applied or will apply are abstraction layers in architecture and service area. Web-development ia a transforming special effect between user actions and database requests with showing the results. No pattern thinking. No multi-layering. Only magic.

Episode 7
Site Suicide Guide: Doctor Why
Here we are interrogating the Universe with our collection of Whys. This part is will be like talk show, where Jimmy Fallon or Oprah will definitely cry, the audience cry, good guys cry, and bad ones celebrate behind the scenes.
Ready? Go!

Why don’t you use Linux? Yes, the interface is not Brad Pitt and fonts may crush, but all-in-all we are here to web-develop, not to photoshoot. I agree this statement to be a great discussion start, so will pause right here. (Applause. Laugh. Supportive cry)
Why don’t you suggest your developers to set up the new dev-environment themselves? Here bad (and lazy) guys may yel, that  it will definitely take a couple of days for a newbie to do that. Of course, it will. And this will no doubt make them and your team closer to project understanding. And for a team-lead or PM this will uncover the real level of a developer they invite into their project. (Applause. Supportive laugh. Single cries).
Why do you code with warning notification turn off? Good guys vote for  early bug detection! Good guys vote for cost cut! Good guys vote for the excessive QA department reduction! Good guys vote not to sound Trump-like, so here we will pause. (Laugh. Supportive applause. No cry). 
Why do you keep your production without any set integration test system? Run Jenkins and some sort of script once an hour to log in/ sign in/ check mail/ purchase/ sell and panic in case of any problem with a message to your mail/ hipchat or whichever you prefer. It will really save money, nerve and client base. Like Head-&-Shoulders! (Cries and applause transforming into a histeria as someone has recalled Beta-less living).
Why do you keep the whole code massive and configs in web-root? It’s risky, it’s stressful. And good guys do warn you to set “deny from all” everywhere it should be. It’s like open fire cooking: you may get burnt any time and the meal might be done in 20% of cases. Want adrenaline? Visit Vegas! (Laugh and applause).
Why do you invest into site SEO, when it’s load time takes ages? Leave this useless and too expensive procedures for bad guys. Have spare budget? Go and buy books, flowers or Bali tickets. Pleasure guaranteed. (Supportive applause and cry. As everyone has dreamt about Bali, but none had ever been).
Our Why-show-live goes to an end. All these (and many more of your own) whys you may solve with appropriate tools, tech support and good guys  ready to help. What comes next is..

Episode 8
Final
Site Suicide Guide: The team and the city
Bad guys do not talk much about the project. They do not really speak business language, that’s why stay mute. Stay mute and make business owner believe in OOP and be eager for clean code in it. What’s next? Investors’ headache (or even worth) guaranteed. 

Let’s say your project is young but already big enough to deal with investors. They are often people far from IT, who will deny all good guys’  base normalization, CAP-theoremes and other brilliant ideas. They invest money, they want ROI level to grow faster. They may talk to you about...money.
Hardly a team work of your dream. Good guys tell us to choose a pull of tasks, which show the cost of implementation on an updated system will be LESS than task completion and system regular repairing. Bad guys arrived to remind you are not Nelson Mandela, and your networking skills ended in childhood while sharing tracks on the playground? Ok. Step aside, as you may do more harm with an uneven communication. In case you have succeeded, take care to choose the new/ fresh tasks from the list, the old ones will provoke reasonable questions “why now? what have you done all the time before?”, and the Why-live has already been shown previously. 
It’s better to build a proper and well-thought-out project at once. Oh, hello,  Cap!

Your team is not only about outer communication, but also related to an inner management very much. Recalling the Microsoft CEO Bill Gates’  authoritarian management style. He used to loudly argue with his colleagues, shift the due dates to the absurd ones and many more, which resulted into a rapid sometimes idiotic architectural and/or technological decisions. And the users are the ones to suffer, alongside with the developers team. The bad guys blessed an axiom of this management style: “developers like re-writing. We be not do this.” Ok, bad guys. Good ones suggest dialogue as the only possible remedy. No dialogue = no further work. That linear, that simple, that sad.
You are still here? Feel distressed? Great! Never give up! 
Both bad and good guys believe in you. Disregard the vector. Yes, we are living in an imperfect world, with a lot of sorry to say, shit in it. Don’t want to mess up with that? Go home, lay still and watch someone live your interesting and impressive life for you!
You may be right now at the very beginning of a huge intercontinental platform building or something even more fantastic. Either a bold CV line or a temporary income. It’s cool anyway! Use any uncomfortable situation to turn into your profit one. Remember the old good phrase about lemons? Bad guys suggest find someone with salt and tequila and make a great fun. Here good ones agree. As fun may be transformed into a big cooperation, future dream-team and overwhelming results.
And finally, who am I to state all that unshaking? I am one of you, still building, creating, networking and dreaming of a better future, while trying to cope with bad and good guys around. My project is my scary fun blockbuster movie. What will be yours?
