Web Project Survival PlanS


# Easy way to kill your site

I'm doing that famous Joker-face when meeting stupid, easily avoidable issues in different web projects. Various database, any language, business areas or monetization schemes. They were rather about “lack of time and opportunity, no way lack of knowledge”. So sad, so true, but rapid development played evil joke to the projects. It provoked millions of so-to-say trash-code lines colonize the web. 
And I dare say, although those temporary architectural solutions are still working, they will  bring problems in future or are already doing this. The huge flow of new requirements leaves no time to make up the infrastructure, making the code to be the least priority at all. If the described above drives you nostalgic or irritated as you face it right now, you are at your survival plan START point. 
p.s. Dearest thoughtful reader, you may consider this article to be for high load only. But what’s wrong with the high load rates? I suppose this to be ambitious!
So, in my opinion, any effective survival needs a proper plan, I suggest dividing the issues we may meet into 5 major groups. 

Group 1: the base
It’s no secret, all the nastiest web project problems are about database. With the help of DNS-balancing or upstream directive in the nginx-configuration we can easily scale anything, except db. Hello again, my thoughtful reader, with your questioning about the clasterization. That’s the painful point! I've personally observed (literally with my own eyes) almost ruined database cluster already three times in my career. It was twice for MySQL and once MongoDB. No Index set, no tables’ make up, while the expensive servers are being paid for the cluster. And they unfortunately have to cope with the mess of unindexed data and unused index during entropy. 
This group of issues is commonly spread within the companies who separate backend-developers from the admins/DevOps/NOC. 
And it’s really dangerous to keep your database in such a moldy state as you are in one step from losing it all. Your orders, clients, SEO page rank may disappear in a sec. 
It is a “brilliant” solution some apply, while the database is crushing, to imply performance improvement all around and except database. No indices, or even row size optimization. Here I only stay away and wait for the final explosion. Bum! As nothing could already be done now. 
«n+1» Issue
It appeared to be two major ways of database ruining with your own hands. And to tell the truth, some months ago I was absolutely unaware of the one of them. As it really used to take place years ago. Have you ever heard about the “n+1 Issue”? I do recall something like that in my junior developer’s past. I swear I could hardly imagine to meet it in a commercial production. But here it is. For your better understanding I’ll prove it with pseudo-code:

list = db.query('SELECT * FROM products;')
for (item in list){
      orders = db.query('SELECT * FROM orders WHERE product_id = ?;', {product_id: product.id});
      ...
}


You may easily detect the issue by taking full web server log access and database query log for one period and comparing the size. If you see 50MB access log turning into 200GB query log - you do have the “n+1 Issue”. Bad news - you’ll have to modify the code, and you’ll definitely meet a lot of work to be done there. 
Here I stay on a simple axiom: webdev is nothing more than users’ actions to database requests transformation and results showing. That’s all. 
This issue is rather common for go-programmers with their go-routines and is rarely met in ActiveRecord adepts, php- и js-coders’ projects.
We may also put here the requests like:

SELECT p.*, ( SELECT count(*) FROM comments WHERE product_id = p.id) comment_count FROM products p WHERE author_id = ?;

 This is NOT one request, this is «n+1» as well. With no LIMIT as a cake topper. You may change it to JOIN (with GROUP BY sub-request) — but it will not solve the issue, although with Index set it may stay alive. To be honest, these are 2 requests and code joining. You’d better perform it manually, unless your ORM can handle it. Maybe I'll even supply you with the appropriate library if necessary.

Indexation
Unfortunately indexing is being underestimated among my web-developing colleagues. 
In fact, the issue may be easily detected by inspecting db with this script-  https://dev.mysql.com/doc/mysql-utilities/1.5/en/mysqlindexcheck.html or this one https://www.percona.com/doc/percona-toolkit/2.1/pt-duplicate-key-checker.html. Just run it and watch the output-list of unused indices. In case you face the really long list, you have a shabby base and need to check the requests asap. If there was an empty list, try to switch on the unindexed query log (just google "how to log unindexed queries in your-database-type"), and analyze the results. You may get appropriate results and conclude the indexing to be applied accurately for now. Unfortunately this is almost the rabbit's hole itself.
 
You are to roll up sleeves if any of these inspections gave any suspicious numbers. Hard work is coming. To start with, you’ll need to index everything and modify code at a time. For MySQL you can not get “ROWS examined” in EXPLAIN SELECT, so use full query log (long_query_time = 0). This data could be aggregated to get statistics for further work. I do enjoy the sum(Rows Examined) parameter as it really depicts the level of base disturbance for such request types. Alongside with the 95th percentile ratio on “ROWS Examined” vs “ROWS Sent” parameters as it demonstrates the possibility of request types’ optimization. 

You may create the aggregating algorythm yourself or run this ready-to-use one www.percona.com/doc/percona-toolkit/2.2/pt-query-digest.html. I do warn you to be really  careful with that as aggregation mistakes will lead to waste time. Here you may apply the Cartesian doubt - treat any aggregation algorythm as buggy unless you have checked it twice.

And do not forget to clean out spare indices, we do spend resources on rebuilding them. This can be easily seen by someone who ignores rule "do not use RDB table for queue organizing". By the way, queue is perfectly built with known and proven mechanisms, even with PostgreSQL
SELECT FOR UPDATE ... SKIP LOCKED;


Table Size Optimization
As I have already told before, the expensive servers are being used for entrothy enlargement: unused index re-building and unindexed data operating. Now let’s add immense tables and get the real image of Orwell glut utilization war in one web project. 
Top chart of the reasons WHY is headed by the desire of keeping outdated data forever. The  vivid example is social media, which store the deleted messages FOREVER in their main table and in the same base. And the scariest part is that there are plenty of projects who suffered a lot because of their forever-keeping. They ended up with no ability to apply ALTER TABLE in the cetralized high-loaded tables. No wonder you can not support such TNT tables,  any-time-explosion-awaiting mode. To risky and stressful for effective work, is not it?
The solution could be found in additional database with "zip_" prefix, in additional tables, in mysqldump and in backup.


Joins Issue
It may sound rude, but you should keep an eye on your developer in case the request to 10 tables at a time takes place in production in web. Unions, Joins and sub-requests with 3d level nesting is nothing but boasting. It may be used in analytics but never can be in a web part of a more or less loaded project.

Locks are not the least reason as well. Problems become long before you'll see deadlock in your error log. And the locks number could be easily cut with the unified sorting filter like parent_id IN (sorted id list). 
I am totally sure your DBMS is not aware of the business logic of your app. It may attempt to learn how to give you the wanted results from dozens of requested tables. And you are the one who know that. And you are the one who may apply hash-index in php and join two data massives in base using the appropriate lib. In case you doubt, I can help. 

Table Centralization Issue
You already know that all the big interesting web projects turn into complicated asynchronous system. Database and/or tables there become the resources which block one operation types while performing the other. For example, index re-building block this index usage and will definitely block next re-builds.
 
Here I used “resource” term on purpose, although it is rather used to name table hardware characteristics such as CPU, bandwidth and RAM. And you may easily detect the usual or critical lack of these resources with the help of concrete tools like munin/monit/sa+sar/htop or/and experienced admin. It will not take much of your time or money. 

I am not sure why, but it is not common to apply the “resource” term to the relational database's tables, although this is more than obvious. For example, any table UPDATE-request will lead to index rebuild, but until finished, all the SELECT using this very index will wait. Using immutable tuples in PostgreSQL and UPDATE-request will lead to all the index recounting in the table. 
UP1: as terrier admits, PostgeSQL is only fitting the ones, who may properly deal with it. As well-known Uber failed and feels awkward. Still not a big surprise, when you keep in mind the Einstein's statement about the infinity of the universe. 
When using MySQL the situation looks better, but even well-set high-loaded table will bring one or two of major index. And most of the updates will end in re-counting them all anyway. So, why do you need one table for all the product types? 
There are projects who preferred to concentrate on table centralization and (micro)Service centralization as well, while building the architecture. They totally ignore the axiom of experience and logics “do not incapsulate into services, do incapsulate into classes”. All these will result into a sad unscalable bottleneck. 
The solution could be found in creating products2 table, setting the non-crossing definitions for the primary key in the original table. And enjoy the show. In case one of the popular products differs by its structure from the rest, you will need to normalize the database and attribute it into a separate table. 

Group 2: code

In any project in any language code is always terrible, even written by you. Such an axiom in developers’ world. In fact, it’s a joke, you know. As there could not possible be so much bad and so much brilliant code at a time. I personally believe in some kind of a Conservation law like with mass or energy or money: if some really beautiful code has appeared, the other one has failed at this very moment. 

If you wear a perfectionist hat, I do recommend saving time and resources and using CodeSniffer in addition to code review on the first level of code quality. These tools are supposed to satisfy your hat. Oops. Your inner perfectionist and your project as well. 

The deeper the worse. Over-pattern-usage and excess abstraction layers are somehow not cut on code reviewing stage. 

Let's talk about singleton. Do you know the rules? So I’ll tell you a bit. Singleton is usually used for resource axess limitation, when this very axess has already been organized within the non-unique class sample. In case singleton is created from scratch in the majority of cases you’ll get the anti-pattern. For example, Dependency Injection pattern allows you to put mockups while unit-testing and/or build the app with configuration file like ZF 1.x. in other words - being an anti-pattern. 

When we talk about Repository + Entity-driven db access, it too often falls into a useless legacy code. Repositories are used in stateless mode like any group of similar functions, sometimes static or as a singletons. This may be the reason why code became spaghetti. The ActiveRecord pattern on the contrary gives you a clear image of what is to be a state from the very beginning. 

And about 3d party soft. Being a developer, do you believe in bug-less projects? All the tools you apply are actually abstraction layers in architecture and service area. Do keep in mind web-development is just a transformation of user actions to database requests with showing the results. 
The solution for the code issue, in my opinion, is to show the full power of well-built interface hierarchy which helps in creating easily appending apps. And please remember excessive abstraction layers to be harmful. No pattern thinking allowed. Prefer using SOLID rules and OOP basic principles instead of patterns, this will make your code simple and sweet.


As for the frontend
Do you hear lot of “javascript's this”-jokes in your team? I do. And every time I face 2000th-style front-end development with dozens of plug-in files and/or JQuery with ExtJS and/or selfmade MVC, I see an absolutely unsupported over-costy at any modification and lame code.
 
The solution is rather obvious, do use es2015 javascript and components-oriented SPA design. The changes should be applied gradually, an evolutionary move starting with routing. And mind not to use TypeScript, as from my experience, it confronts the very JS idea and leads to an anarchy and high-quality rapid-development mess. 


You are still here? Feel distressed? Never give up!
Yes, we are living in an imperfect world, with a lot of sorry to say, office plankton. People who were hired to struggle with JIRA and bring coffee to their team leads. The more chain parts we have, the more firm and stable this chain is. But still - never give up! 
Just imagine, you may be now at the very beginning of a huge intercontinental platform building or something even more fantastic. Either a bold CV line or a temporary income. Use any uncomfortable situation to turn into your profit one. 
And of course, who am I to state all that unshaking. I am one of you, still building, creating, networking and dreaming of a better future. I am also open to discuss and do welcome any sort of critics, comments or recommendations. Go ahead! 


