[[toc]]
+ What is a Search Engine?

A search engine is a web-based program which identifies web-pages that correspond to the search terms supplied by the user and returns a list of links to those web documents, sorted on the basis of their usefulness. The Algorithms that decide the usefulness of a specific web page to the user form the backbone of any search engine. Some popular examples of Search Engines are Google, Yahoo, Bing etc.

+ How a Search Engine works?
The following diagram gives the basic structure of a typical Search Engine :  
[[=]]
[[image https://s18.postimg.org/giueg9sft/search_engine_arch.png]]
[[/=]]
A Search Engine mainly consists of the following parts : -
* Interface
* Database
* Crawler / Spider
* Indexer

+++ Interface
Most Search Engines feature a concise interface composed of just a textbox and a search button. The user enters the keywords to be searched in the textbox and with the click of "Search" button the query is passed on to the search engine's server where it is processed to return a sensibly sorted list of web pages containing those words.

+++ Database
The database stores the index of a large number of web pages on the World Wide Web in a suitable form which facilitates quick querying of page URLs corresponding to user's search field. The organisation of data in the database may vary from one search engine to another as it is one of the major factors that effect the efficiency of a search engine.

+++ Crawler
Crawler/ Spider is a program used by the search engines to scrap the documents on the World Wide Web. The crawler starts by visiting the list of URL's (called seeds) provided to it manually and recursively visits all the web pages these pages are linked to. This is done by suitable graph search Algorithms (Breadth First Search or Depth First Search). Also there is a threshold depth 'D' after reaching which the graph search halts. This is to ensure that the crawler program doesn't take infinite time to terminate. While visiting these documents the crawler either stores the plain text contained in them as separate files on server or sends the scraped string directly to the indexer to be processed and stored in the database. 

+++ Indexer
The most important part of any Search Engine is its Indexer. Indexer is the piece of code which stores the web page data scraped by the crawler in a suitable form such that the more useful query results can be identified at the time of search. This usefulness level maybe measured in terms of frequency of searched word in a document, number of links leading to that web page and several other factors which might be different for different search engines.

+ Our Plan
We will be using HTML5., CSS and Bootstrap for designing the interface for the search engine. The backend server will run PHP scripts and store data in a phpmyadmin database. Indexing and crawling will also be done using PHP only.
We will start by building a naive version using just a frontend interface design and a manually created database of page indices in the backend. The organisation of the data in database and the algorithms deciding the page rank will be improved gradually moving towards a better and more efficient version of the search engine with each update.

+ Version 1.0
This version consists of just a minimal front end design consisting of a text box and a search button.
 
[[=]]
[[image https://s13.postimg.org/u98k26e1z/Homepage.png]]
[[/=]]
The backend server stores a manually created database with a table "search". The table has two columns namely - "Words" and "URL". As the name suggests these columns will map different words to the URLs of the pages that contain them. The columns don't define a map, rather they define a multimap where the same word can have more than one URL's related to it, also same URL can be mapped to different words). Infact there may also be duplicate entries corresponding to the different occurrences of same word on the same URL.
[[=]]
[[image https://s21.postimg.org/xvho4inwn/Database.png]]
[[/=]]

The front end page will send the textbox data to the following script which after comparing it to the existing index in the database will return the links to related web pages.

[[=]]
[[image https://s17.postimg.org/4wfiy29vz/Script_To_Query_The_Backend_Database.png]]
[[/=]]
+++ Demonstration

When I search for a word, say "Foobar", 
[[=]]
[[image https://s13.postimg.org/4c4zy3tw7/Search2.png]]
[[/=]]
I am redirected to a page containing the links to related web pages.

[[=]]
[[image https://s21.postimg.org/640rixmuf/Result2.png]]
[[/=]]
+ Version 2.0
The previous version was just a naive version of what our search engine would look like .Wherein we were trying to get comfortable with the idea of using html ,php and acessing the database .Now we look into the finer details of building a search engine .  The advanced version of our search engine uses the following structure :- 


+++ __Crawling and Indexing__

The Crawling and Indexing in the version 2.0 is taking place in the following way:-
[[f<image https://s15.postimg.org/o8uxiaynf/flowchart.png ]]


[[html]]
<br>
[[/html]]

* **Break into tokens** - First step is to break the page content into a list of words
* **Stemming** - Remove punctuation and special characters. Retrieve a list of root words corresponding to all the words in the list
* **Stop Listing** - Remove the common unimportant words from the list of stem words. For example:- words like 'is','the' etc. are not very important in search queries.
* **Term Weighing** - Assign weights to extracted stem words based on the number of times they appear in the web page.
[[html]]
<br>
[[/html]]

------

Typically, a search engine works by sending out a 'crawler' to fetch as many webpages as possible. The web crawler scans the website, extracting page titles, descriptions, keywords, and links  and add the information to the huge database. For the web page parsing part we have made use of an external script called as the "simple_html_dom". This script allows us to break down an html web page into a class object with various useful fields and methods.
The document database in our project  stores the following:-
# The Document ID (an automatically incrementing field , acts as a unique identifier for any document)
# The Document URL (String)
# Title of the web page (String)
# Content of the web page (String)
# JSON string of the associative array (**stem word** vs frequency) for the page title
# JSON string of the associative array (**stem word** vs frequency) for the page content
The purpose of adding the json string to database can be understood in the following sections of **similarity checking** and **stemming**.

The working of the crawler can be realised through a Breadth First Search (BFS) Algorithm ,where rather than representing a node by a number ,we represent it with a URL.

[[=]]
[[image https://s13.postimg.org/3t7bb0u13/crawler.png]]
[[/=]]
-Include the simple HTML DOM helper file.

-Set the target URL as (say) X.

-Create a new simple HTML DOM object to store the target page.

-Load our target URL into that object

-For each link <a> that we find on the target page, Add the the HREF attribute as the adjacent nodes to the present URL node.

To initiate the BFS we need to provide it a list of seed URLs or links .Each of these web pages acts as the root for our BFS. For these we maintained a table in our database called "Crawler_Seeds" .
This table looks as follows:

[[=]]
[[image https://s18.postimg.org/6xfzxn6op/Capture.png]]
[[/=]]
As it can be seen that this table contains only one entry i.e. "iiitd.ac.in". This is done essentially because it wasn't practically possible for us to crawl a very large number of pages catering to the lack of high network bandwidth and time. Crawling a single web site is good enough to mimic the functioning of a large scale search engine and we have decided to do that with the institutes website itself. The BFS is carried out only upto a certain depth starting with depth 1 at seed URL.
This logic seems pretty straight forward. However there are number of things that we need to keep in mind during execution.

++++ 1) URL Format
The href attribute of an 'a' tag is not always a complete valid web page URL. So, these href attributes cannot be added directly to the "to be visited" list of the Breadth First Search Algorithm.
* **In page link URLs**
Start with '#' and have no significance to a crawler
* **Dynamically generated URLs**
There are situations when a page has a tendency to generate different versions of itself  based on  different HTTP GET parameters. These URLs differ in the part that follows the '?' character, which indeed contains the GET request parameters. For Example :-

https://www.google.co.in/?gfe_rd=cr&ei=-iw8WMzGHuXI8AfOn6awBw 

Such dynamic web pages can be extremely large in number and that's the reason why we would prefer ignoring them. This is done by trimming the GET request part of the web page URL. Of course this might not be the best way of ruling out dynamic web pages but is good enough for our present requirement.
* **Relative Web Page Addresses**
Most of the websites don't use the full name of the linked webpage in the href attribute of 'a' tag. In such cases we must make the following conversions using string slicing operations.
[[=]]
[[image https://s12.postimg.org/bg9az0hx9/url.png]]
[[/=]]

++++ 2) URL Existance
Many a times it happens that a website removes a particular link or modifies a link. In this situation we get an error message of page not being found .There can be more such cases where the page shows some HTTP error code for not displaying the page. Hence we need links which have their status code "200 OK" .
Moreover many a times, the link given points to  a .pdf/.ppt etc. file and not an html page. Our crawler won't work on these pages since the 'simple_html_dom' parser can parse only html content. Hence we need to avoid these URLs as well.
In order to overcome this , we analyse the HTTP headers returned by the URL. If we are able to find a substring of the type "html" or "200 ok" in some header parameter, then we are good to go. Otherwise we drop that link.
[[=]]
[[image https://s16.postimg.org/m21b1dglx/url_exists.png]]
[[/=]]

++++ 3) Similarity with stored documents
Even if a URL represents a valid html page. There is a possibility that it's content is almost similar to some other document which has been stored on the database. To assure zero redundancy, each time before adding a new document, we make a check that it isn't more than 90% similar to any other existing document in the database. The **document similarity** section explains how this is achieved.

Only if a URL passes all three of the above tests, its contents are added in the specified format to the "Documents" Database.
The final Document database looks as follows:-

[[=]]
[[image https://s22.postimg.org/apve722wh/screenie1.png]]
[[/=]]
------
+++ __Breadth First Search__
As stated above our crawler uses the Breadth-first search (BFS) algorithm. Breadth-first search is an algorithm for traversing or searching tree or graph data structures. It starts at the tree root or some arbitrary node of a graph, sometimes referred to as a "search key" and goes to the neighbor nodes first, before moving to the next level neighbors.

Suppose we consider the graph below. This is the order in which the nodes of the graph will be traversed.
[[=]]
[[image https://s21.postimg.org/vaqh4e8af/bfs.gif]]
[[/=]]
A simple description of the algorithm for a BFS is this:
1.	Start at some node ‘i’. This is now our current node.
2.	State that our current node is ‘visited’.
3.	Now look at all nodes adjacent to our current node.
4.	If we see an adjacent node that has not been ‘visited’, add it to the queue.
5.	Then pull out the first node on from the queue and traverse to it.
6.	And go back to step 1.

Here is the pseudocode for running a breadth-first search

unmark all vertices
    choose some starting vertex x
    mark x
    list L = x
    tree T = x
    while L nonempty
    choose some vertex v from front of list
    visit v
    for each unmarked neighbor w
        mark w
        add it to end of list
        add edge vw to T

------
+++ __Stemming__
It is a process of using linguistic analysis to reduce a word to its root word (commonly known as a **stem**).

A root word is so called because it is deprived of any form of suffixes and prefixes, which carries most of the soul of semantic contents, and cannot be fragmented further.
It is not an exercise in etymology or grammar. In fact from an etymological or grammatical viewpoint, a stemming algorithm is liable to make many mistakes.
This technique of contracting a word to its root is greatly exploited by the web developers. It constitutes an important portion of a word search via a search engine. 

Imagine you want to gather information about “fishing”, so when you use search engine and type in “fishing”, what is done is the word “fishing” is compressed to word “fish”. Similarly, the words “fished” “fisher” etc are all reduced to root word “fish”. 
Interestingly, the words "argue", "argued", "argues", "arguing", and "argus" reduce to the stem "argu" although the stem is not itself a word or root. 

On the surface this looks like slicing a piece out of a cake, but in reality there are lots of mathematical algorithm that go into its working.
Here we are using Porter Stemmer algorithm, which effectively utilizes suffix stripping but ignores the prefixes part. It was developed in 1980’s and offers a great trade-off between speed, accuracy and readability:

**Step1**:- Getting rid of plurals and –ed or –ing suffixes. For example- “possesses”->“possess”, “interesting”>“interest” etc
**Step2**:- Turning terminal y to i whenever there is another vowel in the stem. For example- “Furry”>“furri”, “colly” > “coolli” but “fry”>“fry”.
**Step3**- compressing double suffixes to single ones. For example- “operational”>“operate”, “playfulness”>“playful”
**Step4**:- deals with suffixes like –full, -ness etc. For example:- “practical”> “practic” etc
**Step5**:- removes –ant, -ence etc. For Example:- “precedent”>“preced”
**Step6**:- finally removes –e (if any). For example:-“parable”>“parabl” 

Now let’s see what happens to “semantically” when it is passed through the above algorithm:

[[=]]
[[image https://s14.postimg.org/petz7oz29/stemming.png]]
[[/=]]
This way the word “semantically” gets reduced to the stem word “semant”.
In our project in most of the situations we are dealing with only the stem words and not the actual words of the document. The first step of any operation be it similarity checking or querying the first step is to break down the string into a list of stem words.

+++ __Stop listing__
In computer search engines, list of words that are not to be added is called a stop list. It is used to strip those out from a search query before performing a search and presenting search results to a searcher.
Why is it used?
Stop words (irrelevant keywords) could be responsible for the changing search results, to avoid such complications, stop listings comes to our rescue. It saves our time and space by avoiding or ignoring terms at the time of indexing. 
How is it implemented?
List of stop words that are stored in a database, are ignored at the time of indexing. It basically, compares index terms against a stop word list and eliminates certain terms from inclusion in the index for searching.
Stop List Database:-

[[image https://s16.postimg.org/jwl8vfvnp/stoplist.png]]
------
For the sake of simplicity we did both stemming and stop listing together .
[[=]]
[[image https://s22.postimg.org/5cbnl0y1d/stemming.png]]
[[/=]]
* **Strip** - Remove all punctuation and special characters.
* **Porter** - An implementation of the **Porter Algorithm** that returns the stem words from the passed words.
* **get_stop_list** - Reads the Stop List Database and returns a list of stop words, which are then removed from the search query.

------
+++ __Indexer__
-The indexer is the part of code which stores the Documents database into an inverted index. An inverted index refers to storing the documents as a mapping of their content (words) to their location (the Document IDs).
-In our case the indexer simply reads the word vs frequency json object from the Documents database and stores it in the index database, such that each word is mapped to its document ID and the number of times it appears in the specific document.

Index Database :-

[[image https://s18.postimg.org/rp9osa8t5/index.png]]
+++ __Querying and Displaying the results__

-The keywords are broken into their stem distribution associative array (Stem Words vs Frequency mapping).
-The Documents database is queried for the IDs of all the documents containing atleast one of the entered keywords atleast once. Hence, we get a list of related documents to the keyword. Though this list is not ordered.
-The fetched list of document IDs is passed to the ranker function which in turn sorts them based on their relevance.
-The returned list is processed and displayed in a suitable fashion (specifically previewing certain sections of document containing the highlightened keywords).

------

+++ __Ranker__
Once we have in our database a good set of crawled webpages and Indexed Documents ie.maintained a word v/s document ID table .We are set to answer the user's search query.
Question is what should be the basis or the order of significance of search results? 
For this we decided upon a algorithm for Ranking the seach results.
Each search result is given a 'relevance number' based on our formula (The page with higher 'relevance number' is more relevant) :-
[[=]]
[[image https://s15.postimg.org/3z8d8p1mz/ranker.png]]
[[/=]]
The way in which **document similarity** is calculated is explained in the following section.

In the above formula, we give 3 times weightage to **similarity with title** as compared to **similarity with content** because obviously a page showing greater similarity between title and search query will be more relevant than the one sharing common words with the content.
------

+++ __Document Similarity__ 
Basically we want an algorithm to check the similarity of 2 documents .
For this we chose to represent the documents as an associative array (or dictionary) having a mapping from words to their frequency in the document.Moreover these words were in their stemmed form.(refer stemmer)

**NOTE:-** Now we understand the reason for storing the JSON string of **stem words vs frequency mapping** in the "Documents" database. Since that object can be directly fetched from database for similarity detection.

Now the algorithm:-
Once we have the words v/s frequencies input of individual document (the way we have stored content and title in the database), we can think of the document as a vector ,a n-tuple of frequency of the various words.
So we can now imagine them as vectors in an n dimentional vectorspace.Where the dimention of a document is represented by its stem-words. 
consider the following 2 documents:

Doc1:cat eat rat
Doc2:Dog cat rat run Dog
[[=]]
[[image https://s22.postimg.org/hfe6d9stt/docsimilarity.png]]
[[/=]]
We chose to calculate the cosine of the angle between the two document vectors .It gives us an idea about the level of similarity,a value lying between [0,1]. Greater the value of the cosine greater the similarity.
It is to note that the numerator depends only on the words common to both documents,while the denominator takes into consideration the entire magnitude of the document vectors .Hence it is a useful formula for cases where even though the frequency of common words is same and yet the documents are not.

------
+ Demonstration

Link to our Search Engine :  [http://sid-dhawan.eu5.org/Quest.html QUEST]
Searching for the keywords "Student Life"


[[=]]
[[image https://s15.postimg.org/frun8z4ej/Screenshot_from_2016_11_27_23_57_09.png]]
[[/=]]
Give the following results:-
[[=]]
[[image https://s15.postimg.org/41gprlbm3/Screenshot_from_2016_11_27_23_57_27.png]]
[[/=]]
+ The Next Level

We have plans to further expand this project during the winter. Wherin we will add more algorithms to determine the rank of a page in the search result query (apart from the document similarity function used till now.)
Some of these are :-

- A Proximity detecter, which will assign a number to every search result based on the distance at which the query words appear in it. For this we are planning to use a similar approach as the longest common subsequence algorithm.
- A ranking algorithm that uses inlinks and outlinks from a page to determine importance.
- An algorithm which creates a graph of related words and shows suggestions at the time of search by finding the shortest distance nodes from the existing search words.


