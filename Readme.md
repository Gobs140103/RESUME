
To select the highest transaction ID from your database and increment it by 1, you can modify the custom action to query the highest existing transaction ID, add 1 to it, and then set this new value as the transaction ID. Here’s how you can implement this:

### Custom Action to Generate Incremented Transaction ID

```python
import sqlite3
from typing import Any, Text, Dict, List
from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher
from rasa_sdk.events import SlotSet

class ActionGenerateTransactionID(Action):
    def name(self) -> Text:
        return "action_generate_transaction_id"

    def run(self, dispatcher: CollectingDispatcher,
            tracker: Tracker,
            domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:
        
        # Connect to your database
        conn = sqlite3.connect('path_to_your_database.db')
        cursor = conn.cursor()
        
        # Query the highest transaction ID
        cursor.execute("SELECT MAX(transaction_id) FROM transactions")
        max_id = cursor.fetchone()[0]
        
        # If no transaction ID exists, start with 10000
        if max_id is None:
            new_transaction_id = 10000
        else:
            new_transaction_id = int(max_id) + 1
        
        # Close the database connection
        conn.close()
        
        # Set the transaction_id slot
        return [SlotSet("transaction_id", str(new_transaction_id))]
```

### Steps to Follow

1. **Connect to the Database**: Connect to your SQL database using the appropriate library (e.g., `sqlite3` for SQLite).
2. **Query the Highest Transaction ID**: Use `SELECT MAX(transaction_id) FROM transactions` to get the highest existing transaction ID.
3. **Increment the Transaction ID**: If no transaction ID exists, start with 10000; otherwise, increment the highest transaction ID by 1.
4. **Store the New Transaction ID**: Set the new transaction ID in a slot or use it as needed.

### Using the Custom Action in Your Rasa Stories or Rules

In your `domain.yml`:

```yaml
slots:
  transaction_id:
    type: text

actions:
- action_generate_transaction_id
```

In your `stories.yml` or `rules.yml`:

```yaml
- story: generate transaction id
  steps:
  - intent: request_transaction_id
  - action: action_generate_transaction_id
  - action: utter_transaction_id

responses:
  utter_transaction_id:
  - text: "Your transaction ID is {transaction_id}"
```

### Ensure Database Table Exists

Make sure your database has the necessary table (`transactions`) and column (`transaction_id`) for this to work. Adjust the database connection and query code accordingly if you're using a different SQL database system (e.g., PostgreSQL, MySQL).

### Note

- This example uses SQLite for simplicity. Adjust the database connection and query parts if you're using a different SQL database.
- Ensure to handle database connections and queries securely and efficiently in your production environment.
