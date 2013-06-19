jaab - just another awesome boilerplate
========

Description
-----

Based on <a href="https://github.com/visionmedia/" target="_blank">Aaron Heckmann</a>'s <a href="http://vimeo.com/56166857" target="_blank">Modular web applications with Node.js and Express</a>, this is my take on the the file structure and MVC Model at the start of an application.<br>
It works for me, in the way that it let's me build fast and extremely modular. I changed the structure a little bit and have a few personal conventions that you may find to be as easy to work with as I do.<br>
This boilerplate includes everything you need to start an Express Application, except, yet, database connection and user authentication.
<h3>it's built on:</h3>
<ul>
    <li>Express as the main web framework</li>
    <li>Swig and Consolidate as the templating solution</li>
    <li>Twitter Bootstrapp as an HTML+CSS Framework</li>
    <li>jQuery is included but not used</li>
    <li>Buster.js for Unit Testing (open to to other suggestions)</li>
    <li>never forget the modules these are built on</li>
</ul>

<h3>you will like it if:</h3>
<ul>
    <li>you're an experienced node developer who wants to try new approaches</li>
    <li>you are not so experienced and still looking for the best way to structure your apps</li>
    <li>you're a complete noob and wish to learn how things work while being able to develop your first application</li>
</ul>

<h3>you should use it because:</h3>
<ul>
    <li>It's very simple, well documented</li>
    <li>it runs and it passes the tests</li>
    <li>and it actually makes sense</li>
</ul>

Usage
-----

For now, you just have to clone the repository, init your own and edit package.json for app info. Run express install and you're done.


    var originalString = 'André Alçada Padez';
    $ npm install jaab
    var normalizedString = normalizer.normalize(originalString);
    //normalizedString => 'andre alcada padez'

As always you have to require it:

    var normalizer = require('normalizer');

You can use it directly to normalize a string:

    var originalString = 'André Alçada Padez';
    var normalizedString = normalizer.normalize(originalString);
    //normalizedString => 'andre alcada padez'
    
Here is an example insert:

    var doc = req.body;
    doc = normalizer.normalizeSearchFields(doc, Person);
    var person = new Person(doc);
    person.save();

Using with Mongo:

    //you just have to pass an object, containing the fields to be normalized:
    var schema = {
        normalized: {
            name: String,
            ....
        }
    };
    var doc = req.body;
    doc = normalizer.normalizeSearchFields(doc, schema);
    var person = new Person(doc);
    person.save();
    
and for filtering and sorting:
    
    var filter = {nome: 'André'};
    normalizer.treatFilter(filter);
    var sort = {nome: 1};
    normalizer.treatSort(sort);
    Person.find(filter).sort(sort).exec(function(err, people){....});
    //you can expect the correct results here


Detailed Instructions
-----
<br>
<b>Installation</b>
    
    npm install normalizer
and, wherever you need it
    
    var normalizer = require('normalizer');

<br>
<b>Implementation</b>

First, you need to "pimp" your Model's Schema, create a top-level property called normalized, containing the properties you want to normalize:

and for filtering and sorting:
    
   
    var schema = {
        name: String,
        address: {
            city: String,
            county: String,
            localAddress: {
                streetName: String,
                building: String,
                apartment: String
            }
        },
        email: String,
        phone: String,
        normalized: {
            name: String, 
            address: {
                city: String,
                county: String,
                localAddress: {
                    streetName: String,
                }
            }
        }
    };
    
    var schema = new mongoose.Schema(exports.schema);
    var Person = mongoose.model('Person', schema);

Right before upserting the document to the DB, you need to run normalizeSearchFields(), for example:
    
    var doc = req.body;
    normalizer.normalizeSearchFields(doc, Person);
    doc.save(function(){...});


That's it!, the result will be a Person instance, with the added <code>normalized</code> field, containing the transformation you would expect from the original document:

    //this:
    {
        name: 'André Alçada',
        address: {
            city: 'Olhão',
            county: 'Algés',
            localAddress: {
                streetName: 'Calçada de São Julião',
                building: '13',
                apartment: 'A'
            }
        },
        email: 'andre.padez@gmail.com',
        phone: 912345678,
    };
    //becomes:
    var body = {
        name: 'André Alçada',
        address: {
            city: 'Olhão',
            county: 'Algés',
            localAddress: {
                streetName: 'Calçada de São Julião',
                building: '13',
                apartment: 'A'
            }
        },
        email: 'andre.padez@gmail.com',
        phone: 912345678,
        normalized: {
            name: 'andre alcada',
            address: {
                city: 'olhao',
                county: 'alges',
                localAddress: {
                    streetName: 'calcada de sao juliao' 
                }
            } 
        }
    };

Be careful for every time you run <code>normalizeSearchFields</code>, the normalized object is always rewritten so, you have to pass at least all the original fields that will be expected or you will lose the other ones. (side note: this is a question opened for discussion, if i get enough feedback, i will rewrite it to protect the fields).


API
-----
<br>
<code>normalizer.normalize()</code>

    exports.normalize(String string[, Boolean keepCase]){
        //if keepCase == false -> returned string will be lowercased 
        return String
    }
    
    var string = 'André Alçada Padez'; 
    var normString = normalizer.normalize(string);
    //normString = 'andre alcada padez';
    //or
    var normString = normalizer.normalize(string, true);
    //normString = 'Andre Alcada Padez';
<br>
<code>normalizer.normalizeSearchFields()</code>

    exports.normalizeSearchFields = function(Object doc, Mongoose.Model|Object model[, Boolean keepCase]){
        //does not return, only updates the doc object
    }
    
    var doc = req.body;
    normalizer.normalizeSearchFields(doc, Person);
    var person = new Person(doc);
    person.save();
    //see normalizer.normalize for keepCase
    
<br>
<code>normalizer.normalizeFilter()</code>

    exports.normalizeFilter = function(Object filter, Mongoose.Model|Object model[, Boolean wholeString][, Boolean keepCase){
        //returns normalized filter object
        //if wholeString == false -> a regExp is used ( /text/i ), ideal for fuzzy search or 'like' functionality 
    }
    
    var filter = {name: 'André', email: 'andre'};
    filter = normalize.normalizeFilter(filter, Person); filter => {'normalized.name': 'andre', email: 'andre'}
    //or, using keepCase:
    filter = normalize.normalizeFilter(filter, Person, true); filter => {'normalized.name': 'Andre', email: 'andre'}
<br>
<code>normalizer.normalizeSort()</code>

    exports.normalizeSort = function(Object sort, Mongoose.Model|Object model[, Boolean wholeString]){
        //returns normalized sort object
    }
    
    var sort = {name: 1, email: -1};
    sort = normalize.normalizeSort(sort); // sort => {'normalized.name': 1, email: -1}

Road Map
-----

<ul>
    <li>add more characters to the charMap - that will be up to you (: </li>
</ul>

Feel free to use, fork and please contribute reporting bugs and with pull requests
