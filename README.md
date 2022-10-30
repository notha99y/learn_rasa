# What is RASA
RASA is a tool to help you build task oriented dialogue systems.

## Task oriented dialogue systems
When we say "task oriented" we mean that the user wants to accomplish something. 

When we say dialogue system, we're talking about automated systems in a two way conversation. 

# State machines vs neural methods
## NLU
NLU stands for natural language understanding. In the context of Rasa we're usually talking about the part of the system that accepts raw text that goes in and machine-readable information that goes out. That usually means that it's the part that accepts text and can turn it into intents and entities.

## Dialogue policy
When we talk about Dialogue Policies we're referring to the part of the system that predicts the next action to take. The next action isn't just determined based on the current intent, we typically need to know about the entire conversation so far.

# How to make sure your chatbot works and how to improve over times
Manually review and annotate conversations as you're building your assistant. 

We call this process of working "Conversation-Driven Development"

# Setup
```bash
pip install rasa
```

```bash
python -m rasa init
```

# Files needed
## Domain files
The `domain.yml` file is the most important file to your rasa project
### What's in the domain file
The domain file is contains everything your assistant knows:
- responses: things the assistant can say to users
- intents: categories of things users say
- slots: variables remembered over the course of a conversation
- entities: pices of information extracted from incoming text
- forms and actions: these add application logic and extend what your assistant can do

## Config files
`config.yml`
## Data folder

### NLU
`nlu.yml`
### Stories
`stories.yml`
### Rules
`rules.yml`

# Commands
To create a new rasa project
```bash
rasa init
```
To retrain rasa chatbot
```bash
rasa train
```

To run sanic server
```bash
rasa shell
rasa shell --debug # for logs
```

# Training
## Data
- the text datya used to pretrain any models for features you're using (.e.g language models, word embeddings, etc.)
- user-generated text
- patterns of conversations
### Examples
- customer support logs
- user conversations with your assistant (your best choice)