Understood! We'll create a simple form in HTML with checkboxes that sends the selected attributes to the Rasa bot via a custom action. This custom action will process the selected attributes and respond accordingly.

### Frontend: HTML and JavaScript

First, let's update your `index.html` and `style.css` to include the form.

**index.html:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chatbot Widget</title>
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Open+Sans&family=Raleway:500&family=Roboto:wght@300&family=Lato&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css" integrity="sha256-eZrrJcwDc/3uDhsdt61sL2oOBY362qM3lon1gyExkL0=" crossorigin="anonymous">
    <link rel="stylesheet" href="static/css/materialize.min.css">
    <link rel="stylesheet" href="static/css/style.css">
</head>
<body>
    <div class="container">
        <div id="modal1" class="modal">
            <canvas id="modal-chart"></canvas>
        </div>
        <div class="widget">
            <div class="chat_header">
                <span class="chat_header_title">Sara</span>
                <span class="dropdown-trigger" href="#" data-target="dropdown1">
                    <i class="material-icons">more_vert</i>
                </span>
                <ul id="dropdown1" class="dropdown-content">
                    <li><a href="#" id="clear">Clear</a></li>
                    <li><a href="#" id="restart">Restart</a></li>
                    <li><a href="#" id="close">Close</a></li>
                </ul>
            </div>
            <div class="chats" id="chats">
                <div class="clearfix"></div>
            </div>
            <div class="keypad">
                <textarea id="userInput" placeholder="Type a message..." class="usrInput"></textarea>
                <div id="sendButton" role="button" aria-label="Send message">
                    <i class="fa fa-paper-plane" aria-hidden="true"></i>
                </div>
            </div>
        </div>
        <div class="profile_div" id="profile_div">
            <img class="imgProfile" src="static/img/botAvatar.png" alt="Bot Avatar">
        </div>
    </div>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script src="static/js/lib/materialize.min.js"></script>
    <script src="static/js/lib/uuid.min.js"></script>
    <script src="static/js/script.js"></script>
    <script src="static/js/lib/chart.min.js"></script>
    <script src="static/js/lib/showdown.min.js"></script>
    <script src="static/js/custom-query.js"></script>
</body>
</html>
```

**style.css:**
```css
.checkbox-options {
  padding: 10px;
  background: #ffffff;
  box-shadow: 2px 5px 5px 1px #dbdade;
  border-radius: 10px;
  margin: 10px;
}

.checkbox-options h3 {
  margin-bottom: 10px;
  color: #2c3e50;
  font-size: 18px;
}

.checkbox-options label {
  display: block;
  margin-bottom: 5px;
  font-size: 14px;
  color: #2c3e50;
}

.checkbox-options input[type="checkbox"] {
  margin-right: 10px;
}

.checkbox-options button {
  background: #2c3e50;
  color: #ffffff;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
}

.checkbox-options button:hover {
  background: #1a252f;
}
```

**custom-query.js:**
```javascript
document.addEventListener('DOMContentLoaded', function () {
    const chatContainer = document.getElementById("chats");
    const sendButton = document.getElementById("sendButton");
    const userInput = document.getElementById("userInput");

    sendButton.addEventListener("click", function () {
        const message = userInput.value;
        userInput.value = '';
        if (message.toLowerCase() === "custom query") {
            displayCustomQueryOptions();
        } else {
            sendMessage(message);
        }
    });

    function displayCustomQueryOptions() {
        const optionsHTML = `
            <div class="checkbox-options">
                <h3>Select Attributes</h3>
                <label><input type="checkbox" value="Attribute1"> Attribute1</label>
                <label><input type="checkbox" value="Attribute2"> Attribute2</label>
                <label><input type="checkbox" value="Attribute3"> Attribute3</label>
                <button id="submitCustomQuery">Submit</button>
            </div>
        `;
        chatContainer.innerHTML += optionsHTML;
        chatContainer.scrollTop = chatContainer.scrollHeight;

        document.getElementById("submitCustomQuery").addEventListener("click", submitCustomQuery);
    }

    function submitCustomQuery() {
        const checkboxes = document.querySelectorAll('.checkbox-options input[type="checkbox"]');
        const selectedAttributes = [];
        checkboxes.forEach(checkbox => {
            if (checkbox.checked) {
                selectedAttributes.push(checkbox.value);
            }
        });

        const message = selectedAttributes.length ? `Selected attributes: ${selectedAttributes.join(', ')}` : 'No attributes selected.';
        sendMessage(message);
    }

    function sendMessage(message) {
        if (message.trim() === "") {
            return;
        }

        // Display user message
        chatContainer.innerHTML += `<div class="userMsg">${message}</div>`;
        chatContainer.scrollTop = chatContainer.scrollHeight;

        // Send message to Rasa bot
        $.ajax({
            url: "http://localhost:5005/webhooks/rest/webhook",
            type: "POST",
            contentType: "application/json",
            data: JSON.stringify({
                sender: "user",
                message: message
            }),
            success: function (botResponse) {
                displayBotResponse(botResponse);
            },
            error: function () {
                chatContainer.innerHTML += `<div class="botMsg">Sorry, I am having trouble understanding you right now.</div>`;
                chatContainer.scrollTop = chatContainer.scrollHeight;
            }
        });
    }

    function displayBotResponse(response) {
        if (response.length < 1) {
            return;
        }

        response.forEach(function (res) {
            if (res.text) {
                chatContainer.innerHTML += `<div class="botMsg">${res.text}</div>`;
                chatContainer.scrollTop = chatContainer.scrollHeight;
            }
        });
    }
});
```

### Backend: Rasa Custom Action

Create a custom action in Rasa to handle the incoming message with selected attributes.

**actions.py:**
```python
from typing import Any, Text, Dict, List
from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher

class ActionHandleCustomQuery(Action):

    def name(self) -> Text:
        return "action_handle_custom_query"

    def run(self, dispatcher: CollectingDispatcher,
            tracker: Tracker,
            domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:

        # Extract selected attributes from the latest user message
        latest_message = tracker.latest_message['text']
        selected_attributes = latest_message.replace("Selected attributes: ", "").split(", ")

        # Example response based on selected attributes
        if selected_attributes:
            response = f"You have selected the following attributes: {', '.join(selected_attributes)}"
        else:
            response = "You did not select any attributes."

        dispatcher.utter_message(text=response)

        return []
```

**domain.yml:**
```yaml
intents:
  - custom_query

actions:
  - action_handle_custom_query

responses:
  utter_custom_query:
    - text: "Let's start a custom query. Please select the attributes you're interested in."

rules:
  - rule: Handle custom query
    steps:
      - intent: custom_query
      - action: action_handle_custom_query
```

**nlu.yml:**
```yaml
version: "2.0"
nlu:
- intent: custom_query
  examples: |
    - custom query
```

### How it Works

1. When the user types "custom query," the chatbot will display a set of checkboxes.
2. The user selects the desired attributes and submits them.
3. The JavaScript sends the selected attributes as a message to the Rasa backend.
4. The custom action `action_handle_custom_query` processes the message and responds based on the selected attributes.
