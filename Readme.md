Certainly! Below is an example of how you can create a form to book a flight in Rasa. This includes updates to the `domain.yml`, `data/stories.yml`, `data/rules.yml`, and `actions.py` files.

### 1. `domain.yml`

```yaml
version: "3.0"

intents:
  - book_flight
  - inform

entities:
  - departure_city
  - destination_city
  - travel_date
  - number_of_passengers

slots:
  departure_city:
    type: text
  destination_city:
    type: text
  travel_date:
    type: text
  number_of_passengers:
    type: text

forms:
  flight_booking_form:
    required_slots:
      - departure_city
      - destination_city
      - travel_date
      - number_of_passengers

responses:
  utter_ask_departure_city:
    - text: "From which city are you departing?"
  utter_ask_destination_city:
    - text: "What is your destination city?"
  utter_ask_travel_date:
    - text: "On what date would you like to travel?"
  utter_ask_number_of_passengers:
    - text: "How many passengers are traveling?"
  utter_submit:
    - text: "Thank you! You are booking a flight from {departure_city} to {destination_city} on {travel_date} for {number_of_passengers} passengers."

actions:
  - action_submit
```

### 2. `data/stories.yml`

```yaml
version: "3.0"

stories:
- story: book a flight
  steps:
    - intent: book_flight
    - action: flight_booking_form
    - active_loop: flight_booking_form
    - slot_was_set:
        - requested_slot: departure_city
    - slot_was_set:
        - requested_slot: destination_city
    - slot_was_set:
        - requested_slot: travel_date
    - slot_was_set:
        - requested_slot: number_of_passengers
    - action: action_submit
```

### 3. `data/rules.yml`

```yaml
version: "3.0"

rules:
- rule: Activate flight booking form
  steps:
    - intent: book_flight
    - action: flight_booking_form
    - active_loop: flight_booking_form

- rule: Submit flight booking form
  condition:
    - active_loop: flight_booking_form
  steps:
    - action: flight_booking_form
    - active_loop: null
    - action: action_submit
```

### 4. `actions.py`

Create a custom action to handle the submission of the form:

```python
# actions.py

from typing import Any, Text, Dict, List

from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher
from rasa_sdk.events import SlotSet

class ActionSubmit(Action):

    def name(self) -> Text:
        return "action_submit"

    def run(self, dispatcher: CollectingDispatcher,
            tracker: Tracker,
            domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:

        departure_city = tracker.get_slot('departure_city')
        destination_city = tracker.get_slot('destination_city')
        travel_date = tracker.get_slot('travel_date')
        number_of_passengers = tracker.get_slot('number_of_passengers')

        dispatcher.utter_message(text=f"Thank you! You are booking a flight from {departure_city} to {destination_city} on {travel_date} for {number_of_passengers} passengers.")

        return []
```

### 5. Training and Running the Bot

After making these changes, train your model:

```bash
rasa train
```

Then, you can run your action server and your Rasa server:

```bash
rasa run actions
rasa run
```

This setup should allow your Rasa bot to handle a simple flight booking conversation using a form.
Yes, you are correct. We need to update the `nlu.yml` file to include the necessary intents and examples that the bot will recognize and use to trigger the flight booking form. Here is how you can update the `nlu.yml` file:

### 6. `data/nlu.yml`

```yaml
version: "3.0"

nlu:
- intent: book_flight
  examples: |
    - I want to book a flight
    - Book a flight for me
    - Can you help me book a flight?
    - I need to book a flight
    - Help me book a flight
    - I would like to book a flight

- intent: inform
  examples: |
    - I am departing from [New York](departure_city)
    - My destination is [London](destination_city)
    - I will travel on [July 10th](travel_date)
    - I need tickets for [3](number_of_passengers) passengers
    - Departure city is [Los Angeles](departure_city)
    - Destination city is [Paris](destination_city)
    - The travel date is [August 15th](travel_date)
    - [4](number_of_passengers) people are traveling
    - From [Chicago](departure_city) to [Tokyo](destination_city) on [September 5th](travel_date)
    - I need a flight for [2](number_of_passengers) people on [December 1st](travel_date)
```

### Recap of All Files

1. `domain.yml`
2. `data/stories.yml`
3. `data/rules.yml`
4. `data/nlu.yml`
5. `actions.py`

### Training and Running the Bot

After updating the `nlu.yml` file, you should retrain your model:

```bash
rasa train
```

Then, run your action server and your Rasa server:

```bash
rasa run actions
rasa run
```

Now your bot should be able to handle the flight booking conversation, including recognizing the necessary intents and entities.
Providing custom entity extraction logic in Rasa involves creating a custom component that processes user messages and extracts entities. This custom component can be added to the NLU pipeline. Here’s a step-by-step guide:

### 1. Create a Custom Component

First, create a Python file for your custom component. For example, let's name it `custom_entity_extractor.py`.

#### `custom_entity_extractor.py`

```python
from typing import Any, Dict, List, Text

from rasa.nlu.components import Component
from rasa.nlu.extractors.extractor import EntityExtractor
from rasa.nlu.training_data import Message, TrainingData
from rasa.engine.recipes.default_recipe import DefaultV1Recipe
from rasa.engine.graph import GraphComponent
from rasa.shared.nlu.constants import ENTITIES, TEXT

import re

@DefaultV1Recipe.register([GraphComponent], is_trainable=False)
class CustomEntityExtractor(EntityExtractor, Component):
    @classmethod
    def required_components(cls) -> List[Type[Component]]:
        return []

    @classmethod
    def create(
        cls, config: Dict[Text, Any], model_storage: ModelStorage, resource: Resource
    ) -> CustomEntityExtractor:
        return cls()

    def process(self, messages: List[Message], **kwargs: Any) -> None:
        for message in messages:
            self.extract_entities(message)

    def extract_entities(self, message: Message) -> None:
        text = message.get(TEXT)

        # Define patterns for entities
        amount_pattern = re.compile(r"\b\d+\b")
        userid_pattern = re.compile(r"\buser\d+\b|\bid\d+\b|\b[a-zA-Z_]+\d*\b")
        account_number_pattern = re.compile(r"\b\d{10}\b")

        entities = []

        # Extract amount
        for match in re.finditer(amount_pattern, text):
            entities.append({
                "start": match.start(),
                "end": match.end(),
                "value": match.group(),
                "entity": "amount",
                "confidence": 1.0
            })

        # Extract user ID
        for match in re.finditer(userid_pattern, text):
            entities.append({
                "start": match.start(),
                "end": match.end(),
                "value": match.group(),
                "entity": "userid",
                "confidence": 1.0
            })

        # Extract account number
        for match in re.finditer(account_number_pattern, text):
            entities.append({
                "start": match.start(),
                "end": match.end(),
                "value": match.group(),
                "entity": "account_number",
                "confidence": 1.0
            })

        message.set(ENTITIES, message.get(ENTITIES, []) + entities)
```

### 2. Add the Custom Component to Your NLU Pipeline

Modify your `config.yml` to include the custom entity extractor in the NLU pipeline.

#### `config.yml`

```yaml
language: en

pipeline:
  - name: WhitespaceTokenizer
  - name: RegexFeaturizer
  - name: LexicalSyntacticFeaturizer
  - name: CountVectorsFeaturizer
  - name: DIETClassifier
    epochs: 100
  - name: custom_entity_extractor.CustomEntityExtractor
  - name: EntitySynonymMapper
  - name: ResponseSelector
    epochs: 100
```

### 3. Update Your Training Data

Ensure your training data includes enough examples for the custom entity extractor to work effectively.

#### `data/nlu.yml`

```yaml
version: "3.0"

nlu:
- intent: provide_amount
  examples: |
    - The amount is 500
    - I need to transfer 2500
    - Transfer 1000 dollars
    - Send 300
    - 1500
    - 450

- intent: provide_userid
  examples: |
    - My user ID is user123
    - The user ID is abc789
    - user456
    - id1234
    - User ID: john_doe

- intent: provide_account_number
  examples: |
    - The account number is 1234567890
    - Account number: 9876543210
    - 1122334455
    - My account number is 1234509876
    - 6789012345
```

### 4. Train and Run Your Rasa Bot

1. **Train your model:**

```bash
rasa train
```

2. **Run your action server and Rasa server:**

```bash
rasa run actions
rasa run
```

### Example Interaction

1. **User:** I need to transfer 500.
2. **Bot:** What is your user ID?
3. **User:** user123.
4. **Bot:** What is the account number?
5. **User:** 1234567890.
6. **Bot:** Thank you! You are transferring 500 from user ID user123 to account number 1234567890.

With this setup, the custom entity extractor should accurately extract `amount`, `userid`, and `account_number` from the user’s input, reducing confusion and ensuring correct slot filling. Adjust the regex patterns and logic in `custom_entity_extractor.py` as needed based on the specific formats of your entities.
