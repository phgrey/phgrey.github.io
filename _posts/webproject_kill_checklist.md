# Easy way to kill your site

Recently I was badsurpeized by meeting one-typed and easily solved issues in absolutely different web projects. And they all were not about the base, languages, business areas or monetization schemes. They were rather about “lack of time and opportunity, no way lack of knowledge”. So sad, so true, but rapid development played evil joke to the projects. It provoked millions of so-to-say trash-code lines colonize the web. 
And I dare say, although those temporary architectural solutions are still working, they will  bring problems in future or are already doing this. The huge flow of new demands and tasks leaves no time to make up the infrastructure, making the code to be the least priority at all. If the described above drives you nostalgic or irritated as you face it right now, you are at your survival plan START point. 
p.s. Dearest thoughtful reader, you may consider this article to be for high load only. But what’s wrong with the high load rates? I suppose this to be ambitious!
So, in my opinion, any effective survival needs a proper plan, I suggest dividing the issues we may meet into 5 major groups. 

Group 1: the base
It’s no secret, all the nastiest web project problems are somehow connected with database. With the help if DNS-balancing or upstream directive in the nginx-configuration we can easily paste it together. Hello again, my thoughtful reader,  with your questioning about the clasterization. That’s the point! I personally observe almost ruined database cluster already three times in my career. It was twice for MySQL and once MongoDB. No Index set, no tables’ make up, while the expensive servers are being paid for the cluster. And they unfortunately have to cope with the mess of unindexed data and unused index during entropy. 
This group of issues is commonly spread within the teams who separate backend-developers from the admins/DevOps/NOC. 
And it’s really dangerous to keep your base in such a moldy state as you are in one step of losing it all. Your orders, clients, SEO page rank may disappear in a sec. 
There is a “brilliant” solution some apply, while the database is crushing, is to imply performance improvement. Here I only stay away and wait for the final explosion. Bum! As nothing could already be done now. 
«n+1» Issue
It appeared to be two major ways of database ruining with your own hands. And to tell the truth, some months ago I was absolutely unaware of the one of them. As it really used to take place ages ago. Have you ever heard about the “n+1 Issue”? I do recall something like that in my junior developer’s past. I swear I could hardly imagine to meet it in a commercial production. But here it is. For your better understanding I’ll prove it with pseudo-code:

list = db.query('SELECT * FROM products;')
for (item in list){
      orders = db.query('SELECT * FROM orders WHERE product_id = ?;', {product_id: product.id});
      ...
}


You may easily detect the issue by taking full web server log access and database query log for one period and comparing the size. If you see 50MB access log turning into 200GB query log - you do have the “n+1 Issue”. Bad news - you’ll have to modify the code, and you’ll definitely meet a lot of work to be done there. 
Here I stay on a simple axiom: webdev is nothing like pure users’ database requests transformation and results showing. That’s all. 
This issue is rather common for go-programmers with their go-routines and is rarely  met in ActiveRecord adepts, php- и js-coders’ projects.
We may also put here the requests like that:
SELECT p.*, ( SELECT count(*) FROM comments WHERE product_id = p.id) comment_count FROM products p WHERE author_id = ?;



 This is NOT one request, this is «n+1» as well. With no LIMIT as a cake topper. You may change it to JOIN (with GROUP BY sub-request) — but it will not solve the issue, although with Index set it may stay alive. To be honest, these are 2 requests and code joining. Perform it manually, unless your ORM is not capable to. I may even supply you with the appropriate library.
Index Issue
Unfortunately Indexing is being underestimated among my web-developing colleagues. 
In fact, the issue may be easily detected by downloading this dev.mysql.com/doc/mysql-utilities/1.5/en/mysqlindexcheck.html or this one www.percona.com/doc/percona-toolkit/2.1/pt-duplicate-key-checker.html, running it and watching the list of unused index. In case you face the really long one, you have a shabby base and need to check the requests asap. If there was an empty list, try to switch on the unindexed query log, just google the base type you have as they differ, and analyze the results. You may get appropriate results and conclude the indexing to be applied accurately for now. Unfortunately in 90% the issue is hiding here. 
You are to roll up sleeves if any of the suggested above gave any suspicious numbers. Hard work is coming. To start with, you’ll need to index everything and modify code at a time. For MySQL you can not get “ROWS examined” in EXPLAIN SELECT, so use full query log (long_query_time = 0). This data could be aggregated to get the statistics for further work. I do enjoy the sum(Rows Examined) parameter as it really depicts the level of base disturbance for such request types. Alongside with the 95th percenthile ratio on “ROWS Examined” vs “ROWS Sent” parameters as it demonstrates the possibility of request types’ optimization. 
You may create the aggregating algorythm yourself or run this ready-to-use one www.percona.com/doc/percona-toolkit/2.2/pt-query-digest.html. I do warn you to be specially careful with that as aggregation mistakes will lead to waste of your time. Here you may apply the Cartesian doubt - any aggreagtion algorythm unless you have personally experienced it working properly is potencially false one. 
I insist on keepin in mind the fact that spare index is more dangerous than the missing one. Your #1 task is to minimize the used index list. Your #2 task is not to use RDB table for queue organizing. As queue is perfectly built with known and proven mechanisms, even with PostgreSQL
SELECT FOR UPDATE ... SKIP LOCKED;
Small remarque for php-developers who keep ignoring index, I’ll publish the article devoted to “Index Dos and Dont’s” for you in the nearest future. 
Table Size Optimization
As I have already told before, the expensive servers are being used for entrothy enlargment: unused index re-counting and unindexed data makeup. Now let’s add immense tables and get the real image of Orwell glut utilization war in one web project. 
Top chart of the reasons WHY is headed by the desire of keeping outdated data forever. The  vivd example is social media, which store the deleted messages FOREVER in their main table and in the same base. And the scariest part is that there are plenty of projects who suffered a lot because of their forever-keeping. They ended up with no ability to apply ALTER TABLE in the cetralized high-loaded tables. No wonder you can not support such TNT tables, waing for the explosion any time. To risky for effective work, is not it?
The solution could be found in additional base with zip-prefix, in additional tables, in mysqldump and in backup.
Joins Issue

It may sound rude, but you should keep an eye on your developer in case the request to 10 tables at a time takes place in production in web. Unions, Joins and sub-requsts with 3d level nesting is nothing but boasting. Or for personal search in base. But they are far from database communication in a more or less loaded project.   
Locks are not the least reason as well. They are convering into issues long before you get deadlock in your error log. And the locks number could be easily cut with the unified sorting filter like parent_id IN (sorted id list). 
I am 99% sure your DBMS is not aware of the business logic of your app. It may attempt to learn how to give you the wanted results from dozens of requested tables. And you are the one who know that. And you are the one who may apply hash-index in php and join two data massives in base using the appropriate lib. In case you doubt, I am here to help. 
Table Centralization Issue
You already know that  all the huge web projects turn into  complicated asynchronous system. Where base and/or tables become the resources which block one operation types while performing the other. For example, index re-counting block this index and will definitely block one of the re-counts. 
Here I used “resource” term on purpose, although it is rather used to name table hardware characteristics such as CPU, bandwidth and RAM. And you may easily detect the usual or critical lack of these resources with the help of concrete tools like munin/monit/sa+sar/htop or/and experienced admin. It will not take much of your time or money. 
I am not sure why, but it is not common to apply the “resource” term to the relational database, although this is more than obvious. For example, any table UPDATE-request will lead to index re-count, but untill finished, all the SELECT using this very index will fail. Using immutable tuples in PostgreSQL and UPDATE-request will lead to all the index re-counting in the table. 
UP1: as terrier admits, PostgeSQL is only fitting the ones, who may properly deal with it. As well-known Uber failed and feels awkward. Still not a big surprise, when you keep in mind the Einstein's statement about the infinity of the universe. 
When using MySQL the situation looks better, but even well-set high-loaded table will bring one or two of major index. And most of the updates will end in re-counting them all anyway. So, why do you need one table for all the product types? 
There are projects who preffered to concentrate on table centralization and (micro)Service centralization as well, while building the architecture. They totally ignore the axiom of experiance and logics “do not incapsulate into services, do incapsulate into classes”. All these will result into a sad bottleneck mess with no ability to repair the system when the load is growing. 
The solution could be found in creating products2 table, setting the non-crossing definitions for the first key in the original table. And enjoy the show. In case one of the popular products differs by its structure from the rest, you will need to normalize the database and attribute it into a separate table. 

Group 2: code (coding?) 

Code is ugly unless written by you. Such an axiom in developers’ world. In fact, it’s a joke, you know. As there could not possible be so much bad and so much brilliant code at a time. I personally believe in some kind of a Conservation law like with mass or energy or money: if some really beautiful code has appeared, the other one has failed at this very moment. 
If you wear a perfectionist hat, I do recommend to save time and resources and use CodeSniffer in addition to code review on the first level of code quality. These tools are supposed to satisfy your hat. Oops. Your inner perfectionist and your project as well. 
The deeper the worse. Over-pattern-usage and excess abstraction layers are somehow not cut on code reviewing stage. 
Another important moment in code-talk is the usage of singleton. Do you know the rules? So I’ll tell you a bit. Singleton is usually used for resource axess limitation, when this very axess has already been organized within the non-unique class sample. In case singleton is created from scratch in the majority of cases you’ll get the anti-pattern. For example, Dependency Injection pattern allows you to put mockups while unit-testing and/or build the app with configuration file like ZF 1.x. in other words - being an anti-pattern. 
When we talk about Repository + Entity-driven db access, from my experience I should unfortunately admit it to convert into a useless legacy code. As repositories are used in stateless regime like any group of similar functions this may be the reason why. The ActiveRecord pattern on the contrary gives you a clear image of what is to be a state from the very beginning. 
One more interesting fact, when we are talking about code, is that simple and really helpful SOLID rules and OOP definition encapsulation are being totally ignored. 
Being a developer, do you believe in bug-less projects? All the tools you apply are actually abstraction layers in architecture and service area. Do keep in mind web-development to be a transforming magic between user actions and database requests with showing the results. 
The solution for the code issue, in my opinion, is to show the full power of well-built interface hierarchy which helps in creating easily appending apps. And please remember excessive abstraction layers to be harmful. No pattern thinking allowed. 

Group 3: Front-end
Do you recall “this”-jokes about javascript developers? I do. And every time I face 2000th-style front-end development with dozens of plug-in files and/or JQuery with ExtJS and/or selfmade MVC, I see an absolutely unsupported over-costy at any modification and lame code. 
The solution is rather obvious, do use es2015 javascript, create one-page app with accurate routing, single entry point. The changes should be applied gradually, an evolutionary move starting with routing. And mind not to use TypeScript, as from my experience, it confronts the very JS idea and leads to an anarchy and high-quality rapid-development mess. 

Group 4: why? 
I can hardly understand why...
Why don’t you use Linux? Although the interface is ugly and fonts may crush, but all-in-all we are here to web-develop, not to design. I agree this statement to be a great discussion start, so will pause right here. 

Why do not you set the new dev-environment yourself? Yes, it will definitely take a couple of days for a newbie to do that, but this will no doubt make him closer to project understanding as well. And for a team-lead or PM this will uncover the real level of a developer they invite into their project. 
Why do you code with warning notification turn off? I vote for the early bug detection and cost cut for the excessive QA department. 
Why do you work without beta? I can not imagine zero-time-deployment and proper testing without beta at all. 
Why do you keep your production without any set integration test system? I do insist on running Jenkins and some sort of script once an hour to log in/ sign in/ check mail/ purchase/ sell and panic in case of any problem with a message to your mail/ hipchat or whichever you prefer. It will really save your money, nerve and client base. 
Why do you keep the whole code massive and configues in web-route? It’s rather risky, I should say. And I am not sure you’ve set “deny from all” everywhere it should be.
Why do you invest into site SEO, when it’s load time takes several seconds? This is useless and too expensive. 
Here the why-group ends. All these whys you may solve with appropriate tools and tech support. What comes next is..
Group 5: the team
I am pity to say, but there are issues without any possible (positive) solution. 
Networking issue
When you deal with investors, they will deny all your base normalization, CAP-theoremes and other IT treasures. They invest money, they want ROI level to grow faster. They may talk to you about terms and...money. 
The solution I would apply is to choose a pull of tasks, make the client understand the cost of implementation on an updated system will be LESS than task completion and system regular repair one. You may be far from a good networker. So, stay aside, as you may do more harm with uneven communication. In case you have succeeded, I recommend choosing the new/ fresh tasks from the list, the old ones will provoke reasonable questions “why now? what have you done all the time before?”
And yes, it’s better to build a proper and well-thought-out project at once. 
Management issue
Let’s agree, that any poor code was not created by itself. All of us recall the Miscosoft SEO Bill GAtes authoritarian management style. He used to loudly argue with his colleagues, shift the due dates to the absurd ones and many more, which resulted into a rapid sometimes idiotic architectural and/or technological decisions. And the users are the ones to suffer, alongside with the developers team. 
The axiom of the adepts of this management style sounds like “developers like re-writing. We will not do this.” 
I do not see a positive solution here rather than to dialogue. If there is no possibility to have an effective communication, you’ll better search for another project. 
The other side of the coin is specialist centralization. There are people who keep control over too many tasks and projects, so you have to wait for a simple solution/ resolution for ages. Bottleneck as it is. But it may be solved, if agreed, with microteams creation. 
You are still here? Feel distressed? Never give up!
Yes, we are living in an imperfect world, with a lot of sorry to say, office plankton. People who were hired to struggle with JIRA and bring coffee to their team leads. The more chain parts we have, the more firm and stable this chain is. But still - never give up! 
Just imagine, you may be now at the very beginning of a huge intercontinental platform building or something even more fantastic. Either a bold CV line or a temporary income. Use any uncomfortable situation to turn into your profit one. 
And of course, who am I to state all that unshaking. I am one of you, still building, creating, networking and dreaming of a better future. I am also open to discuss and do welcome any sort of critics, comments or recommendations. Go ahead! 
