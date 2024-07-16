To handle the custom query functionality in your Rasa bot, you need to update your Rasa project to include an intent for custom queries, a custom action to process the selected attributes, and update the domain file to support these changes. Below are the steps to update your Rasa bot:

### 1. Define the Intent in `nlu.yml`
Add an intent for custom queries and the possible user message.

```yaml
version: "3.1"
nlu:
  - intent: custom_query
    examples: |
      - custom query
      - I want to make a custom query
      - Show me the custom query form
```

### 2. Define the Slots and Entities in `domain.yml`
Update your `domain.yml` to include slots and entities that will capture the selected attributes.

```yaml
version: "3.1"
intents:
  - custom_query

entities:
  - attribute

slots:
  attribute:
    type: list
    influence_conversation: false

responses:
  utter_ask_custom_query:
    - text: "Please select the attributes you want to query."

actions:
  - action_process_custom_query
```

### 3. Add a Form for Custom Queries in `domain.yml`
Define a form to handle the user input for custom queries.

```yaml
forms:
  custom_query_form:
    required_slots:
      attribute:
        - type: from_entity
          entity: attribute
          intent: custom_query
          not_intent: deny
```

### 4. Create the Custom Action in `actions.py`
Add a custom action to process the selected attributes and respond accordingly.

```python
from typing import Any, Text, Dict, List
from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher
from rasa_sdk.forms import FormValidationAction

class ActionProcessCustomQuery(Action):

    def name(self) -> Text:
        return "action_process_custom_query"

    def run(self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:
        attributes = tracker.get_slot("attribute")
        if not attributes:
            dispatcher.utter_message(text="No attributes selected.")
        else:
            query_message = f"Custom query for: {', '.join(attributes)}"
            dispatcher.utter_message(text=query_message)
        
        return []
```

### 5. Update `stories.yml`
Add a story to handle the flow of custom queries.

```yaml
version: "3.1"
stories:
  - story: custom query path
    steps:
    - intent: custom_query
    - action: action_process_custom_query
```

### 6. Update `endpoints.yml`
Ensure that the action server is correctly defined in `endpoints.yml`.

```yaml
action_endpoint:
  url: "http://localhost:5055/webhook"
```

### 7. Run the Rasa Action Server
Ensure your custom actions are running by starting the action server:

```bash
rasa run actions
```

### 8. Train the Rasa Model
Train your Rasa model to incorporate the new changes.

```bash
rasa train
```

### 9. Run the Rasa Bot
Start your Rasa bot server:

```bash
rasa run
```

### Testing the Bot
With these updates, your bot should now handle the custom query functionality, displaying the custom query form when the user types "custom query," processing the selected attributes, and responding appropriately.

### Handling Frontend Integration
Ensure your frontend integration correctly sends the selected attributes to the bot and displays the response:

- The JavaScript function `submitCustomQuery` already handles form submission.
- Ensure the bot processes the message and returns the appropriate response.

By following these steps, your Rasa bot should now support a custom query feature triggered by user input and handle the processing of selected attributes.
