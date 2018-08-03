# Example of Chatbot Communication core API: pokedex-bot

This repository contains a example of a Chatbot core api. This document, describes the process taken in order to create it.

I am using Node.JS at the server side, Recast.Ai as Natural Language Processing and probably web semantic in the future ...

Freely inspired by this beautiful document published at Recast.Ai tutorials:  [link](https://recast.ai/blog/build-a-pokebot-with-recast-ai-and-nodejs/)

This tutorial guides you throw the creation of a simple chatbot that can performs two skills: (i) basic information about pokemons, and (ii) evolution information also about those little monsters.

## Requisites:

* An account on Recast.AI.
* An up-to-date version (6.0 or higher) of Node.JS.
* A Unix-like command line prompt.


## Files description

* node_modules (folder): generated automatically by Node.JS;
* index.js : the server;
* package.json : some configurations used to start the server;
* pokedex.json : contains all the data necessary to make queries (it is our database)
* ngrok : necessary application to make our server public. You should download it from the official website.  

## Step 1:

For now, the whole step 1 should be done manually at Recast.Ai. :unamused:

* create a new bot at Recast.Ai;
* create Intents
  * write some Expressions on it
* create Entities
* create Gazzetes
* create the skills set, one for each intent.

Remember that skills are triggered if one of the intents are present. We have to set up all the relevant requirements, and the most important part: send the requests to our API (by setting up the webhook url parameter).

We also should set up the fallback messages if some of those requirements are missing.

## Step 2: Create the API

This is the most exciting part ! :smiley:

### Creating the workspace and the dependencies

```bashing
mkdir ~/project_name && cd ~/project_name
npm init # you can accept all the default settings
npm install --save express body-parser
npm touch index.js # if this does not work just use touch index.js
```

### Creating the routes

Inside our index.js file we should define our routes, i.e: the actions that our application should take when some skills are triggered.

```javascript

// loading modules and setting up the server
const express = require('express');
const bodyParser = require('body-parser');
const db = require('./pokedex.json');

const app = express();
app.use(bodyParser.json());

// Load routes
// it should be a route for each skill and also one when some errors occurs.
app.post('/pokemon-informations', getPokemonInformations);
app.post('/pokemon-evolutions', getPokemonEvolutions);
app.post('/errors', function (req, res) {
  console.error(req.body);
  res.sendStatus(200);
});

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`App is listening on port ${PORT}`));
```
These routes are going to retrieve information about pokemons from the json file (pokedex.json). In order to do this we are going to need a function to take that data:

```javascript
function findPokemonByName(name) {
  const data = db.find(p => p.name.toLowerCase() === name.toLowerCase());
  if (!data) {
    return null;
  }
  return data;
};
```

Probably in the future, this function (findPokemonByName) will make Sparql queries in my ontology.

### Creating server functions:

Following the fallback function:

```javascript
function getPokemonInformations(req, res) {
  const pokemon = req.body.conversation.memory.pokemon;
  const pokemonInfos = findPokemonByName(pokemon.value);

  if (!pokemonInfos) {
    res.json({
      replies: [
        { type: 'text', content: `I don't know a Pokémon called ${pokemon} :(` },
      ],
    });
  } else {
    res.json({
      replies: [
        { type: 'text', content: `🔎${pokemonInfos.name} infos` },
        { type: 'text', content: `Type(s): ${pokemonInfos.types.join(' and ')}` },
        { type: 'text', content: pokemonInfos.description },
        { type: 'picture', content: pokemonInfos.image },
      ],
    });
  }
}
```

We also should have a function to control the /pokemon-evolutions route:

```javascript
function getPokemonEvolutions(req, res) {
  const pokemon = req.body.conversation.memory.pokemon;
  const pokemonInfos = findPokemonByName(pokemon.value);

  if (!pokemonInfos) {
    res.json({
      replies: [
        { type: 'text', content: `I don't know a Pokémon called ${pokemon} :(` },
      ],
    });
  } else if (pokemonInfos.evolutions.length === 1) {
    res.json({
      replies: [{ type: 'text', content: `${pokemonInfos.name} has no evolutions.` }],
    });
  } else {
    res.json({
      replies: [
        { type: 'text', content: `🔎${pokemonInfos.name} family` },
        {
          type: 'text',
          content: pokemonInfos.evolutions.map(formatEvolutionString).join('\n'),
        },
        {
          type: 'card',
          content: {
            title: 'See more about them',
            buttons: pokemonInfos.evolutions
              .filter(p => p.id !== pokemonInfos.id) // Remove initial pokemon from list
              .map(p => ({
                type: 'postback',
                title: p.name,
                value: `Tell me more about ${p.name}`,
              })),
          },
        },
      ],
    });
  }
}

```

In addition to this, we are going to need a function to format the output of evolution's data.

```javascript
function formatEvolutionString(evolution) {
  let base = `🔸 ${evolution.name}`;
  if (evolution.trigger === 'leveling') {
    base += ` -> lvl ${evolution.trigger_lvl}`;
  }
  if (evolution.trigger === 'item') {
    base += ` -> ${evolution.trigger_item}`;
  }
  return base;
}
```

## Step 3: Launching the API

To launch the app we just need to run two commands in the terminal:
```bashing
npm start # in your server folder
```
The app will start in the default port 5000.
If everything works well you should see this output:

```bashing
App is listening on port 5000
```

Now is time to make this port public:

```bashing
./ngrok http 5000
```
To finish we need to paste the https given by ngrok at our Recast.Ai bot webhook base URL.

And that's it!
