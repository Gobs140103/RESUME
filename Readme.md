If you prefer not to use React for the frontend, you can still achieve this functionality using plain HTML, CSS, and JavaScript. Here's how you can create a custom query interface where the user selects attributes through buttons or a checklist, and then sends the data to Rasa.

### 1. Define Intents and Entities in Rasa

First, define the intents and entities in your `domain.yml` and `nlu.yml` files as mentioned before.

### 2. Custom Action to Handle Query

Ensure you have the custom action in your `actions.py` as described previously.

### 3. HTML and JavaScript for the Frontend

You can create an HTML page with checkboxes for selecting attributes and a button to submit the selection. Use JavaScript to handle the selection and send the data to Rasa.

#### index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Custom Query</title>
</head>
<body>
    <h2>Select Attributes for Custom Query</h2>
    <div id="attributes">
        <div>
            <input type="checkbox" id="attribute1" name="attribute1" value="attribute1">
            <label for="attribute1">Attribute 1</label>
        </div>
        <div>
            <input type="checkbox" id="attribute2" name="attribute2" value="attribute2">
            <label for="attribute2">Attribute 2</label>
        </div>
        <div>
            <input type="checkbox" id="attribute3" name="attribute3" value="attribute3">
            <label for="attribute3">Attribute 3</label>
        </div>
    </div>
    <button onclick="submitQuery()">Submit</button>

    <script>
        function submitQuery() {
            const selectedAttributes = [];
            const checkboxes = document.querySelectorAll('input[type="checkbox"]:checked');
            checkboxes.forEach((checkbox) => {
                selectedAttributes.push(checkbox.value);
            });

            const payload = {
                intent: "ask_custom_query",
                entities: selectedAttributes.map(attr => ({ entity: "attribute", value: attr }))
            };

            fetch("http://localhost:5005/webhooks/rest/webhook", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify({ sender: "user", message: payload })
            })
            .then(response => response.json())
            .then(data => {
                console.log(data);
            })
            .catch(error => {
                console.error("Error:", error);
            });
        }
    </script>
</body>
</html>
```

### 4. Update Rasa Stories and Rules

Update your `stories.yml` or `rules.yml` to include the new intent and custom action as previously described.

### 5. Run Your Rasa Server

Ensure your Rasa server is running:
```bash
rasa run
```

And run your custom action server:
```bash
rasa run actions
```

### 6. Serve Your HTML Page

Open the `index.html` file in a web browser. You can do this by simply double-clicking the file or using a local web server.

### 7. Test the Functionality

- Open the HTML page in your browser.
- Select the desired attributes.
- Click the "Submit" button.
- Check the console output or Rasa server logs to see the custom query being generated and processed.

With these steps, you can create a simple HTML and JavaScript-based interface for selecting attributes and sending a custom query to your Rasa server.
Creating a custom query in Rasa where the user decides the attributes through the frontend (using buttons and checklists) involves several steps. Here's a guide to help you implement this:

### 1. Define Intents and Entities in Rasa

First, define the intents and entities in your `domain.yml` and `nlu.yml` files.

#### domain.yml
```yaml
intents:
  - ask_custom_query

entities:
  - attribute

slots:
  attribute:
    type: list
    influence_conversation: false

responses:
  utter_ask_custom_query:
    - text: "Please select the attributes you want to include in your query."

actions:
  - action_custom_query
```

#### nlu.yml
```yaml
nlu:
  - intent: ask_custom_query
    examples: |
      - I want to make a custom query
      - Custom query
      - Query with specific attributes
```

### 2. Custom Action to Handle Query

Create a custom action that will process the user's selected attributes. This custom action will generate the query based on the selected attributes.

#### actions.py
```python
from typing import Any, Text, Dict, List
from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher

class ActionCustomQuery(Action):

    def name(self) -> Text:
        return "action_custom_query"

    def run(self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:
        selected_attributes = tracker.get_slot('attribute')
        if not selected_attributes:
            dispatcher.utter_message(text="No attributes selected.")
            return []

        # Example: Construct the query
        query = f"SELECT {', '.join(selected_attributes)} FROM your_table"
        dispatcher.utter_message(text=f"Your custom query: {query}")

        # Add your database query execution logic here if needed

        return []
```

### 3. Frontend (React) Integration

In your React frontend, you can create a component with buttons or checklists for the user to select the attributes. When the user makes their selection, send this information to Rasa.

#### Example React Component
```jsx
import React, { useState } from 'react';

const CustomQuery = ({ sendMessage }) => {
  const [selectedAttributes, setSelectedAttributes] = useState([]);

  const attributes = ["attribute1", "attribute2", "attribute3"]; // Replace with your actual attributes

  const handleSelection = (attribute) => {
    setSelectedAttributes((prev) =>
      prev.includes(attribute) ? prev.filter((attr) => attr !== attribute) : [...prev, attribute]
    );
  };

  const handleSubmit = () => {
    const message = {
      intent: { name: "ask_custom_query" },
      entities: selectedAttributes.map((attr) => ({ entity: "attribute", value: attr })),
    };
    sendMessage(message);
  };

  return (
    <div>
      <h2>Select Attributes for Custom Query</h2>
      {attributes.map((attribute) => (
        <div key={attribute}>
          <input
            type="checkbox"
            id={attribute}
            name={attribute}
            value={attribute}
            onChange={() => handleSelection(attribute)}
          />
          <label htmlFor={attribute}>{attribute}</label>
        </div>
      ))}
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
};

export default CustomQuery;
```

### 4. Update Rasa Stories and Rules

Update your `stories.yml` or `rules.yml` to include the new intent and custom action.

#### stories.yml
```yaml
stories:
  - story: custom query story
    steps:
      - intent: ask_custom_query
      - action: action_custom_query
```

### 5. Test Your Implementation

- Run your Rasa server with `rasa run`.
- Run your custom action server with `rasa run actions`.
- Integrate your React frontend with the Rasa server.
- Test the entire flow by selecting attributes and submitting the query through the React component.

With these steps, you should be able to create a custom query in Rasa where the user decides the attributes through the frontend using buttons and checklists.
