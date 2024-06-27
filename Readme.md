Using Haystack for document search with Rasa can provide a robust solution for retrieving answers from documents. Haystack is a powerful open-source framework designed for building search systems using NLP techniques. Here’s how you can integrate Haystack with Rasa to achieve this:

### Step 1: Set Up Haystack
First, you need to install Haystack. You can do this via pip:
```bash
pip install farm-haystack
```

### Step 2: Prepare Your Document
Assume you have a text file (`document.txt`) that you want to search through. You will need to load this document into Haystack.

### Step 3: Configure Haystack
Set up a simple Haystack pipeline to load your document and enable searching.

1. **Create a Haystack script** (`haystack_setup.py`):

```python
from haystack.document_stores import InMemoryDocumentStore
from haystack.nodes import FARMReader, TransformersReader, BM25Retriever
from haystack.pipelines import ExtractiveQAPipeline
from haystack.utils import clean_wiki_text, convert_files_to_docs, fetch_archive_from_http

# Initialize document store
document_store = InMemoryDocumentStore()

# Fetch and index documents
doc_dir = "data"
s3_url = "https://path_to_your_document/document.txt"
fetch_archive_from_http(s3_url, output_dir=doc_dir)
docs = convert_files_to_docs(dir_path=doc_dir, clean_func=clean_wiki_text, split_paragraphs=True)
document_store.write_documents(docs)

# Initialize retriever and reader
retriever = BM25Retriever(document_store=document_store)
reader = FARMReader(model_name_or_path="deepset/roberta-base-squad2", use_gpu=False)

# Initialize pipeline
pipeline = ExtractiveQAPipeline(reader, retriever)

def search_document(query):
    prediction = pipeline.run(query=query, params={"Retriever": {"top_k": 10}, "Reader": {"top_k": 5}})
    return prediction['answers'][0].answer
```

### Step 4: Create a Custom Action in Rasa
Create a custom action in Rasa that will use the Haystack pipeline to search for answers.

1. **Create a custom action file** (`actions.py`):

```python
from haystack_setup import search_document
from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher

class ActionSearchDocument(Action):

    def name(self) -> str:
        return "action_search_document"

    def run(self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: dict) -> list:
        query = tracker.latest_message.get('text')
        result = search_document(query)
        dispatcher.utter_message(result)
        return []
```

2. **Update your domain file** (`domain.yml`):

```yaml
actions:
  - action_search_document

intents:
  - ask_question

responses:
  utter_ask_question:
    - text: "What would you like to know?"

```

3. **Update your stories** (`data/stories.yml`):

```yaml
stories:
  - story: user asks a question
    steps:
      - intent: ask_question
      - action: action_search_document
```

### Step 5: Train Your Rasa Model
Train your Rasa model with the command:
```bash
rasa train
```

### Step 6: Run Your Rasa Bot
Run your Rasa bot and the actions server to interact with it:
```bash
rasa run actions
rasa shell
```

With this setup, your Rasa chatbot will be able to handle user queries by using Haystack to search through the document and return relevant answers. This approach leverages Haystack's powerful search capabilities to provide accurate and relevant information from your documents.
