In this first article of a possible series for building [grappletree.com](http://www.grappletree.com), I am dealing with the question on how to modify and display lineage with neo4j. As I am writing these blog posts, I am building grappletree.com. If I have to name a single technology that is most important in building this site, I'd have to name the graph database Neo4j.

Since posting my first AirPair article with the title [The painful journey of painless deployments
](https://www.airpair.com/docker/posts/the-painful-journey-of-painless-deployments), Docker has only increased in its significance in my development and deployment process. For this article however, we won't be going to much in detail on how to use Docker. At this time, we also won't be using a Docker container for Neo4j, but we'll be using a milestone release.

##A Lineage Example
To display lineage, I'm answering one of the most important questions for beginners in the martial arts: "What's your belt and who gave it to you?"

The application we're building will be hosted on grappletree.com, therefore we're capturing (brazilian) jiu jitsu, luta livre, submission wrestling, grappling (...) rankings.

![grappletree whiteboard](http://16d52.http.sjc01.cdn.softlayer.net/grappletree_airpair_article/grappletree_whiteboard_simple.JPG)

Later on we can also capture more relationships between people. Who fought who at what time and what was the outcome. Who had the most victories via *(insert your favorite submission here)*. What's the shortes path between two grapplers. There's a lot of data and interesting facts that we can collect through the usage of Neo4j.

##Setting up and starting Neo4j
Neo4j 2.3 is going to have some great improvements. In order to get an early peak, I've downloaded the Neo 2.3.0-M02 release for Mac as a tar archive from [http://neo4j.com/download/](http://neo4j.com/download/). After unpacking, I've disabled Neo4j's basic authentication in the ```config/neo4j-server.properties``` file of the unpacked Neo4j archive. To do that, set ```dbms.security.auth_enabled``` to ```false``` (line 24). In production we will secure access to the database in a different matter and we don't want to deal with the overhead of authentication.

To start Neo4j, run ```bin/neo4j start``` from within the unpacked Neo4j directory.

You can then access Neo4j's web interface via http://localhost:7474.
![accessing neo4j via web interface](http://16d52.http.sjc01.cdn.softlayer.net/grappletree_airpair_article/Screen%20Shot%202015-06-18%20at%201.06.45%20PM.png)

###Adding some data
From within the web interface, let's create some data.

```
CREATE (eddie:Black {first_name: 'Eddie', lastName: 'Bravo'}) CREATE (jjm:Black {first_name: 'Jean', middle_name: 'Jacques', last_name: 'Machado'}) CREATE (matthias:Blue {first_name: 'Matthias', last_name: 'Sieber'}) CREATE (verena:Blue {first_name: 'Verena', last_name: 'Sehring'}) CREATE (martin:Blue {first_name: 'Martin', last_name: 'Audorff'}) CREATE (jjm)-[:PROMOTED_TO_BLACK]->(eddie) CREATE (eddie)-[:PROMOTED_TO_BLUE {year: 2010, month: 4, day: 30}]->(matthias) CREATE (matthias)-[:PROMOTED_TO_BLUE {year: 2011, month: 5, day: 24}]->(verena) CREATE (matthias)-[:PROMOTED_TO_BLUE {year: 2011, month: 8, day: 30}]->(martin) RETURN eddie, jjm, matthias, verena, martin
```

![grappletree neo4j exmple](http://16d52.http.sjc01.cdn.softlayer.net/grappletree_airpair_article/grappletree_neo4j_example.png)

You'll notice that Jean Jacques Machado and Eddie Bravo have different colors. These two outstanding grapplers are both black belts. Jean Jacques promoted Eddie to black belt, while Eddie promoted me to blue belt in 2010. One year later I was allowed to promote my (now former) students to blue belt, which is reflected in this graph as well.

This is the very simple graph that we're going to work with for the rest of this article.

##The Back End
The source code for this article can be viewed on [GitHub](https://github.com/manonthemat/Grappletree/tree/article1). After cloning the repository, run ```npm install --production``` from within the marcelo directory (named after one of the best back-takers in Jiu Jitsu: Marcelo Garcia) to install the dependencies.

```MARCELO_DB_PROTOCOL=http MARCELO_DB_HOST=127.0.0.1 MARCELO_DB_PORT=7474 node server.js``` will start the hapijs server and use the running Neo4j instance.

Curl, browse to or make a http get request to http://localhost:8000/grapplers to get a list of all the grapplers in the database.

The resulting Cypher query will be this: ```MATCH (grappler) RETURN grappler ORDER BY grappler.last_name```.

At this point, you also have the ability to get a list of all grapplers filtered by belt. To filter by belt (blue, purple, brown, black), use the belt color as a query param. The value can be "true" or "yes" to include that specific color. While http://localhost:8000/grapplers includes all grapplers, http://localhost:8000/grapplers?blue=yes will return a list of only blue belts, while http://localhost:8000/grapplers?brown=yes&black=yes will return a list of all the grapplers having a current brown (none in this database) or black belt.

```
// result of querying http://localhost:8000/grapplers?brown=yes&black=yes
[{"grappler":{"last_name":"Machado","first_name":"Jean","middle_name":"Jacques"}},{"grappler":{"first_name":"Eddie","lastName":"Bravo"}}]
```

##Next steps
We'll further build out the API for querying the database through the back end service "marcelo", which will be dockerized. Furthermore we're going to start working on the front end with Google Polymer 1.0 (in the style of [dockerized software development](https://www.airpair.com/docker/posts/dockerized-software-development) and deploy all services.
