To create a date range picker in Rasa and handle the selected date range as JSONified start and end dates, you'll need to make a few modifications to the frontend and backend. Here's the complete guide:

### Frontend (HTML, CSS, and JavaScript)

1. **HTML**: Add two date pickers for the start and end dates and a button to submit the selected date range.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rasa Date Range Picker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 50px;
        }

        .date-picker-container {
            margin-bottom: 20px;
        }

        .date-picker {
            padding: 10px;
            font-size: 16px;
        }

        .submit-btn {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="date-picker-container">
        <label for="start-date-picker">Start date:</label>
        <input type="date" id="start-date-picker" class="date-picker">
    </div>
    <div class="date-picker-container">
        <label for="end-date-picker">End date:</label>
        <input type="date" id="end-date-picker" class="date-picker">
    </div>
    <button id="submit-btn" class="submit-btn">Submit</button>

    <script>
        document.getElementById('submit-btn').addEventListener('click', function() {
            const startDate = document.getElementById('start-date-picker').value;
            const endDate = document.getElementById('end-date-picker').value;
            if (startDate && endDate) {
                sendDateRangeToRasa(startDate, endDate);
            } else {
                alert('Please select both start and end dates.');
            }
        });

        function sendDateRangeToRasa(startDate, endDate) {
            fetch('http://localhost:5005/webhooks/rest/webhook', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    sender: 'user',
                    message: {
                        start_date: startDate,
                        end_date: endDate
                    }
                })
            })
            .then(response => response.json())
            .then(data => {
                console.log('Response from Rasa:', data);
                alert('Date range submitted successfully.');
            })
            .catch(error => {
                console.error('Error:', error);
                alert('Failed to submit date range.');
            });
        }
    </script>
</body>
</html>
```

### Rasa Backend

1. **Actions**: Define a custom action to handle the date range in Rasa.

Create a custom action in your Rasa actions file (`actions.py`):

```python
# actions.py
from typing import Dict, List, Any
from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher

class ActionReceiveDateRange(Action):

    def name(self) -> str:
        return "action_receive_date_range"

    def run(self, dispatcher: CollectingDispatcher,
            tracker: Tracker,
            domain: Dict[str, Any]) -> List[Dict[str, Any]]:
        
        latest_message = tracker.latest_message['text']
        if isinstance(latest_message, str):
            # If latest_message is a string, parse it as JSON
            import json
            date_range = json.loads(latest_message)
        else:
            date_range = latest_message

        start_date = date_range.get('start_date')
        end_date = date_range.get('end_date')

        dispatcher.utter_message(text=f"Received date range: {start_date} to {end_date}")
        return []
```

2. **Domain**: Update your `domain.yml` file to include the custom action and the required intents.

```yaml
# domain.yml
intents:
  - inform_date_range

actions:
  - action_receive_date_range
```

3. **NLU**: Update your `nlu.yml` file to include training examples for the `inform_date_range` intent.

```yaml
# nlu.yml
version: "3.0"
nlu:
- intent: inform_date_range
  examples: |
    - {"start_date": "2023-07-01", "end_date": "2023-07-15"}
    - {"start_date": "2023-08-01", "end_date": "2023-08-31"}
```

4. **Stories/Rules**: Update your `stories.yml` or `rules.yml` to include the interaction.

```yaml
# stories.yml
stories:
- story: receive date range
  steps:
  - intent: inform_date_range
  - action: action_receive_date_range
```

Now, when a user selects a start and end date and submits them, the frontend will send the date range to the Rasa backend, where it will be processed by the custom action.

To obtain dates in the format `26-JUL-11` from the date picker, you'll need to format the selected dates in JavaScript before sending them to the Rasa backend. Here's how you can modify the JavaScript to achieve this:

### Frontend (HTML, CSS, and JavaScript)

1. **HTML**: No changes needed from the previous setup.

2. **JavaScript**: Modify the `sendDateRangeToRasa` function to format the dates correctly before sending them.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rasa Date Range Picker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 50px;
        }

        .date-picker-container {
            margin-bottom: 20px;
        }

        .date-picker {
            padding: 10px;
            font-size: 16px;
        }

        .submit-btn {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="date-picker-container">
        <label for="start-date-picker">Start date:</label>
        <input type="date" id="start-date-picker" class="date-picker">
    </div>
    <div class="date-picker-container">
        <label for="end-date-picker">End date:</label>
        <input type="date" id="end-date-picker" class="date-picker">
    </div>
    <button id="submit-btn" class="submit-btn">Submit</button>

    <script>
        const monthNames = ["JAN", "FEB", "MAR", "APR", "MAY", "JUN", "JUL", "AUG", "SEP", "OCT", "NOV", "DEC"];

        document.getElementById('submit-btn').addEventListener('click', function() {
            const startDate = document.getElementById('start-date-picker').value;
            const endDate = document.getElementById('end-date-picker').value;
            if (startDate && endDate) {
                const formattedStartDate = formatDate(startDate);
                const formattedEndDate = formatDate(endDate);
                sendDateRangeToRasa(formattedStartDate, formattedEndDate);
            } else {
                alert('Please select both start and end dates.');
            }
        });

        function formatDate(dateStr) {
            const date = new Date(dateStr);
            const day = String(date.getDate()).padStart(2, '0');
            const month = monthNames[date.getMonth()];
            const year = String(date.getFullYear()).slice(-2);
            return `${day}-${month}-${year}`;
        }

        function sendDateRangeToRasa(startDate, endDate) {
            fetch('http://localhost:5005/webhooks/rest/webhook', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    sender: 'user',
                    message: {
                        start_date: startDate,
                        end_date: endDate
                    }
                })
            })
            .then(response => response.json())
            .then(data => {
                console.log('Response from Rasa:', data);
                alert('Date range submitted successfully.');
            })
            .catch(error => {
                console.error('Error:', error);
                alert('Failed to submit date range.');
            });
        }
    </script>
</body>
</html>
```

### Rasa Backend

No changes are needed in the Rasa backend since it will receive the dates in the desired format.

### Summary

In the provided JavaScript, the `formatDate` function converts the date from the picker (`YYYY-MM-DD`) into the format `DD-MMM-YY` where `MMM` is the three-letter month abbreviation. This formatted date is then sent to the Rasa backend.
