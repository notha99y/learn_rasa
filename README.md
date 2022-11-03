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

### Piplines and Policies
NLU pipeline and dialogue policies are defined inside of your `config.yml` file

The NLU pipeline defines the steps user messages will be passed through until a decision on what user's message is about is made e.g.

text: Hello. I would like to check my account balance
1. tokenization
2. featurization
3. intent classification and entity extraction
4. intent

intent: check_balance

```yaml
language: en

pipeline:
# will be selected by the suggested config feature
# The order methods
- name: WhitespaceTokenizer
- name: RegexFeaturizer
- name: LexicalSyntacticFeaturizer
- name: CountVectorsFeaturizer
- name: CountVectorsFeaturizer
    analyzer: char_wb
    min_ngram: 1
    max_ngram: 4
- name: DIETClassifier
    epochs: 100

policies:
    - name: MemoizationPolicy
    - name: TEDPolicy
    max_history: 5
    epochs: 10
    - name: RulePolicy
```

#### Dialogue policies
Dialogue policies are techniques your assistants use to decide on the next best action.

They are different from NLU policies as NLU policies have to be ran sequentially, when they can be run in parrallel.

It is highly recommended that we used the default policy priority:
- RulePolicy: 6
- MemoizationPolicy or AugmentatedMemoizationPolicy: 3
- TEDPolicy: 1

There are two types of policies:
- Rule Policies
Assistant makes the decision on how to responsd based on rules defined inside of your `rules.yml` file
- Machine Learning policies
Assistant makes the decision on how to respond by learning from the data defined inside of the `stories.yml` file

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
### Responses
Respones are simple messages that your assistant can send back to your users

You can also specify your output channel: e.g. slack

Response Template
```yaml
entities:
    - name
slots:
    - name:
        type: any
responses:
    utter_greet:
        - text: "Hello! How are you?"
        - text: "Hello there!"
        - text: "Hello {name}! How are you today?"
    utter_goodbye:
        - text: "Bye Bye!"

```
### Basic Forms
```yaml
- rule: Activate Pizza Form
 steps:
    - intent: buy_pizza
    - action: simple_pizza_form
    - active_loop: simple_pizza_form
- rule: Submit Pizza Form
    condition:
        - active_loop: simple_pizza_form
    steps:
        - action: simple_pizza_form
        - active_loop: null
        - slot_was_set:
            - requested_slot: null
        - action: utter_submit
        - action: utter_pizza_slots
```



### Entities
example: "I would like to book a flight to Sydney" => entity: destination. value: Sydney

The training data  for entity extraction should be included in your `nlu.yml` file
```yaml
nlu:
- intent: inform
    examples: |
    - My account number is [123456789](account_number) # [value](entity_type)
```

There are three ways entities can be extracted in rasa:
1. using pre-built models:
    - duckling for extracting numbers, dates, url, email addresses
    - SpaCy for extracting names, product names, locations, etc.

2. using regex:
    - for entities that match a specific pattern (e.g. phone numbers, postcodes, etc.)

```yaml
nlu:
    - regex: account_number
    examples: |
    - \d{10,12}
    - intent: inform
    examples: |
    - My account number is [123456789](account_number)
```

The output of entity are in a `json` format

#### Synonyms:
- synonyms can be used to map extracted values to a single standardized values. e.g. Credit Card Account, Credit Account => Credit

Approach of adding synonyms

```yaml
nlu:
    - syonym: credit
        examples: |
        - credit card account
        - credit account
```
or by adding another value key

```yaml
nlu:
    - intent: check_balance
        examples: |
        - I would like to check my [credit card account]{"entity": "account", "value": "credit"}
        - How do I check the [credit account]{"entity": "account", "value": "credit"}
```

3. using machine learning
    - for extracting custom entities

#### Lookup tables
Lookup tables are lists of words used to generate case-sensitive regular expression patterns

#### Entity roles and groups
Entity roles allows you to define the roles of the entities of the same groups. e.g.

I am looking for a flight from New york to Boston. New york role is origin while Boston role is destination

### Slots
Slots are your assistant's memory.

#### Configuring slots
slots are defined inside of your `domain.yml`

```yaml
slots:
    destination:
        type: text
        influence_conversation: false
        mappings:
        - type: custom
```
Slots can also be used to influence the flow of the conversation when the slot is set. You have to include them in your training stories. 
```yaml
stories:
- story: booking a flight ticket
    steps:
    - intent: book_a_ticket
    - or:
        - slot_was_set:
            - destination: Toronto
        - slot_was_set:
            - destination: London
```

#### Slot mappings
Slot mapping allows you to define how each slot will be filled in. They are applied after each user message.
- from_entity
- from_text
- from_intent
- from_trigger_intent (from a form is activated by a user message with a specific intent)
- custom (using slot validation actions)

```yaml
slots:
    slot_name:
        type: any
        mappings:
            - type: from_intent
                value: my_value
                intent: intent_name
                not_intent: exclude_intent
```

#### Slot types
- text (the actual value of the text has no influence, just the presence)
- boolean 
- categorical
- float
- list (like text, value has no influence, just the presence)

## Config files
`config.yml`
## Data folder

### NLU
`nlu.yml` is the file to store your intent. It is a multi classification problem.

- if you have data use the modified content analysis:
    - go through data (or sample) by hand and assign each datapoint to a group
    - if no existing group fits, add a new one
    - at given intervals, go through your groups and combine or seperate them as needed
    - start with 2-3 passes through your dataset

- if you don't have data
    - start with the most common intent
        - most poeple want to the same thing
        - use the experts in your institution
    - start with the smallest possible number of intents (that cover your core use case)
        - don't fall to the long tail trap
    - everything else goes in an out of scope intent
        - if your assistant can't handle something, give users an esacpe hatch right away
    - additional intents will come from user data


### Stories
Stories are training data to teach your assistant what it should do next. `stories.yml`

- if you have conversation data. start with those patterns
- if not, you can generate your own conversation patterns:
    - it's easiest to use interactive learning to create stories
    - start with common flows, "happy paths"
    - then add common errors/ digressions

- Once your model is trained, add more data from user conversations

### Rules
Rules are a way to describe short pieces of conversations that always go the same where you dont really need any machine learning and it will always go the same way. `rules.yml`

For example: greet

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
rasa x # launch a web app
rasa run --enable-api --cors="*" --debug
```

# Training
## Data
- the text datya used to pretrain any models for features you're using (.e.g language models, word embeddings, etc.)
- user-generated text
- patterns of conversations
### Examples
- customer support logs
- user conversations with your assistant (your best choice)

## Dos and Donts
### Does
- use actual user conversations as stories
- have small stories that aren't full conversations
- use rules for one-off interactions (checking account balance, checking if this is a bot)

### Don'ts
- use rules for multi-turn interactions
- use OR statements and checkipointing often
- write out every possible conversation flow start to finish
- delay user testing!


# Deployment
## Integrating with slack
```bash
rasa run --connector slack --credentials credentials.yml --endpoints endpoints.yml --cors * --enable-api --debug
```

Useful links:
- https://medium.com/analytics-vidhya/integrating-your-rasa-chat-bot-with-slack-c18bffc6018b
- https://api.slack.com/apps
- https://rasa.com/docs/rasa/connectors/slack/