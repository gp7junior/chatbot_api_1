# Chatbot Communication core API

This repository contains a Chatbot core api. This document, describes the process taken in order to create it.

I am using Node.JS at the server side, Recast.Ai as Natural Language Processing and probably web semantic in the future ...

Freely inspired by this beautiful document published at [link](https://recast.ai/blog/build-a-pokebot-with-recast-ai-and-nodejs/)

## Requisites:

* An account on Recast.AI.
* An up-to-date version (6.0 or higher) of Node.JS.
* A Unix-like command line prompt.


## Files description

* node_modules (folder): generated automatically by node.js;
* index.js : the server;
* package.json : some configurations used to start the server;
* pokedex.json : contais all the data necessary to make queries (it is our database)
* ngrok : mecessary application to make our server public

## Step 1:

For now, the whole step 1 should be done manually at Recast.Ai. :unamused:

* create a new bot at Recast.Ai;
* create Intents
  * write some Expressions on it
* create Entities
* create Gazzetes
* create the skills set, one for each intent.

Remember that skills are triggered if one of the intents are present. We have to set up all the relevant requirements, and the most important part: send the requests to our API (by setting up the webhook url paramenter).

We also should set up the fallback messages if some of those requirements are missing.

## Step 2: Create the API

This is the most exciting part ! :smiley:

```bashing
mkdir ~/project_name && cd ~/project_name
npm init # you can accept all the default settings
npm install --save express body-parser
npm touch index.js # if this does not work just use touch index.js
```
