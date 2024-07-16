Creating a checklist of buttons in Rasa, where users can select multiple options, involves capturing each button click as an individual event and updating the slot values accordingly. Here's how you can achieve this:

### 1. Define the Slots and Custom Action in your domain.yml

Define a list slot to keep track of the selected options:

```yaml
slots:
  selected_options:
    type: list
    initial_value: []

actions:
- action_add_option
- action_show_selected_options
```

### 2. Implement the Custom Actions in your actions.py

Implement a custom action to add an option to the list and another action to show the selected options:

```python
from rasa_sdk import Action
from rasa_sdk.events import SlotSet

class ActionAddOption(Action):
    def name(self):
        return "action_add_option"

    def run(self, dispatcher, tracker, domain):
        selected_options = tracker.get_slot("selected_options") or []
        option = tracker.get_latest_entity_values("option").lower()

        if option not in selected_options:
            selected_options.append(option)

        return [SlotSet("selected_options", selected_options)]

class ActionShowSelectedOptions(Action):
    def name(self):
        return "action_show_selected_options"

    def run(self, dispatcher, tracker, domain):
        selected_options = tracker.get_slot("selected_options")
        dispatcher.utter_message(text=f"Selected options: {', '.join(selected_options)}")

        return []
```

### 3. Add the Buttons to your domain.yml

In the responses section, add a response with buttons for each option:

```yaml
responses:
  utter_ask_options:
    - text: "Please choose your options:"
      buttons:
        - title: "Option 1"
          payload: '/select_option{"option": "option 1"}'
        - title: "Option 2"
          payload: '/select_option{"option": "option 2"}'
        - title: "Option 3"
          payload: '/select_option{"option": "option 3"}'
        - title: "Done"
          payload: '/show_selected_options'
```

### 4. Define the Intents and Entities in your nlu.yml

```yaml
nlu:
- intent: select_option
  examples: |
    - Option 1
    - Option 2
    - Option 3

entities:
- option

- intent: show_selected_options
  examples: |
    - Done
    - Show selected options
```

### 5. Define the Rules or Stories in your rules.yml or stories.yml

#### Using Rules

```yaml
rules:
- rule: Add option to the list
  steps:
  - intent: select_option
  - action: action_add_option

- rule: Show selected options
  steps:
  - intent: show_selected_options
  - action: action_show_selected_options
```

#### Using Stories

```yaml
stories:
- story: Add option to the list
  steps:
  - intent: select_option
  - action: action_add_option

- story: Show selected options
  steps:
  - intent: show_selected_options
  - action: action_show_selected_options
```

### 6. Trigger the Response with the Buttons

You can trigger the response with the checklist of buttons in your stories or rules where needed:

```yaml
rules:
- rule: Ask user to select options
  steps:
  - action: utter_ask_options
```

Now, users can select multiple options from the checklist, and the selected options will be stored in the `selected_options` slot. When the user clicks "Done," the selected options will be shown.
