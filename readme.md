### This is example for [Neo4jreactivity](https://github.com/VeliovGroup/ostrio-Neo4jreactivity)

From this repository you will find out how to use Neo4j and Meteor within [Neo4jreactivity](https://github.com/VeliovGroup/ostrio-Neo4jreactivity) atmospherejs.com package

## Issues
Currently we met some weird issues within downloaded, not cloned, version of repository, (and within fresh install of Meteor) when meteor uses older versions of packages on first `meteor` command run. If you met any issues like `method "..." not found` or `"..." package removed`, just run (inside `Meteor-Leaderboard-Neo4j` folder):
```bash
meteor remove ostrio:neo4jreactivity
meteor add ostrio:neo4jreactivity
```

### Install
###### Neo4j
Pick up Neo4j version from [Official Neo4j Website](http://neo4j.com/download/)
After download - unzip and place into root folder of your user (name it neo4j), then in Terminal run:
```bash
cd ~/neo4j #downloaded and replaced folder
bin/neo4j start
```
* Note: __Neo4j database should be running before Meteor__ *

###### Node.js
Pick up version for your system on [Official Node.js Website](http://nodejs.org/download/)
Follow installer instructions

###### Meteor
Run this in Terminal:
```bash
curl https://install.meteor.com/ | sh
```

###### Neo4j NPM Package
To install Neo4j NPM Package, run this in Terminal:
```bash
npm -g install neo4j
```

###### Clone this repository
Run in Terminal:
```bash
git clone git@github.com:VeliovGroup/Meteor-Leaderboard-Neo4j.git
cd Meteor-Leaderboard-Neo4j
```

###### Running meteor
Run a line below in Terminal, inside ```Meteor-Leaderboard-Neo4j``` folder:
```bash
meteor
```

###### What next
 - In your browser go to [localhost:3000](http://localhost:3000/)
 - To see Neo4j browser, go to [localhost:7474](http://localhost:7474/)


------


### How we rewrite Leaderboard example to be used with Neo4j
In article below you will understand the basics of `ostrio:neo4jreactivity` and `ostrio:neo4driver` packages. How to translate queries and other code from MongoDB to Neo4j. How to write basic query in Neo4j. How to use reactivity in Neo4j.

#### Meteor’s Leaderboard example app, driven by Neo4j database
We've decided to take very basic example application at Meteor - "Leaderboard" and move it from MongoDB to Neo4j.

##### Prepare dev stage:
###### Create example app 
First of all download Meteor and Neo4j onto your working machine, then run:
```shell
$ meteor create --example leaderboard
$ cd leaderboard
$ meteor
```

###### Run Neo4j DB
Run inside of Neo4j directory:
```shell
$ bin/neo4j start
```

###### Installing Neo4j Meteor’s drivers
```shell
$ meteor add ostrio:neo4jreactivity
```

-----

##### Understanding the packages
After installing `ostrio:neo4jreactivity` package - we have next variables:
 - `Meteor.Neo4j;`
 - `Meteor.N4JDB;`
 - `Meteor.neo4j;`

###### var Neo4j;
```javascript
/* 
 * Server only
 * @class
 * @name Neo4j
 * @param url {string} - url to Neo4j database
 * Note: It’s better to store url in environment 
 * variable, 'NEO4J_URL' or 'GRAPHENEDB_URL' - 
 * so it will be automatically picked up by our driver
 * 
 * @description Comes from ostrio:neo4jdriver package 
 * Run it to create connection to database
 */
Meteor.N4JDB = new Neo4j();
```

Newly created object has next functions, you will use:
```javascript
/* @name query */
Meteor.N4JDB.query('MATCH (n:User) RETURN n', opts /* A map of parameters for the Cypher query */, function(err, data){
    Session.set('allUsers', data);
});

/* @name listen */
Meteor.N4JDB.listen(function(query, opts){
    console.log('Incoming request to neo4j database detected!');
});
```

###### var neo4j;
```javascript
/* Both (Client and Server)
 * @object
 * @name neo4j
 * @description Application wide object neo4j
 *
 */
Meteor.neo4j;
Meteor.neo4j.allowClientQuery = true; /* Allow/deny client query executions */
Meteor.neo4j.connectionURL = null; /* Set custom connection URL to Neo4j DB, Note: It’s better to store url in environment variable, 'NEO4J_URL' or 'GRAPHENEDB_URL' - so it will be automatically picked up by the driver */
```

`neo4j` object has multiple functions, you will use:
```javascript
/* @namespace neo4j.set
 * @name allow
 * @param rules {array} - Array of Cypher operators to be allowed in app
 */
Meteor.neo4j.set.allow(rules /* array of strings */);

/* @namespace neo4j.set
 * @name deny
 * @param rules {array} - Array of Cypher operators to be forbidden in app
 */
Meteor.neo4j.set.deny(rules /* array of strings */);


/*
 * @function
 * @namespace neo4j
 * @name query
 * @param query {string}      - Cypher query
 * @param opts {object}       - A map of parameters for the Cypher query
 * @param callback {function} - Callback function(error, data){...}. Where is data is [REACTIVE DATA SOURCE]
 *                              So to get data for query like:
 *                              'MATCH (a:User) RETURN a', you will need to: 
 *                              data.a
 * @param settings {object}   - {returnCursor: boolean} if set to true, returns Mongo\Cursor 
 * @description Isomorphic Cypher query call
 * @returns Mongo\Cursor or ReactiveVar [REACTIVE DATA SOURCE] 
 *
 * @note Please keep in mind what on client it returns ReactiveVar, but on server it returns just data, see difference in usage at example below
 *
 */
allUsers = Meteor.neo4j.query('MATCH (n:User) RETURN n');
var users = allUsers.get();

/*
 * Server only
 * @name methods
 * @param methods {object} - Object of methods, like: { methodName: function(){ return 'MATCH (a:User {name: {userName}}) RETURN a' } }
 * @description Create server methods to send query to neo4j database
 */
Meteor.neo4j.methods({
   'GetAllUsers': function(){
      return 'MATCH (n:User) RETURN n';
   }
});


/*
 * Client only
 * @name call
 * @description Call for server method registered via neo4j.methods() method, 
 *              returns error, data via callback.
 */
Meteor.neo4j.call('GetAllUsers', null, function(error, data){
   Session.set('AllUsers', data);
});
```

###### var N4JDB;
```javascript
/* 
 * Server only
 * @description Current GraphDatabase connection object, basically created from 'new Neo4j()''
 */
Meteor.N4JDB;

/* You may run queries with no returns on server with N4JDB: */
Meteor.N4JDB.query('CREATE (a:User {_id: ”123”})');
```

Okay, we’ve understood most used functions at `ostrio:neo4jdriver` and `ostrio:neo4jreactivity` packages, next we will understand what we need to change, to move on Neo4j DB.


----


##### Writing the code

###### Understanding the “Leaderboard” sources

 - When server start, it’s creates Players (we'll handle it with `N4JDB.query` method)
 - In template’s helper `players` - all players ordered by score (we will handle it with isomorphic `neo4j.query`)
 - In template’s helper `selectedName` - player name comes via `findOne` method (we will handle `selectedName` and `selectedPlayer` via Sessions, which will be set on `click` inside `Template.player.events`)
 - Incrementation of score implemented via `update` method (for security - we will use `neo4j.methods` and `neo4j.call`)

Let’s put our hands on it

###### Moving to Neo4j:
At first let’s create neo4j config file `lib/neo4j.js`:
```javascript
/* Allow client query execution */
Meteor.neo4j.allowClientQuery = true;
/* Custom URL to Neo4j should be here */
Meteor.neo4j.connectionURL = 'http://...';
/* But deny all writing actions on client */
Meteor.neo4j.set.deny(neo4j.rules.write);
```

Let’s move to `leaderboard.js` file and get rid of this line:
```javascript
Players = new Mongo.Collection("players");
/* To get all players: */
Players = neo4j.query('MATCH (a:Player) RETURN a ORDER BY a.score DESC');
```
This is isomorphic code, so we have all data we need inside `Players` variable on both - server and client sides. From now for reactivity use `Players.get()` method, and for cursor you have `Players.cursor` property

To check qty of players we use:
```javascript
(!Players.get() || Players.get().a && Players.get().a.length === 0)

/* Instead of:
 * Players.find().count()
 */
```

To generate Players we do:
```javascript
Meteor.N4JDB.query('CREATE (a:Player {name: {userName}}, {score: {userScore}})', {
   _id: String.generate(), /* See ‘/lib/String.js’ file */
   userName: name, 
   userScore: Math.floor(Random.fraction() * 10) * 5
});
/* Instead of:
 * Players.insert({
 *   name: name,
 *   score: Math.floor(Random.fraction() * 10) * 5
 * });
 */
```

To increment ‘score’, we create server method:
```javascript
if (Meteor.isServer) {
 Meteor.neo4j.methods({
    'incrementScore': function(){
      return 'MATCH (a:Player {_id:{playerId}}) SET a.score = a.score + toInt({incrementBy})';
    }
  });
}
```

and replace helper:
```javascript
Meteor.neo4j.call('incrementScore', {playerId: Session.get('selectedPlayer'), incrementBy: 5});
/* Instead of:
 * Players.update(Session.get("selectedPlayer"), {$inc: {score: 5}});
 */
```

To return Players list on client side, with `get()` method:
```javascript
Template.leaderboard.helpers({
  players: function () {
    return Players.get();
  },
  ...
});
```


