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
