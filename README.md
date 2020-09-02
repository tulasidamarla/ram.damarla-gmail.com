Search engine characteristics
-
Search engines like Solr are optimized to handle data exhibiting four main characteristics:<br>
- Text-centric<br>
- Read-dominant<br>
- Document-oriented<br>
- Flexible schema

Common Search engine use cases
-

Basic Keyword search
-
- Users don't want to fill some complex form to get the results. In general, users want to type in a few simple keywords and get back great results.

- To provide a great user experience the following few issues must be addressed.
  - Relevant results must be returned quickly, within a second or less in most cases.
  - Spelling correction is needed in case the user misspells some of the query terms.
  - Autosuggestions save keystrokes, particularly for mobile applications.
  - Synonyms of query terms must be recognized.
  - Documents containing linguistic variations of query terms must be matched.
  - Phrase handling is needed; that is, does the user want documents matching all words or any of the words in a phrase.
  - Queries with common words like “a,” “an,” “of,” and “the” must be handled properly.
  - The user must have a way to see more results if the top results aren’t satisfactory.

RANKED RETRIEVAL
-
- With SQL query to an RDBMS a row either matches a query or it doesn’t, and results are sorted based on one or more of the columns. 
- A search engine returns documents sorted in descending order by a score that indicates the strength of the match of the document to the query. 
- Ranking documents by relevancy is important because modern search engines store large volumes of data. 
- Without ranking users can become overloaded with results with no clear way of navigate them.

BEYOND KEYWORD SEARCH
-
- Typically users don't have knowledge of what information is present in your system. 
- A good search engine helps users narrow in on their information needs.
- To return documents from an initial query, as well as tools to help users refine their search. For example, categorize search results using document features to allow users to narrow down their results. This is known as faceted search, and it’s one of the main strengths of Solr.

DON’T USE A SEARCH ENGINE TO
-
- Retrieving huge no of records: Search engines are designed to return a small set of documents per query, usually 10 to 100. 
  - pagination can be used for next set of matching records. But, retrieving all records at once reconstructing a huge no of documents from the underlying index structure will be extremely slow, as engines like Solr store fields on disk.
- Search engines aren’t the place for querying across relationships between documents. 
  - Solr does support querying using a parent-child relationship, but doesn’t provide support for navigating complex relational structures as is possible with SQL.
- No provision for document level security. If fine-grained permissions on documents is needed, then it has to be handled outside of the search engine i.e application.

Features overview
-----------------
Solr provides a number of important features that help you deliver a search solution that’s easy to use, intuitive, and powerful. It provides the following features.

User experience
---------------
■ Pagination and sorting
■ Faceting
■ Autosuggest
■ Spell-checking
■ Hit highlighting
■ Geospatial search

Data Modelling
--------------
■ Result grouping/field collapsing
■ Flexible query support
■ Joins
■ Document clustering
■ Importing rich document formats such as PDF and Word
■ Importing data from relational databases
■ Multilingual support

Solr terminology
----------------

Index
-----
Like an index in book, solr index also contains content. By adding content to an index, it can be used for search later.

Core
----
A core is composed of a set of configuration files, Lucene index files, and Solr’s transaction log. A core typically indexes documents of one type.(It is like a table in DB)

A solr server can have more than one core.

Shard
-----
If an index becomes large it can be split across multple nodes/servers. Each node is a shard.

Replica
-------
For increasing search performance an index of a shard can be replicated into multiple nodes.

Collection
----------
Collection is a term which only has meaning in the context of solr cluster, in which a single index is split across multiple servers. In other words it's a logical grouping of data available in different shards of a single core. 

Note: In case of single node solr server, A collection is nothing but a core.

Exploring Solr query
--------------------
Sample solr search query
------------------------
http://localhost:8983/solr/collection1/select?
q=iPod&
fq=manu:Belkin&
sort=price asc&
fl=name,price,features,score&
df=text&
wt=xml&
start=0&rows=10&echoParams=all

q -> main query
fq -> filter query
sort -> Sort results by the price field, ascending (lowest price on top).
fl -> Specifies which fields to return for each document in the results.
df -> search in the fields(text means look in all fields)
wt -> Response-writer type, such as XML, CSV, or JSON.
echoParams --> To see all the query paramters

Ranked Retrieval
----------------
The key differentiator between Solr’s query processing and that of a database or other NoSQL data store is ranked retrieval. For ex, if you search using a query "ipod power", solr looks for documents that contain both words ipod or power. That means it gives equal preference to the terms ipod and query. 

If we use the boost query, with the search "ipod power^2", you are mentioning the term power is twice as important to the term ipod. Now, you may get different order of the results based on the ranking.

The inverted index
------------------
Solr uses Lucene’s inverted index to power its fast searching capabilities, as well as many of the additional bells and whistles it provides at query time.

While a traditional database representation of multiple documents would contain a document’s ID mapped to one or more content fields containing all of the words/terms
in that document, an inverted index inverts this model and maps each word/term in the corpus to all of the documents in which it appears.

Here is an example of how lucene inverted index works.

Original documents								Lucene’s inverted index	
Doc #	Content field							Term		Doc#				Term position
1	A Fun Guide to Cooking						a			1,3,4,5,6,7,8		1
2	Decorating Your Home						becoming	8					4
3	How to Raise a Child						beginner’s	6					5
4	Buying a New Car							buy			9					1
5	Buying a New Home							buying		4,5,6				2
6	The Beginner’s Guide to Buying a House		car			4					4
7	Purchasing a Home							child		3					5
8	Becoming a New Home Owner					cooking		1					1
9	How to Buy Your First House 				decorating	2					3
												first		9					2
												fun			1					5
												home		2,5,7,8				4
												house		6,9					6
												new			4,5,8				1

Note: Term position is the relative position of the terms with in a document.
Note:Original input text was split on spaces and that each term was transformed into lowercase text before being inserted into the inverted index. It is worth noting that many additional text transformations are possible, not only these simple ones; terms can be modified, added, or removed during the content-analysis process.

Two important details should be noted about the inverted index:
■ All terms in the index map to one or more documents.
■ Terms in the inverted index are sorted in ascending lexicographical order.

Terms, phrases, and Boolean logic
---------------------------------
Lets look for the following search critierias.

■ Search for two different terms, new and house, requiring both to match
■ Search for two different terms, new and house, requiring only one to match
■ Search for the exact phrase "new house"

REQUIRED TERMS
--------------
If the query into multiple terms and requiring them all to match, then there are two ways to write query.
a)+new +house
b)new AND house (This gets parsed into the above)

OPTIONAL TERMS
--------------
If either the part of the query preceding or the part of the query following it is required to exist in any documents matched, then query can be written in the following two ways.
a)new house
b)new OR house(OR is the default operator)

NEGATED TERMS
-------------
It’s also possible to require that some query parts not exist in any matched documents through either of the following equivalent queries:
■ new house –rental
■ new house NOT rental

Note: If we want to change the default operator, we can do that using the below query.
/select?q=new house&q.op=AND

PHRASES
-------
Solr can also search for phrases,ensuring that multiple terms appear together in order. For ex,
■ "new home" OR "new house"
■ "3 bedrooms" AND "walk in closet" AND "granite countertops"

GROUPED EXPRESSIONS
-------------------
The Solr query syntax can represent arbitrarily complex queries through grouping terms using parentheses, as in the following examples:
■ New AND (house OR (home NOT improvement NOT depot NOT grown))
■ (+(buying purchasing -renting) +(home house residence –(+property - bedroom)))

Finding set of documents
------------------------
If a customer now passes in a query of new home, how exactly is Solr able to find documents matching that query, given the preceding inverted index?
Ans: From the above index it has the following results.
	Term 	Document
	home	2,5,7,8
	new		4,5,8

query
-----
new home -> returns union of the two results. i.e. 2,4,5,7,8.
new AND home -> returns intersection of the two results. i.e. 5,8.
-new +home -> 4,8
new -home -> 4

Phrase queries and term positions
---------------------------------
Each term in a phrase query is still looked up in the Lucene index individually, as if the query new home had been submitted instead of "new home". The term positions will help.

Term	Document	Position
home 	5 			4
		8			4
new		5			3
		8			3
		
In the above inverted index, Document 5 has term "new" at position 3 and "home" at position 4. 

Fuzzy matching
--------------
Fuzzy matching is defined as the ability to perform inexact matches on terms in the search index. For example, someone may want to search for any words that start with a particular prefix (known as wildcard searching), may want to find spelling variations within one or two characters (known as fuzzy searching or edit distance searching), or may want to match two terms within some maximum distance of each other (known as proximity searching).

WILDCARD SEARCHING
------------------
you can use the asterisk (*) wildcard character to perform wildcard search. For ex,

■ Query: offi* Matches office, officer, official, and so on

■ Query: off*r Matches offer, officer, officiator, and so on

If you want to match only a single character, you can make use of the question mark (?) for this purpose:

■ Query: off?r Matches offer, but not officer


Leading wildcards
-----------------
The more characters you specify at the beginning of the term before the wildcard, the faster the query should run. For example, the query engineer* will not be expensive (because it matches few terms in the inverted index), but the query e* will be expensive, as it matches all terms beginning with the letter e.

Executing a leading wildcard query is an expensive operation. If you needed to match all terms ending in ing (like caring, liking, and smiling), for example, this could cause
major performance issues.

■ Query: *ing

If you need to be able to search using these leading wildcards, a faster solution exists, but it requires additional configuration. The solution is achieved by adding ReversedWildcardFilterFactory to your field type’s analysis chain.ReversedWildcardFilterFactory works by double-inserting the indexed content in the Solr index (once for the text of each term, and once for the reversed text of each term):

■ Index: caring liking smiling
#gnirac #gnikil #gnilims

Note:Important point to note about wildcard searching is that wildcards are only meant to work on individual search terms, not on phrase searches, as demonstrated by the following example:

■ Works: softwar* eng?neering
■ Does not work: "softwar* eng?neering"

RANGE SEARCHING
---------------
If you only wanted to match documents created in the six months between February 2, 2012, and August 2, 2012, you could perform the following search:

■ Query: created:[2012-02-01T00:00.0Z TO 2012-08-02T00:00.0Z]

This range query format also works on other field types:

■ Query: yearsOld:[18 TO 21] Matches 18, 19, 20, 21
■ Query: title:[boat TO boulder] Matches boat, boil, book, boulder, etc.
■ Query: price:[12.99 TO 14.99] Matches 12.99, 13.000009, 14.99, etc.

Each of these range queries surrounds the range with square brackets, which is the “inclusive” range syntax. Solr also supports exclusive range searching through the use
of curly braces:

■ Query: yearsOld:{18 TO 21} Matches 19 and 20 but not 18 or 21

Solr also provides the ability to mix and match inclusive and exclusive bounds:

■ Query: yearsOld:[18 TO 21} Matches 18, 19, 20, but not 21

Note: If we have to create text field containing integers, those integers would be found in the following order.
1,11,111,12,120,13 etc.

So, better choose the data types carefully when used in range queries or sorting queries.

FUZZY/EDIT-DISTANCE SEARCHING
-----------------------------
Solr provides the ability to handle character variations using edit-distance measurements based upon Damerau-Levenshtein distances, which account for more than 80% of all human misspellings. Solr achieves these fuzzy edit-distance searches through the use of the tilde (~) character as follows:

■ Query: administrator~ Matches: adminstrator, administrater, administratior, and so forth.

This query matches both the original term (administrator) and any other terms within two edit distances of the original term. An edit distance is defined as an insertion, a deletion, a substitution, or a transposition of characters. The term adminstrator (missing the “i” in the sixth position) is one edit distance away from administrator
because it has one character deletion. Likewise the term sadministrator would be one edit distance away because it has one insertion (the “s” that was prepended), and
the term administratro would also be one edit distance away because it has transposed the last two characters (“or” became “ro”).

It’s also possible to modify the strictness of edit-distance searches to allow matching of terms with any edit distance:
■ Query: administrator~1 Matches within one edit distance.
■ Query: administrator~2 Matches within two edit distances. (This is the default if no edit distance is provided.)
■ Query: administrator~N Matches within N edit distances.

Note:
Please note that any edit distances requested above two will become increasingly slower and will be more likely to match unexpected terms.

PROXIMITY SEARCHING
-------------------
Edit distance searching can also be applied between terms for variations of phrases. For example, if you want to findout executives in the company. One way to try this is 
with the query

■ Query: "chief executive officer" OR "chief financial officer" OR "chief marketing officer" OR "chief technology officer" OR …

This query assume us to know all possibilities. Other possibility is with the query:

■ Query: chief AND officer

This query matches document that contains both of those words anywhere in the document.

Note:
The solution to above problem is proximity searching. Here are the examples.

■ Query: "chief officer"~1
– Meaning: chief and officer must be a maximum of one position away.
– Examples: "chief executive officer", "chief financial officer"

■ Query: "chief officer"~2
– Meaning: chief and officer must be a maximum of two edit distances away.
– Examples: "chief business development officer","officer chief"

■ Query: "chief officer"~N
– Meaning: Finds chief within N positions of officer.

Note:
Solr’s proximity searching is not a true use of edit distance because it requires all specified terms to exist, whereas a true edit distance would also allow for substitutions and deletions (as you saw with fuzzy searching on a single term). That means you may have also noticed that it required a phrase slop of 2 to be specified ("chief officer"~2)
in order to match the text officer chief. This is because the first edit is to move the terms chief and officer into the same position; the second edit is to move chief one more position to come before officer.

Relevancy
---------
Relevancy is calculated using the formula

		Score =(q,d)Σt in q ( tf(t in d) • idf(t)2• t.getBoost() • norm(t,d) ) • coord(q,d) • queryNorm(q)

Where:
t = term; d = document; q = query; f = field
tf(t in d) = numTermOccurrencesInDocument
idf(t) = 1 + log (numDocs / (docFreq +1))
coord(q,d) = numTermsInDocumentFromQuery / numTermsInQuery
queryNorm(q) = 1 / (sumOfSquaredWeights )
sumOfSquaredWeights = q.getBoost()2• Σ ( idf(t) • t.getBoost() )2
norm(t,d) = d.getBoost() • lengthNorm(f) • f.getBoost()

Term frequency(tf)
------------------
The more times the search term appears within a document, the more relevant that document is considered. It’s not likely to be the case that 10 appearances of a term make the document 10 times more relevant, however, so tf is calculated using the square root of the number of times the search term appears within the document, in order to diminish the additional contribution to the relevancy score for each subsequent appearance of the search term.

Inverse document frequency(idf)
-------------------------------
Common sense would indicate that words that are more rare across all documents are likely to be better matches for a query than terms that are more common. For example consider the query "The Cat in the Hat". The terms cat and hat are likely to be better matches than The and in.

Inverse document frequency (idf), a measure of how “rare” a search term is, is calculated by finding the document frequency (how many total documents the search term appears within), and calculating its inverse (see the full formula in figure 3.7 for the calculation).

Likewise, if someone were searching for a profile for an experienced Solr development team lead across a large collection of resumes, we wouldn’t expect documents to rank higher that best match the words an, team, or experienced.

Term frequency and inverse document frequency, when multiplied together in the relevancy calculation, provide a nice counterbalance. The term frequency elevates terms that appear multiple times within a document, whereas the inverse document frequency penalizes those terms that appear commonly across many documents.Therefore, common words in the English language such as the, an, and of ultimately yield low scores, even though they may appear many times in any given document.

Boosting
--------
It is not necessary to leave all aspects of your relevancy calculations up to Solr. If you have domain knowledge about your content—you know that certain fields or terms are
more (or less) important than others—you can supply boosts at either indexing time or query time to ensure that the weights of those fields or terms are adjusted accordingly.

Query-time boosting, the most flexible and easiest to understand form of boosting, uses the following syntax:

■ Query: title:(solr in action)^2.5 description:(solr in action)

This example provides a boost of 2.5 to the search phrase in the title field, while providing the default boost of 1.0 to the description field.Unless otherwise specified, all
terms receive a default boost of 1.0

Note:
In addition to query-time boosting, it’s possible to boost documents or fields within documents at index time. These boosts are factored into the field norm.

Normalization factors
---------------------
The default Solr relevancy formula calculates three kinds of normalization factors (norms): field norms, query norms, and the coord factor.

FIELD NORMS
-----------
The field normalization factor (field norm) is a combination of factors describing the importance of a particular field on a per-document basis. Field norms are calculated
at index time and are represented as an additional byte per field in the Solr index.This byte packs a lot of information: the boost set on the document when indexed,
the boost set on the field when indexed, and a length normalization factor that penalizes longer documents and helps shorter documents (under the assumption that finding
any given keyword in a longer document is more likely and therefore less relevant). It uses the following formula.

norm(t,d) = d.getBoost() • lengthNorm(f) • f.getBoost()

Note:The d.getBoost() component represents the boost applied to the document when it’s sent to Solr, and the f.getBoost() component represents the boost applied to the field for which the norm is being calculated.The length norm is computed by taking the square root of the number of terms in the field for which it is calculated.

QUERY NORMS
-----------
Query norm merely serves as a normalization factor to attempt to make scores between queries comparable. It uses the sum of the squared weights for each of the query terms to generate this factor, which is multiplied with the rest of the relevancy score to normalize it.

THE COORD FACTOR
----------------
Its role is to measure how much of the query each document matches. Let’s say you perform the following search:

■ Query: Accountant AND ("San Francisco" OR "New York" OR "Paris")

You may prefer to find an accountant with offices in Sanfranciso or new york or paris. If an account has offices in all 3, i.e. a document matches all of the four terms(account, san franciso, new york and paris), it's coord factor is 4/4. If 3 matches it's coord factor is 3/4, if two matches 2/4 and if only one matches 1/4.

Precision and Recall
--------------------
Precision answers the questions “Were the documents that came back the ones I was looking for?”

Precision is defined as (between 0.0 and 1.0) 
	# Correct Matches / # Total Results Returned

For example, if a search query(say "buy new home") returns 3 documents. If all 3 documents are relavant, then precision of the query is 3/3 i.e. 1. If search returns 6 results and only 3 of them match, then precision is 0.5.

Recall
------
Recall is a measure of how thorough the search results are. Recall is answering the question: “How many of the correct documents were returned?”

More technically, Recall is defined as # Correct Matches / (# Correct Matches + # Missed Matches)

Precision is high when the results returned are correct; Recall is high when the correct results are present. Recall does not care that all of the results are correct. Precision does not care that all of the results are present.

Example for Precision and Recall
--------------------------------
The Search query "buying a new home". Imagine in our index we have the below 3 relavant documents and 3 Irrelavant documents.

Relavant books
--------------
1 The Beginner’s Guide to Buying a House
2 How to Buy Your First House
3 Purchasing a Home

Irrelevant books
----------------
4 A Fun Guide to Cooking
5 How to Raise a Child
6 Buying a New Car

Precision
---------
If all 6 records appear in the result, precision is 0.5 because 3 of the 6 records are irrelavant.
If all 6 records appear in the result, recall is 1 because all 3 matched records are found. That means none of the matched records are missed.

If 3 results are returned but only 1 matched record found in the result, recall is 1/3.

Striking the right balance
--------------------------
Maximizing for full Precision and full Recall is the ultimate goal of most every search-relevancy-tuning endeavor.Many techniques can be undertaken within Solr to improve either Precision or Recall, though most are geared more toward increasing Recall in terms of the full document set being returned. Aggressive textual analysis(to find multiple variations of words) is a great example of trying to find more matches, though these additional matches may hurt overall Precision if the textual analysis is so aggressive that it matches incorrect word variations.

The best solution to the problem is measuring for Recall across the entire result set and measuring for Precision only within the first page (or few pages) of search results.
Because many search websites, for example, want to appear to have as much content as possible, and because those sites know that visitors will never go beyond the first few pages, they can show precise results on the first few pages while still including many less precise matches on subsequent pages.

Note:Most search applications fall somewhere between these two extremes, and striking the right balance between Precision and Recall is a never-ending challenge:

CONFIGURING SOLR
----------------
Most of the configuration you’ll do with Solr focuses around three main XML files:
■ solr.xml—Defines properties related to administration, logging, sharding, and SolrCloud --> This is not present in the newer versions of solr.
■ solrconfig.xml—Defines the main settings for a specific Solr core
■ schema.xml—Defines the structure of your index, including fields and field types

Note:
solrconfig.xml is used during the Solr initialization process to create and set up the collection.

Overview of solrconfig.xml
--------------------------
Lets look at the condensed version of solrconfig.xml file.

<config>
	<luceneMatchVersion>4.7</luceneMatchVersion>					-->Activates version-dependent features in Lucene
	<lib dir="../../../contrib/extraction/lib" regex=".*\.jar" /> 	--> Lib directives indicate where Solr can find JAR files for extensions
	<dataDir>${solr.data.dir:}</dataDir>							--> Index management settings
	<directoryFactory name="DirectoryFactory” class="..."/>
	<indexConfig> ... </indexConfig>
	<jmx />															--> Enables JMX instrumentation of Solr MBeans
	<updateHandler class="solr.DirectUpdateHandler2"> 				--> Update handler for indexing documents
		<updateLog> ... </updateLog>
		<autoCommit> ... </autoCommit>
	</updateHandler>
	<query>															--> Cache management settings
		<filterCache ... />											--> Register event handlers for searcher events; for example, queries to execute to warm new searchers
		<queryResultCache ... />
		<documentCache ... />
		<listener event="newSearcher" class="solr.QuerySenderListener">
			<arr name="queries"> ... </arr>
		</listener>
		<listener event="firstSearcher" class="solr.QuerySenderListener">
			<arr name="queries"> ... </arr>
		</listener>
	</query>
	<requestDispatcher handleSelect="false" >						--> Unified request dispatcher
		<requestParsers ... />
		<httpCaching never304="true" />
	</requestDispatcher>
	<requestHandler name="/select" class="solr.SearchHandler">		--> Request handler to process queries using a chain of search components
		<lst name="defaults"> ... </lst>
		<lst name="appends"> ... </lst>
		<lst name="invariants"> ... </lst>
		<arr name="components"> ... </arr>
		<arr name="last-components"> ... </arr>
	</requestHandler>
	<searchComponent name="spellcheck"
		class="solr.SpellCheckComponent"> ... 						--> Example search component for doing spell correction on queries.
	</searchComponent>
	<updateRequestProcessorChain name="langid"> ...					--> Extends indexing behavior using update-request processors, such as language detection.
	</updateRequestProcessorChain>
	<queryResponseWriter name="json"								--> Formats the response as JSON.
		class="solr.JSONResponseWriter"> ... </queryResponseWriter>
	<valueSourceParser name="myfunc" ... />							--> Declares a custom function for boosting, ranking, or sorting result documents.
	<transformer name="db" 
		class="com.mycompany.LoadFromDatabaseTransformer">			--> Transforms result documents.
	...
	</transformer>
</config>

Miscellaneous settings
----------------------
a)LUCENE VERSION
----------------
The <luceneMatchVersion> element controls the version of Lucene your index is based on. After running Solr for several months and indexing millions of documents, you decide that you need to upgrade to a later version of Solr the updated Solr server, it uses the <luceneMatchVersion> to understand which version your index is based on and whether to disable any Lucene features that depend on a later version than what is specified.

You’ll be able to run the upgraded version of Solr against your older index, but at some point you may need to raise the <luceneMatchVersion> to take advantage of new features and bug fixes in Lucene.

b)LOADING DEPENDENCY JAR FILES
------------------------------
The <lib> element allows you to add JAR files to Solr’s runtime classpath so that it can locate plugin classes.

<lib dir="../../../contrib/langid/lib/" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-langid-\d.*\.jar" />

Note:
Each <lib> element identifies a directory and a regular expression to match files in the directory. Notice that the dir attribute uses relative paths, which are evaluated from the
core directory root.

ENABLE JMX
----------
The <jmx> element activates Solr’s MBeans to allow system administrators to monitor and manage core Solr components from popular system-monitoring tools, such as Nagios.

Query request handling
----------------------

http://localhost:8983/solr/collection1/select?q=iPod&fq=manu:Belkin&sort=price asc&fl=name,price,features,score&df=text&wt=xml&start=0&rows=10

The sequence of events and the main components involved in handling this Solr request are as follows.

a)Jetty accepts the request and routes it to Solr’s unified request dispatcher using the /solr context in the request path. In technical terms, the unified request dispatcher is a Java servlet filter mapped to /* for the Solr web application; see org.apache.solr.servlet.SolrDispatchFilter.

b)Solr’s request dispatcher uses the collection1 part of the request path to determine the core name. Next, the dispatcher locates the /select request handler registered in solrconfig.xml for the collection1 core.

c)The /select request handler processes the request using a pipeline of search components.

d)After the request is processed, results are formatted by a response writer component and returned to the client application; by default, the /select handler returns results as XML.


Note:In practice, the default configuration for the request dispatcher is sufficient for most applications. On the other hand, it’s common to define a custom search request handler or to customize one of the existing handlers, such as /select.

NOte:
In Solr, there are two main types of request handlers: search handler for query processing and update handler for indexing.

Search handler
--------------
Behind the scenes, all request handlers are implemented by a Java class:in this case, solr.SearchHandler.

Note: In general, anytime you see solr. as a prefix of a class in solrconfig.xml, you know this translates to one of Solr’s core Java packages: "analysis.", "schema.", "handler.", "search.", "update.", "core.","request.", "update.processor.", "util.", "spelling.", "handler.component.", or "handler.dataimport.

Note:
In Solr, there are two main types of request handlers: search handler for query processing and update handler for indexing.

Custom Request(or search) handler
---------------------------------
you can define your own request handler or, more commonly, add a custom search component to an existing request handler, such as /select. In general, a search handler is
comprised of the following phases, and each phase can be customized in solrconfig.xml:

A search handler is comprised of the following phases, and each phase can be customized in solrconfig.xml

1)Request Parameter decoration using
	a) defaults --> Set default parameters on the request if they are not explicitly provided by the client
	b) invariants --> Set parameters to fixed values, which override values provided by the client
	c) appends --> Additional parameters to be combined with the parameters provided by the client

2)First Components
An optional chain of search components that are executed first to perform preprocessing tasks

3)Components
The primary chain of search components that must at least include the query component(Built-in set of standard search components such as Facet, More Like This, Highlight, Stats, Debug etc chained together.)

4)Last components
An optional chain of search components that are applied last to perform postprocessing tasks(Include zero or more postprocessing search components such as a spellchecker.)

The following flow represents the search request handling
------------------------------------------------------------------
Request Parameters(defaults --> invariants --> appends) --> First Components --> Query (Searcher --> Lucene Index) --> Components(Facet --> More Like This --> Highlight --> Stats --> Debug) --> Last Components (spellchecker) --> response writer 

Configuring custom search handler
---------------------------------
Hiding complexity from client code is at the heart of web services and object-oriented design. We can write our custom handler by setting default and last components as per the need of our application. Here is an example with /browse(instead of default /select provided by solr).

<requestHandler name="/browse" class="solr.SearchHandler">
	<lst name="defaults">								--> default list of query parameters that is passed to solr with every request when the user explicitly don't send.
		<str name="echoParams">explicit</str>
		<str name="wt">velocity</str>
		<str name="v.template">browse</str>
		<str name="v.layout">layout</str>
		<str name="title">Solritas</str>
		<str name="defType">edismax</str>
		<str name="qf">text^0.5 features^1.0 ...</str>
		<str name="mlt.qf">text^0.5 features^1.0 ...</str>
		<str name="facet">on</str>
		...
		<str name="hl">on</str>
		...
		<str name="spellcheck">on</str>
		...
	</lst>
	<arr name="last-components">					--> last components that are applied with every request
		<str>spellcheck</str>
	</arr>
</requestHandler>

If we send this request to http://localhost:8983/solr/collection1/browse?q=iPod&wt=xml&echoParams=all solr, then we get the following response.

<lst name="params">
	<str name="facet">on</str>
	<str name="mlt.fl">text,features,name,sku,id,manu,cat,title,description,keywords,author,resourcename</str>
	<str name="f.manufacturedate_dt.facet.range.gap">+1YEAR</str>
	<str name="f.price.facet.range.gap">50</str>
	<str name="q.alt">*:*</str>
	<str name="f.content.hl.fragsize">200</str>
	<str name="v.layout">layout</str>
	<str name="echoParams">all</str>
	<str name="fl">*,score</str>
	<str name="f.price.facet.range.end">600</str>
	<str name="hl.simple.post">&lt;/b&gt;</str>
	<str name="f.name.hl.fragsize">0</str>
	<arr name="facet.field">
		<str>cat</str>
		<str>manu_exact</str>
		<str>content_type</str>
		<str>author_s</str>
	</arr>
	<str name="hl.encoder">html</str>
	<str name="v.template">browse</str>
	<str name="spellcheck.alternativeTermCount">2</str>
	<str name="f.popularity.facet.range.end">10</str>
	<str name="f.manufacturedate_dt.facet.range.start">
	NOW/YEAR-10YEARS</str>
	<str name="spellcheck.extendedResults">false</str>
	<str name="spellcheck.maxCollations">3</str>
	<str name="hl.fl">content features title name</str>
	<str name="f.content.hl.maxAlternateFieldLength">750</str>
	<str name="spellcheck.collate">true</str>
	<str name="wt">xml</str>
	<str name="defType">edismax</str>
	<str name="rows">10</str>
	<str name="facet.range.other">after</str>
	<str name="f.popularity.facet.range.start">0</str>
	<str name="f.title.hl.alternateField">title</str>
	<str name="facet.pivot">cat,inStock</str>
	<str name="f.title.hl.fragsize">0</str>
	<str name="spellcheck">on</str>
	<str name="spellcheck.maxCollationTries">5</str>
	<arr name="facet.range">
		<str>price</str>
		<str>popularity</str>
		<str>manufacturedate_dt</str>
	</arr>
	<str name="hl.simple.pre">&lt;b&gt;</str>
	<str name="hl">on</str>
	<str name="title">Solritas</str>
	<str name="df">text</str>
	<arr name="facet.query">
		<str>ipod</str>
		<str>GB</str>
	</arr>
	...
</lst>
...

Note:It should be clear that parameter decoration for a search request handler is a powerful feature in Solr. Defaults help simplify client code by establishing sensible defaults for your application in one place.

QUERY COMPONENT
---------------
The query component is the core of Solr’s query-processing pipeline. At a high level, the query component parses and executes queries.

FACET COMPONENT
---------------
Given a result set identified by the query component, the facet component, if enabled, calculates field-level facets. It provides unique values with count for a given field.

MORE LIKE THIS COMPONENT
------------------------
Given a result set created by the query component, the More Like This component, if enabled, identifies other documents that are similar to the documents in search results.
For ex, search for a query "hard drive" and assume we have a result “Samsung SpinPoint P120 SP2514N - hard drive - 250 GB - ATA-133”. By enabling More Like This query, it returns results of similar hard drives.

HIGHLIGHT COMPONENT
-------------------
If enabled, the highlight component highlights highly relevant sections of text in matching documents.

STATS COMPONENT
---------------
The stats component computes simple statistics like min, max, sum, mean, and standard deviation for numeric fields in matching documents. To see an example of this try the below query.

/select?q=*:*&wt=json&stats=true&stats.field=price

Response for the above query will give the stats like below.

"stats": {
	"stats_fields": {
		"price": {
			"min": 0,
			"max": 2199,
			"count": 16,
			"missing": 16,
			"sum": 5251.270030975342,
			"sumOfSquares": 6038619.175900028,
			"mean": 328.20437693595886,
			"stddev": 536.3536996709846,
			"facets": {}
		}
	}
}

DEBUG COMPONENT
---------------
The debug component returns the parsed query string that was executed and detailed information about how the score was calculated for each document in the result set. Try the below query 

/select?q=ipod&wt=json&debug=true

Here is the debug response

"debug": {
	"rawquerystring": "ipod",
	"querystring": "ipod",
	"parsedquery": "text:ipod",
	"parsedquery_toString": "text:ipod",
	"explain": {
	"IW-02": "\n1.3334373 = (MATCH) weight(text:ipod in 4) [DefaultSimilarity], result of:\n  1.3334373 = fieldWeight in 4, product of:\n    1.7320508 = tf(freq=3.0), with freq of:\n      3.0 = termFreq=3.0\n    3.0794415 = idf(docFreq=3, maxDocs=32)\n    0.25 = fieldNorm(doc=4)\n",
	"F8V7067-APL-KIT": "\n0.7698604 = (MATCH) weight(text:ipod in 3) [DefaultSimilarity], result of:\n  0.7698604 = fieldWeight in 3, product of:\n    1.0 = tf(freq=1.0), with freq of:\n      1.0 = termFreq=1.0\n    3.0794415 = idf(docFreq=3, maxDocs=32)\n    0.25 = fieldNorm(doc=3)\n",
	"MA147LL/A": "\n0.28869766 = (MATCH) weight(text:ipod in 5) [DefaultSimilarity], result of:\n  0.28869766 = fieldWeight in 5, product of:\n    1.0 = tf(freq=1.0), with freq of:\n      1.0 = termFreq=1.0\n    3.0794415 = idf(docFreq=3, maxDocs=32)\n    0.09375 = fieldNorm(doc=5)\n"
	},
	"QParser": "LuceneQParser",
...

ADDING SPELLCHECK AS A LAST-COMPONENT
-------------------------------------
There is a spellcheck component in solrconfig.xml like this.

<searchComponent name="spellcheck" class="solr.SpellCheckComponent">
	<str name="queryAnalyzerFieldType">textSpell</str>
	<lst name="spellchecker">
	<str name="name">default</str>
	<str name="field">name</str>
	<str name="classname">solr.DirectSolrSpellChecker</str>
	...
	</lst>
</searchComponent>

Also, there is a /spell request handler with below configuration.

<requestHandler name="/spell" class="solr.SearchHandler" startup="lazy">
    <lst name="defaults">
      <str name="df">text</str>
      <!-- Solr will use suggestions from both the 'default' spellchecker
           and from the 'wordbreak' spellchecker and combine them.
           collations (re-written queries) can include a combination of
           corrections from both spellcheckers -->
      <str name="spellcheck.dictionary">default</str>
      <str name="spellcheck.dictionary">wordbreak</str>
      <str name="spellcheck">on</str>
      <str name="spellcheck.extendedResults">true</str>       
      <str name="spellcheck.count">10</str>
      <str name="spellcheck.alternativeTermCount">5</str>
      <str name="spellcheck.maxResultsForSuggest">5</str>       
      <str name="spellcheck.collate">true</str>
      <str name="spellcheck.collateExtendedResults">true</str>  
      <str name="spellcheck.maxCollationTries">10</str>
      <str name="spellcheck.maxCollations">5</str>         
    </lst>
    <arr name="last-components">
      <str>spellcheck</str>
    </arr>
</requestHandler>

Note: we can try /spell to see for any spelling corrections. For ex, Try wrong query battry instead of battery like,

/spell?wt=json&q=battry

Response:

"spellcheck": {
	"suggestions": [
	"battry",
	{
		"numFound": 1,
		"startOffset": 0,
		"endOffset": 6,
		"origFreq": 0,
		"suggestion": [
		{
		"word": "battery",
		"freq": 2
		}
		]
	},
	"correctlySpelled",
	false,
	"collation",
		[
			"collationQuery",
			"battery",
			"hits",
			2,
			"misspellingsAndCorrections",
			[
				"battry",
				"battery"
			]
		]
	]
}

Managing searchers
------------------
The <query> element contains settings that allow you to optimize query performance using techniques like caching, lazy field loading, and new searcher warming. Designing for optimal query performance from the start is critical to the success of your search application.

	<query>															--> Cache management settings
		<filterCache ... />											--> Register event handlers for searcher events; for example, queries to execute to warm new searchers
		<queryResultCache ... />
		<documentCache ... />
		<listener event="newSearcher" class="solr.QuerySenderListener">
			<arr name="queries"> ... </arr>
		</listener>
		<listener event="firstSearcher" class="solr.QuerySenderListener">
			<arr name="queries"> ... </arr>
		</listener>
	</query>

New searcher overview
---------------------
In Solr, queries are processed by a component called a searcher. There is only one “active” searcher in Solr at any given time. All query components for all search request
handlers execute queries against the active searcher. The active searcher has a read-only view of a snapshot of the underlying Lucene index.

If you add a new document to Solr, then it is not visible in search results from the current searcher. To make the new document visible, the current searcher has to be closed and a new searcher has to be opened. This is the meaning of committing the documents to solr.

In Solr admin, Under Core section of the plugin/stats page we can see the current (or active) searcher in QueryHandler --> /select section. If we observe searcherName property, then it has the object address(like Searcher@25082661). Now post same xml documents in the exampledocs folder using post.jar. This will close the current searcher and opens a new searcher. This change can be seen in the same QueryHandler --> /selection with a new object address for Searcher.

Implications of creating new searcher
-------------------------------------
First, the old searcher must be destroyed. But there could be queries currently executing against the old searcher, so Solr must wait for all in-progress queries to complete. Any cached objects that are based on the old searcher’s view of the index must be invalidated because some of the documents in the cached results may have been deleted and new documents may match the query. Because of these opening a new searcher is an expensive operation.

Warming a new searcher
----------------------
Imagine a user paging through search results. After clicking on a page results and before going to next page if a new searcher is opened, all of the previously computed caches and filters are no longer valid in the context of a new searcher. User could experience slowness in the search results. To alleviate from this problem, Solr supports the concept of warming a new searcher in the background and keeping the current searcher active until the new one is fully warmed.

Note: Solr compromises to give stale data for short period(until new searcher is warmed) rather than compromising on query performance.

There are two steps in warming a new searcher.

a)Executing Cache warming queries
b)Auto warming new caches from old cache queries (This is covered in next section with Cache management)

Executing Cache warming queries
-------------------------------
A cache-warming query is a preconfigured query (in solrconfig.xml) that gets executed against a new searcher in order to populate the new searcher’s caches. For ex,

<listener event="newSearcher" class="solr.QuerySenderListener">
	<arr name="queries">
		<lst>
			<str name="q">solr</str>
			<str name="sort">price asc</str>
		</lst>
		<lst>
			<str name="q">rocks</str>
			<str name="sort">weight asc</str>
		</lst>
	</arr>
</listener>

In the above example there are two queries that get executed when warming a new searcher.

CHOOSING WARMING QUERIES
------------------------
Having the facility to warm new searchers by executing queries is only a great feature if you can identify queries that will help improve query performance. As a rule of thumb, warming queries should contain query-request parameters (q, fq, sort, etc.) that are used frequently by your application.

Note: In general there is no need to use warming queries, but if query performance begins to suffer after commits, then you’ll know it’s time to consider using warming queries.

TOO MANY WARMING QUERIES
------------------------
Having many warming queries configured can lead to long delays in opening new searchers. It turns out that warming too many searchers in your application concurrently
can consume too many resources (CPU and memory), leading to a degraded search experience.

FIRST SEARCHER
--------------
There is also the concept of warming the first searcher during Solr initialization or after reloading a core. We can configure this in solrconfig.xml. Default configuration is,

    <listener event="firstSearcher" class="solr.QuerySenderListener">
      <arr name="queries">
        <lst>
          <str name="q">static firstSearcher warming in solrconfig.xml</str>
        </lst>
      </arr>
    </listener>

USECOLDSEARCHER
---------------
The <useColdSearcher> element covers the case in which a new search request is made and there is no currently registered searcher. If <useColdSearcher> is false, then Solr will block until the warming searcher has completed executing all warming queries; this is the default configuration for the example Solr server: 
	
	<useColdSearcher>false</useColdSearcher>.

Note: If <useColdSearcher> is true, then Solr will immediately register the warming searcher regardless of how “warm” it is.

MAXWARMINGSEARCHERS
-------------------
It's possible that new documents are indexed and commit issued before the new searcher warming is completed. This will invalidate the new searcher and opens another new searcher. This is true if new searchers warm up takes considerable amount of time. The <maxWarmingSearchers> element allows you to control the maximum number of searchers that can be warming up in the background concurrently. Once this threshold is reached, new commit requests will fail, which is a good thing, because allowing too many warming searchers to run in the background can quickly eat up memory and CPU resources on your server. Solr ships with a default of 2, which is a good value to start with:
	
	<maxWarmingSearchers>2</maxWarmingSearchers>

Cache Management
----------------
Solr provides a number of built-in caches to improve query performance. It’s important to understand cache-management fundamentals in Solr.

Cache fundamentals
------------------
There are four main concerns when working with Solr caches:
■ Cache sizing and eviction policy
■ Hit ratio and evictions
■ Cached-object invalidation
■ Autowarming new caches

CACHE SIZING
------------
Solr keeps all cached objects in memory and does not overflow to disk. Solr requires you to set an upper limit on the number of objects in each cache. Solr will evict objects when the cache reaches the upper limit, using either a Least Recently Used or Least Frequently Used eviction policy.

Least Recently Used (LRU) evicts objects when a cache reaches its maximum threshold based on the time when an object was last requested from the cache. LRU policy will remove the oldest entry; age is determined by the last time each object in the cache was requested.

Solr also provides a Least Frequently Used (LFU) policy that evicts objects based on how frequently they are requested from the cache. This is beneficial for applications
that want to give priority to more popular items in the cache, rather than only those that have been used recently.

Note: Solr’s filter cache is a good candidate for using the LFU eviction policy, because filters are typically expensive to create and store, so you want to keep the filter cache small and give priority to the most popular filters in your application.

Note:A common misconception with cache sizing is that you should make your cache sizes quite large if you have the memory available. The problem with this approach is that once a cache becomes invalidated after a commit, there can be many objects that need to be garbage collected by the JVM. Remember, closing a searcher invalidates all cached values.

HIT RATIO AND EVICTIONS
-----------------------
Hit ratio is the proportion of cache read requests that result in finding a cached value. The hit ratio indicates how much benefit your application is getting from its cache.
Ideally, you want your hit ratio to be as close to 1 (100%) as possible. A low hit ratio is an indication that Solr is not benefiting from caching.

Having a large number of evictions is an indication that the maximum size of your cache may be too small for your application. Eviction count and hit ratio are interrelated, as a high eviction count will lead to a sub-optimal hit ratio.

AUTOWARMING NEW CACHES
----------------------
Some of the keys in the soon-to-be-closed searcher’s cache can be used to populate the new searcher’s cache, a process known as autowarming in Solr.

Every Solr cache supports an autowarmCount attribute that indicates either the maximum number of objects or a percentage of the old cache size to autowarm.

Types of Caches
---------------
1)Filter cache
--------------
In Solr, a filter restricts search results to documents that meet the filter criteria, but it does not affect scoring. In the example we have used the filter query fq=manu:Belkin with query string "ipod". Now consider what happens if another query is sent to Solr with the same filter query (fq=manu:Belkin) but a different query, such as q=USB.
It would be nice if the second query uses the results of the filter query which is executed during first query. This is the purpose of Solr’s filter cache!

	<filterCache class="solr.FastLRUCache" size="512" initialSize="512" autowarmCount="0"/>

AUTOWARMING THE FILTER CACHE
----------------------------
If a filter is generic enough to apply to multiple queries in your application, it makes sense to use cache. We need to autowarm some of the cached filters when opening a new searcher. 

we know that objects cannot easily be moved from the old to the new cache because the underlying index has changed, which invalidates cached objects like filters. Each object in the cache has a key. For the filter cache, the key is the filter query, such as manu:Belkin. To warm the new cache, a subset of keys is pulled from the old cache and executed against the new searcher, which recomputes the filter. So, Autowarming the filter cache requires Solr to re-execute the filter query with the new searcher.

Imagine the scenario in which you have hundreds of filters cached and your autowarmCount is set to 100. When warming the new searcher, Solr must execute 100 filter queries.

Note: It is recommonded to use filter cache, but set the autowarmCount attribute to a small number to start. LFU eviction policy is more appropriate for the filter cache because it allows you to keep the filter cache small and give priority to the most popular filters in your application. For ex,

	<filterCache class="solr.LFUCache" size="100" initialSize="20" autowarmCount="10"/>

Note: We need to experiment with these settings as per your application need.

2)Query result cache
--------------------
If you execute the below query for more than once, subsequent results are returned from query result cache.

http://localhost:8983/solr/collection1/select?q=iPod&fq=manu:Belkin&sort=price asc&fl=name,price,features,score&df=text&wt=xml&start=0&rows=10

The query result cache is defined as

	<queryResultCache class="solr.LRUCache" size="512" initialSize="512" autowarmCount="0"/>

Note: The query result cache holds a query as the key and a list of internal Lucene document IDs as the value. Internal Lucene document IDs can change from one searcher to the next, so the cached values must be recomputed when warming the query result cache.

a)QUERY RESULT WINDOW SIZE
--------------------------
Imagine your application shows 10 documents per page and in most cases your users only look at the first and second pages. You can set <queryResultWindowSize> to 20 to avoid having to re-execute the query to retrieve the second page of results.  The configuration for this is,
	
	<queryResultWindowSize>20</queryResultWindowSize>

b)QUERY RESULT MAX DOCS CACHED
------------------------------
Even though we have limit of how many query results can be cached using query result cache, we don't have limit of howmany documents to cache per a query result. This setting helps to limit the max docs that can be cached per a query result.

	<queryResultMaxDocsCached>200</queryResultMaxDocsCached>

Note: In the above setting queryResultWindowSize(20), if a user keeps paginating through the results documents are keep adding to cache until the setting queryResultMaxDocsCached(200).

c)ENABLE LAZY FIELD LOADING
---------------------------
A common design pattern in Solr is to have a query return a subset of fields for each document. In the example, we requested the name,price, features, and score fields. But, documents in the index have many more fields. It is recommonded to use the lazy field loading setting because it improves the search performance.

	<enableLazyFieldLoading>true</enableLazyFieldLoading>

3)Document cache
----------------
Document cache is used to cache documents. Query result cache uses the document cache to find cached versions of documents in the cached result set.

Note:If most of your index is relatively static, the document cache may provide value, because document caches are invalidated with new searcher.

4)Field value cache
-------------------
Field value cache is used by Lucene and is not managed by Solr. The field value cache provides fast access to stored field values by internal document ID. It is used during sorting and when building documents for the response. The setting for this cache is,

	<fieldValueCache class="solr.FastLRUCache" size="512" autowarmCount="128" showItems="32" />

Note: The fieldValueCache is created by default even if we don't configure.

																		Indexing
																		--------
As we know Solr finds documents using an inverted index, which in its simplest form is a dictionary of terms and a list of documents in which each term occurs. Solr uses this index to match terms in a user’s query with the documents in which they occur. A key factor in indexing documents is text analysis.

Example microblog search application
------------------------------------
Microblog is a generic term for the short, informal messages and other media that people share with each other on social networks. Examples of microblogs are tweets on Twitter, Facebook posts, and check-ins on Foursquare.

Representing content for searching
----------------------------------
As we know each document in a Solr index is made up of fields, and each field has a specific type that determines how it’s stored, searched, and analyzed. An example document,
{
        "id": "1",
        "screen_name": "@thelabdude",
        "type": "post",
        "lang": "en",
        "timestamp": "2012-05-22T09:00:00Z",
        "favorites_count": 10,
        "text": "#Yummm :) Drinking a latte at Caffe Grecco in SF's historic North Beach... Learning text analysis with #SolrInAction by @ManningBooks on my i-Pad",
}

screen_name, type, timestamp, lang, and text fields are good candidates to use from a search perspective because they contain information that a typical user could use to build a query. For ex, users might want all english tweets from a specific user for the given date range.

Note: Eventhough we can index all fields, but when we develop a large scale system to support millions of documents and high query volumes its better to index fields that will be searched by users.

Solr indexing process
---------------------
Solr indexing process has 3 steps at highlevel.
a)Convert a document from its native format into a format supported by Solr, such as XML or JSON.
b)Add the document to Solr using one of several well-defined interfaces, typically HTTP POST.
c)Configure Solr(Designing Schema) to apply transformations to the text in the document during indexing.

Designing Schema
----------------
Designing schema needs to answer the below key questions about your search application:

■ What is a document in your index?
■ How is each document uniquely identified?
■ What fields in your documents can be searched by users?
■ Which fields should be displayed to users in the search results?

Document Granularity
--------------------
In the above tweet example, the text content is typically short, so each tweet will be a document. But if the content you want to index is large, such as a technical computer book, you may want to treat subsections(or chapters) of a large document as the indexed unit.

For ex, Imagine searching for "text analysis" on a website that sells technical computer books. If the site treated each book as a single document, the user would see Solr in
Action in the search results, but would need to page through the table of contents or index to find specific places where "text analysis" occurs in the book.

If the site treated individual chapters in each book as documents in the index, then the search results might show the user the “Text analysis” chapter in Solr in Action as the top result.

Note: 
We may need to consider the type of content. Splitting a technical book by chapters make sense, but splitting a fiction novel by chapter doesn't make any sense.

In the end, it’s your choice what makes a document in your index, but definitely consider how document granularity impacts user experience. In general, you want your documents to be as granular as possible without causing your users to miss the forest for the trees.

Unique Key
----------
Solr doesn’t require a unique identifier for each document, but if one is supplied, Solr uses it to avoid duplicating documents in your index. This id can be used to update a document.

Indexed fields
--------------
Once determined on what a document represents(Document granularity) and how to uniquely identify a document, next step is to identify indexed fields in each document. The best way to think about indexed fields is to ask whether a typical user could develop a meaningful query using that field.

For example, every book has a title and an author. When searching for books, people generally expect to find books of interest based on title and author, so these fields should both be indexed. Although every book has an editor, users typically don’t search by editor name.

Note:If you were building a search index for the book publishing industry, it’s likely that your users would want to search by editor name, so you’d include that as an indexed field.

In out tweet example, the screen_name, type, timestamp, lang, and text fields should be indexed for our microblog example. The id and user_id fields are used internally by Twitter and are unlikely to be missed if you don’t allow users to search by these fields.

Stored fields
-------------
Although users probably won’t search by editor name to find a book to read, we may still want to display the editor’s name in the search results. In general, your documents may contain fields that aren’t useful from a search perspective but are still useful for displaying search results. In Solr, these are called stored fields. favourites_count is another good example of stored result.

Of course, a field may be indexed and stored, such as the screen_name, timestamp, and text fields in our microblog search application. Each of these fields can be searched and displayed in results.

Note:
As a search-application architect, one of your goals should be to minimize the size of your index. Each indexed field consumes the index size. Each stored field consumes disk space and requires CPU and I/O resources to read the stored value to return it in search results.

Preview of schema.xml
---------------------
The schema.xml file is in the conf/ directory for your Solr core.

			<schema name="example" version="1.5">
				<fields>
					<field name="id" type="string" indexed="true" stored="true" .../>
					<field name="name" type="text_general" indexed="true" stored="true"/>
					<field name="cat" type="string" indexed="true" stored="true" .../>
					...
					<dynamicField name="*_s" type="string" indexed="true" stored="true"/>
					<dynamicField name="*_t" type="text_general" indexed="true" .../>
					...
				</fields>
				<uniqueKey>id</uniqueKey>
				<copyField source="cat" dest="text"/>
				...
				<copyField source="manu" dest="text"/>
				<types>
					<fieldType name="string" class="solr.StrField" .../>
					<fieldType name="boolean" class="solr.BoolField" .../>
					...
					<fieldType name="tint" class="solr.TrieIntField" .../>
					<fieldType name="tfloat" class="solr.TrieFloatField" .../>
					<fieldType name="text_general" class="solr.TextField">
						<analyzer type="index">
							<tokenizer class="solr.StandardTokenizerFactory"/>
							<filter class="solr.StopFilterFactory" .../>
							<filter class="solr.LowerCaseFilterFactory"/>
						</analyzer>
						<analyzer type="query">
							<tokenizer class="solr.StandardTokenizerFactory"/>
							<filter class="solr.StopFilterFactory" .../>
							<filter class="solr.SynonymFilterFactory" .../>
							<filter class="solr.LowerCaseFilterFactory"/>
						</analyzer>
					</fieldType>
					...
				</types>
			</schema>

There are three main sections in the schema.
1)<fields> --> contains <field> and <dynamicField> elements
2)<uniqueKey> --> defined the unique id for each document in solr
	<copyField> --> used to copy data from one field to another field
3)<types> --> which defines how text, date, numbers and other types are handled

Fields
------
The <fields> section in schema.xml defines <field> elements for all fields in your document. Solr uses the field definitions from schema.xml to figure out what kind of analysis needs to be performed for fields in your documents in order to add their content as terms to the inverted search index.

		<fields>
			<field name="id" type="string" indexed="true" stored="true" required="true"/>
			<field name="screen_name" type="string" indexed="true" stored="true"/>
			<field name="type" type="string" indexed="true" stored="true"/>
			<field name="timestamp" type="tdate" indexed="true" stored="true"/>
			<field name="lang" type="string" indexed="true" stored="true"/>
			<field name="favorites_count" type="int" indexed="true" stored="true"/>
			<field name="text" type="text_microblog" indexed="true" stored="true"/>
			<field name="link" type="string" indexed="true" stored="true" multiValued="true"/>
			...
		</fields>

Each field must define a type, that identifies the <fieldType> to use for that field.
If a field has indexed=true, then the field value is analyzed, so that you can search on it.
If a field has stored=true, then the field original value is also stored, so that it's original content can be returned in search results.
If a filed has required=true, then as part of the document this value must be present in the index, otherwise solr will reject the document.

Multivalued Field
-----------------
Mutlivalued fields support 0 or more values for the same field in a single document for indexing. For ex,

		<add>
			<doc>
				<field name="id">2</field>
				...
				<field name="link">http://manning.com/grainger/</field>
				<field name="link">http://lucene.apache.org/solr/</field>
				...
			</doc>
		</add>

Dynamic fields
--------------
In Solr, dynamic fields allow you to apply the same definition to any fields in your documents whose names match either a prefix or suffix pattern, such as s_* or *_s. Dynamic fields use a special naming scheme to apply the same field definition to any fields that match this kind of glob-style pattern. Dynamic fields help address common problems that occur when building search applications, including
■ Modeling documents with many fields
■ Supporting documents from diverse sources

				<fields>
					<dynamicField name="*_s" type="string" indexed="true" stored="true"/>
					<dynamicField name="*_t" type="text_general" indexed="true" .../>
				</fields>

Solr ignores the dynamic field definitions in your schema.xml until you start indexing documents that make use of them.

MODELING DOCUMENTS WITH MANY FIELDS
-----------------------------------
Imagine, if we have dozens of string fields that are also stored and indexed. you can define a single <dynamicField> element to account for all of these string fields using a suffix pattern on the field name. For ex, 
	
	<dynamicField name="*_s" type="string" indexed="true" stored="true"/>

With this glob pattern, any field with a name ending in _s will inherit this field definition, such as subject_s. For ex, link in the below sample will get string type

		<add>
			<doc>
				<field name="id">2</field>
				<field name="link_s">http://manning.com/grainger/</field>
			</doc>
		</add>

SUPPORTING DOCUMENTS FROM DIVERSE SOURCES
-----------------------------------------
In our social media example, if we index documents from Twitter, Facebook, YouTube, and Google+, then documents from each of these sources will have some fields that are unique to each social network. It's easy to handle them with dynamic fields instead of defining type for each field. For ex, to index the following document we don't need any change in schema.

		<add>
			<doc>
				<field name="id">9999012345678</field>
				<field name="screen_name">@thelabdude</field>
				<field name="type">post</field>
				...
				<field name="facebook_f1_s">hello</field>
				<field name="facebook_f2_s">world</field>
				<field name="twitter_f1_s">foo</field>
				<field name="twitter_f2_s">bar</field>
			</doc>
		</add>

Copy fields
-----------
copy fields allow you to populate one field from one or more other fields. Specifically, copy fields support two use cases that are common in most search applications:

■ Populate a single catch-all field with the contents of multiple fields.
■ Apply different text analysis to the same field content to create a new searchable field.

CREATE A CATCH-ALL FIELD FROM MANY FIELDS
-----------------------------------------
In most search applications, users are presented with a single search box in which to enter a query. The intent of this approach is to help your users quickly find documents
without having to fill out a complicated form; think about how successful a simple search box has been for Google.

In Solr, without a catchall field, to search for the given text the query format is {field_name}:{search_text}. For ex, in our tweet example if you want to search for a text say "north beach", the query should be text:north beach. It is tough for the users to remember all the fields present in your schema and to search. Solr makes it easy to create a single catch-all search field from many other fields in your document using the <copyField> directive. For ex,

	<schema>
		<fields>
				<field name="catch_all" type="text_en" indexed="true" stored="false" multiValued="true"/>
		</fields>
		<copyField source="screen_name" dest="catch_all" />
		<copyField source="text" dest="catch_all" />
		<copyField source="link" dest="catch_all" />
		<types>
			...
		</types>
	</schema>

Note: This field isn’t stored (stored="false"), which makes sense because we probably don’t want to display a blob of many fields concatenated together to our users.

Note:If any of the source fields are multivalued, the destination field must be a multivalued field (multiValued="true"). In our case, the link field is multivalued, so 
we must define our copy field as multivalued as well.

APPLY DIFFERENT ANALYZERS TO A FIELD
------------------------------------
We may want to analyze the contents of a single field differently. Let's see this with stemming example. Stemming is a technique that transforms terms into a common base form,
known as a stem, in order to improve Recall. With stemming, the terms fishing, fished, and fishes can all be reduced to a common stem of fish. Thus, stemming can help your users find documents without having to think about all the possible linguistic forms of a word, which is a good approach for general text search fields.

Consider how stemming would affect a type-ahead suggestion box (autosuggest). In this case, stemming would work against your users in that you could only suggest the stemmed values and not the full terms. With stemming enabled, for example, your search application wouldn’t be able to suggest fishing, fished etc when the user started typing fish.

Solr copy fields give you the flexibility to enable or disable certain text-analysis features like stemming without having to duplicate storage but storing the terms differently in your index. For ex,

	<field name="text" type="stemmed_text" indexed="true" stored="true"/>
	<field name="auto_suggest" type="unstemmed_text" indexed="true" stored="false" multiValued="true"/>
	<copyField source="text" dest="auto_suggest" />

Note: The auto_suggest field isn’t stemmed. Solr sends the raw, unanalyzed contents of the text field to the auto_suggest field, which allows a different text-analysis strategy.

Unique key field
----------------
If you provide a unique identifier field for each of your documents, Solr will avoid creating duplicates during indexing. In addition, if you plan to distribute your Solr index across multiple servers, you must provide a unique identifier for your documents. 
	<uniqueKey>id</uniqueKey>

Note: It’s best to use a primitive field type, such as string or long, for the field you indicate as being the <uniqueKey/> as that ensures Solr doesn’t make any changes to the value during indexing. We’ve seen instances in which Solr doesn’t return results correctly if you don’t use string as the type for text-based keys. Save yourself some trouble and use string or one of the other primitive field types for your unique key field.

Field types for structured nontext fields
-----------------------------------------

						     FieldType
							    |
							    |
						--------------------
						|				   |
						|				   |
				PrimitiveFieldType	   TextField

PrimitiveFieldType is the super class for all nontext field types. It has DateField, TrieFieldType, StrField and BoolField. TrieFieldType has subtypes TrieIntField, TrieLongField,TrieFloatField,TrieDoubleField. DateField also has subtype called TrieDateField. All the TrieFieldTypes support for efficient range search queries.

String fields
-------------
In out tweet example, we decided that screen_name,type, timestamp, and lang should be indexed fields. Consider the case of lang. It is a standard ISO-639-1 language code. We don’t want Solr to make any changes to it during indexing and query processing unlike text fields. At query time, you also need to pass the exact value(For ex en to match English documents).

	<fieldType name="string" class="solr.StrField" sortMissingLast="true" omitNorms="true"/>

Date fields
-----------
A common approach to searching on date fields is to allow users to specify a date range. Because searching within a date range is such a common use case, Solr provides an optimized built-in <fieldType> called tdate, shown next.

	<fieldType name="tdate" class="solr.TrieDateField" omitNorms="true" precisionStep="6" positionIncrementGap="0"/>

Solr uses a trie data structure to support efficient range queries and sorting of numeric and date values.

First decide if your users can search across a range of values. If yes, think about the range of possible values that will be indexed;are there potentially millions of unique values or only a handful? In Solr terminology, the number of unique values in a field is called the cardinality of the field.

Note: In Solr 7.x they introduced a new data structure inplace of trie called point fields.

















































































node states
-----------
Active
------
Active nodes are happily serving queries and accepting update requests. Active replicas are in sync with their shard leader. A healthy cluster is one in which all nodes are active.

Inactive
--------
Used during shard splitting to indicate that a Solr instance is no longer participating in the collection. Shards that get split enter this state once the splits are active.

Construction
------------
Used during shard splitting to indicate that a split is being created. Shards in this state buffer update requests from the parent shard but do not participate in queries.

Recovering 
----------
Recovering instances are running but can’t serve queries. They do accept update requests while recovering so that they don’t continue to fall behind the leader.

Recovery Failed 
---------------
The instance attempted to recover but encountered an error. In most cases, you will need to consult the logs and manually resolve the issue preventing the instance from recovering.

Down 
----
The instance is running and is connected to ZooKeeper but is not in a state in which it can recover, such as when Solr is initializing. A downed instance does not participate
in queries or accept updates. The down state is usually temporary, and the node will transition to one of the other states.

Gone 
----
The instance is not connected to ZooKeeper and has probably crashed. If a node is still running but ZooKeeper thinks it’s gone, the most likely cause is an OutOfMemoryError in the JVM.

CAP theorm
----------
The easiest way to understand CAP is to think of two nodes on opposite sides of a partition. Allowing at least one node to update state will cause the nodes to become inconsistent, thus forfeiting C. Likewise, if the choice is to preserve consistency, one side of the partition must act as if it is unavailable, thus forfeiting A. Only when nodes communicate is it possible to preserve both consistency and availability, thereby forfeiting P. 

Leader/Replica vs Master/slave
------------------------------
One of the key design principles in SolrCloud is that every node in the cluster performs indexing and executes queries. Contrast this with a master-slave architecture in which master nodes perform indexing, and slave nodes execute queries.

In practice, distributed system designers assume that failures will occur, so the P is a must; designers are forced to choose between consistency and availability.

Consistency means that an operation either succeeds or fails. When thinking about consistency in a distributed system like Solr, it boils down to the question of what can be expected from a read after a write. SolrCloud makes the assumption that once a write succeeds, all active replicas will return the same version of a document. This implies that update requests must succeed on all replicas, or the request fails. For instance, if shard1 has replicas A, B, and C, then the write is not considered successful
until all three replicas accept the update. If A and B work and C fails, the write fails.

The key point is that SolrCloud does not allow replicas participating in queries to have different versions of the same documents. The index may not be fully up to date because
writes are failing, but it will not return inconsistent results depending on which replicas participate in a query.

Contrast this with a system that favors write availability over consistency. In this case, the system would accept update requests on any replica for a shard, and the replica would try to propagate the update to the other replicas in the same shard. If the update fails on some of the replicas, the write may still be considered successful. Some systems allow the client application to specify their tolerance for this weak consistency in favor of having highly available writes; this is commonly known as eventual consistency because replicas eventually have a consistent state.

Solr uses ZooKeeper for three critical operations:
■ Centralized configuration storage and distribution
■ Detection and notification when the cluster state changes
■ Shard-leader election

ZooKeeper data model
--------------------
ZooKeeper organizes data into a hierarchical namespace similar to a filesystem. Each level in the hierarchy is called a znode. Each znode encapsulates basic metadata such as
creation time and last-modified time and can also store a small amount of data. Here is the sample from solr admin tree(under cloud).

 /
  /aliases.json
  /autoscaling
  /autoscaling.json
  /clusterstate.json
  /collections
  /configs
  /live_nodes
  /overseer
  /overseer_elect
  /security.json
  /solr.xml
  /zookeeper

Note:ZooKeeper is not a general-purpose data-storage system; it’s intended for storing small bits of metadata that need to be accessed by a large number of distributed servers.

Ephmeral Znode
--------------
A central concept in ZooKeeper is the ephemeral znode, which requires an active client connection to keep the znode alive. When a Solr node joins the cluster, it creates an ephemeral znode under the /live_nodes node.

Here is the sample:
  /live_nodes
  10.194.156.41:8984_solr
  10.194.156.45:8984_solr

Note: Solr keeps an active connection to this /live_nodes using the ZooKeeper API. If Solr crashes, the connection to the ephemeral znode is lost, causing that node to be considered gone. When the state of a znode changes, ZooKeeper notifies the other nodes in the cluster that one of the nodes is down. This is important so that Solr doesn’t try to send distributed query requests to the failed node.

ZNODE WATCHER
-------------
Any client application can register itself as a watcher of a znode. If the state of the znode changes, ZooKeeper will notify all registered watchers of the change. For instance, solr registers itself as a watcher of the /clusterstate.json, so that it can receive notifications when the state of the cluster changes.

ZOOKEEPER CLIENT TIMEOUT
------------------------
Solr creates ephemeral znodes when it joins a cluster to indicate there is a live node. If a Solr node crashes, ZooKeeper will detect the crash after the timeout period. Consequently, you want the timeout to be as brief as possible to ensure the cluster state managed in ZooKeeper reflects the current state of your cluster. The default timeout in Solr is 15 seconds. To change it, you can set the zkClientTimeout parameter to an integer value in milliseconds: -DzkClientTimeout=30000, for example, will change the timeout to 30 seconds.

Full garbage collection and the ZooKeeper client timeout
--------------------------------------------------------
The JVM pauses all executing threads when running a full garbage collection, including the thread that keeps the ZooKeeper session alive. It follows that if a full garbage collection takes longer than the ZooKeeper client timeout, then the instance will appear as being offline to ZooKeeper. This is a good thing because the instance cannot accept requests while it is performing full garbage collection. Solr will attempt to reestablish the ZooKeeper connection once the full garbage collection pause is over.

CENTRALIZED CONFIGURATION STORAGE AND DISTRIBUTION
--------------------------------------------------
When you bootstrap a new collection in a SolrCloud cluster, you need to upload the configuration files such as solrconfig.xml and schema.xml into ZooKeeper. When Solr launches in cloud mode, it will pull the configuration files from ZooKeeper automatically.

Shard-leader election
---------------------
A shard leader is responsible for accepting update requests and distributing them to replicas in a coordinated fashion. Specifically, the shard leader provides the following
additional responsibilities for handling update requests beyond a replica:
■ Accepts update requests for the shard
■ Increments the value of the _version_ field on the updated document and enforces optimistic locking
■ Writes the document to its update log
■ Sends the update (in parallel) to all replicas and blocks until a response is received

Solr uses ZooKeeper sequence flags to keep track of the order in which nodes register themselves as candidates to be the leader. Sequence flags are incremented in an atomic fashion, so it doesn’t matter how many clients attempt to increment the sequence concurrently.

LEADER VOTE WAIT PERIOD
-----------------------
The leaderVoteWait parameter provides a safety mechanism intended to prevent a node with stale data from automatically assuming the leader role until other nodes hosting the same shard have had a chance to vote to be the leader.

Consider the scenario where there are two instances hosting a shard; specifically node X is the leader and node Y is a replica of shard1. Now imagine that node Y crashes but node X continues to accept update requests as the leader. In this scenario, node Y becomes out of sync with the shard leader while it is offline. If node X crashes before node Y recovers, then node Y should not immediately assume the role as the shard leader when it is restarted because node Y has stale data compared to node X. The leaderVoteWait parameter, which defaults to three minutes, protects against situations described in this fictitious scenario. If node X comes back online within the leaderVoteWait period, it will resume the leader status and node Y will recover as a replica. This only protects the case where node X and Y are offline for a short period of time, of course.

The implication of this safety mechanism is that node Y will not become active in the cluster within the leaderVoteWait period if node X is offline. Specifically, node Y will enter a waiting period to see if other nodes for the same shard join the cluster. This implies that shard1 will remain offline until node X rejoins the cluster or until the leaderVoteWait period ends. This situation only occurs when all nodes hosting a shard go offline, which can be mitigated by using more replicas per shard.

Note:
The above approach favors write-availability at the potential expense of lost consistency. In the above scenario when the replica is down, leader(X) might have received update requests. Because replica(Y) is down, it has updated the document. By the time Y recovers, X has crashed and waiting to become leader. If X has not recovered with in the wait time, then Y becomes leader. Now Y has no updated data even though it is the leader.

Distributed Indexing
--------------------
When a new document is indexed in Solr, it needs to be assigned to one of the shards. Solr uses a component called a document router to determine which shard a document should be assigned to. There are two basic document-routing strategies supported by SolrCloud: compositeId (default) and implicit.

Implicit
--------
Implicit strategy places all of the routing logic on your application client code. Whichever shard you indicate on the indexing request (or within each document) will be used as the destination for those documents.

If you created the collection and defined the "implicit" router at the time of creation, you can additionally define a router.field parameter to use a field from each document to identify a shard where the document belongs. If the field specified is missing in the document, however, the document will be rejected. You could also use the _route_ parameter to name a specific shard.

Composite
---------
When a collection is created, we mention numShards. For example, if numShards are 2, then it assigns each shard a range of 32-bit hash values. It assigns a range of 80000000-ffffffff to one shard and a range of 0-7fffffff to another shard. The default compositeId router computes a numeric hash of the unique ID field in a document and assigns the document to the shard whose range includes the computed hash value.

Solr uses murmur hash algorithm to create a unique documentId and also to uniformly distribute documents. To have a balanced distribution is must because If one shard has more documents that becomes a bottle neck for the search. We know distributed queries are only as fast as the slowest shard to respond.

Custom routing with composite hash
----------------------------------
If you use the compositeId router (the default), you can send documents with a prefix in the document ID which will be used to calculate the hash Solr uses to determine the shard.

For example, if you want to co-locate documents for a customer, you could use the customer name or ID as the prefix. If your customer is "IBM", for example, with a document with the ID "12345", you would insert the prefix into the document id field: "IBM!12345". The exclamation mark ('!') is critical here, as it distinguishes the prefix used to determine which shard to direct the document to.

Note: We cannot influence in which shard to save the document. It is upto solr. The prefix is only used to identify a shard for searching the documents.

Then at query time, you include the prefix(es) into your query with the _route_ parameter (i.e., q=solr&_route_=IBM!) to direct queries to specific shards. In some situations, this may improve query performance because it overcomes network latency when querying all the shards.

The compositeId router supports prefixes containing up to 2 levels of routing. For example: a prefix routing first by region, then by customer: "USA!IBM!12345"

Another use case could be if the customer "IBM" has a lot of documents and you want to spread it across multiple shards. The syntax for such a use case would be: shard_key/num!document_id where the /num is the number of bits from the shard key to use in the composite hash.

So IBM/3!12345 will take 3 bits from the shard key and 29 bits from the unique doc id, spreading the tenant over 1/8th of the shards in the collection. Likewise if the num value was 2 it would spread the documents across 1/4th the number of shards. At query time, you include the prefix(es) along with the number of bits into your query with the _route_ parameter (i.e., q=solr&_route_=IBM/3!) to direct queries to specific shards.

If you do not want to influence how documents are stored, you don’t need to specify a prefix in your document ID.

LIMITATIONS OF CUSTOM HASHING
-----------------------------
The biggest concern when using custom hashing is that it may create unbalanced shards in your cluster. For this reason, you will need to understand your data to determine if you need to further subdivide shard keys or avoid their use altogether.

Send the update requests using CLOUDSOLRSERVER class of solrj
-------------------------------------------------------------
CloudSolrServer connects to ZooKeeper to get the current state of the cluster. CloudSolrServer knows which nodes are shard leaders. Because update requests must be routed to shard leaders before their replicas, CloudSolrServer can save time by sending them directly to a shard leader instead of to a replica.

CloudSolrServer provides basic load balancing and retry logic on the client side. If a node crashes while your client application is indexing documents, CloudSolrServer will get a notification from ZooKeeper that the node is unavailable so it can stop sending requests to that node.

Note:All of this happens automatically behind the scenes because CloudSolServer registers a watcher on the /clusterstate.json and /live_nodes znodes in Zookeeper.

NRT(near real time search)
--------------------------
NRT makes documents visible in search results within seconds of their being indexed, hence the use of the near qualifier. To allow documents to be visible in NRT, Solr provides
a soft commit mechanism, which skips the costly aspects of hard commits, such as flushing documents stored in memory to disk.

The design decision to send updates to all replicas is largely driven by the need to support NRT search. In contrast, a master-slave setup cannot support NRT search because the intent of the search is to make documents visible in search results within about one second of their being added to the index. Master–slave-based replication relies on moving entire segments from the master to the slave. If Solr had to replicate segments to slaves every second, your engine would build up many small segments, and query performance would suffer greatly.

Enabling NRT behavior simply requires a configuration change in solrconfig.xml.
	<autoSoftCommit>
		<maxTime>1000</maxTime>
	</autoSoftCommit>

The above setting issues a soft commit every 1000 milliseconds. It means leader transfers updates to replicas every 1000 milliseconds(or 1 sec).

Note:When you perform a soft commit, Solr must open a new searcher to make the soft committed documents visible in search results. This implies that Solr must also invalidate
all caches to remain consistent with the changes applied by the soft commit. When opening a new searcher after a soft commit, Solr warms caches and executes warming queries configured in solrconfig.xml. Consequently, this implies that your cache autowarming settings and warming queries must execute faster than your soft commit frequency.

Note:
Although NRT search is a powerful feature, you do not have to use it with SolrCloud. It’s perfectly acceptable to not use soft commits, and we recommend not using
them unless you really need indexed documents to be visible in near real-time. One of the drawbacks to using soft commits is that your caches are constantly being invalidated.

Distributed search
------------------
Client sends query to any node. The node that receives the initial request is the query controller (or aggregator) and is responsible for creating a unified result set to return to the client.

The query controller needs to know the status of all nodes in the cluster, which it gets from ZooKeeper. Incase of a failure node, it retries requests on other nodes for the same shard. The query controller sends a nondistributed query (distrib=false) to every shard to identify matching documents in the shard.

The query that is sent to every shard only requests the id and score fields. It does this to avoid reading stored fields prematurely. If each shard gives 10(default row size) records, then the query controller waits until final 10 records are identified based on score. Once the matching documents are identified, then query controller sends requests to a subset of nodes to get the rest of the fields needed to fulfill the request.

Note:
If you only need the IDs of the documents, this second query to get additional fields is not needed.

Note: 
If there are no servers hosting a shard, then request will fail, because Solr chooses to not return an incomplete result set for your query. You can override this behavior by specifying shards.tolerant=true as a query parameter to indicate that you’ll accept incomplete results.























 
