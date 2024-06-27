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

Certainly! In Rasa 3.1, the setup and configuration have evolved slightly compared to earlier versions. Here’s how you can integrate Haystack with Rasa 3.1 for document search:

### Step-by-Step Guide

#### Step 1: Project Structure and Setup
Ensure your Rasa project structure is set up correctly:

```
my_project/
│
├── actions/
│   └── actions.py        # Custom actions code
│
├── data/
│   ├── nlu/
│   │   └── nlu.yml       # NLU training data
│   ├── rules.yml         # Rules for conversation handling
│   └── stories.yml       # Conversation stories
│
├── models/               # Trained models will be saved here
│
├── haystack_setup.py     # Haystack setup and document loading
│
├── credentials.yml       # API keys and other credentials
├── domain.yml            # Domain definition
└── config.yml            # Rasa configuration file
```

#### Step 2: Install Dependencies
Make sure you have the necessary dependencies installed. You can install Rasa and Haystack dependencies using:

```bash
pip install rasa
pip install farm-haystack
```

#### Step 3: Set Up Haystack
Create `haystack_setup.py` to configure Haystack and load your document.

**haystack_setup.py**:
```python
from haystack.document_store import InMemoryDocumentStore
from haystack.preprocessor.utils import fetch_archive_from_http, convert_files_to_dicts

# Initialize document store
document_store = InMemoryDocumentStore()

# Load and index documents
doc_dir = "data"
doc_path = "data/document.txt"

# Convert the document into a dictionary
docs = convert_files_to_dicts(dir_path=doc_dir, clean_func=None, split_paragraphs=False)

# Write documents to the document store
document_store.write_documents(docs)

# Print the total number of documents indexed
print(f"Number of documents indexed: {document_store.get_document_count()}")
```

#### Step 4: Define a Custom Action in Rasa
Create `actions/actions.py` to define the custom action that utilizes Haystack for document search.

**actions/actions.py**:
```python
from haystack.reader import FARMReader
from haystack.retriever import BM25Retriever
from haystack.pipeline import ExtractiveQAPipeline
from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher

# Initialize retriever and reader
retriever = BM25Retriever(document_store=document_store)
reader = FARMReader(model_name_or_path="deepset/roberta-base-squad2", use_gpu=False)

# Initialize pipeline
pipeline = ExtractiveQAPipeline(reader, retriever)

class ActionSearchDocument(Action):

    def name(self) -> str:
        return "action_search_document"

    async def run(self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: dict) -> list:
        query = tracker.latest_message.get('text')
        prediction = await pipeline.run(query=query, params={"Retriever": {"top_k": 10}, "Reader": {"top_k": 5}})
        answer = prediction['answers'][0].answer if prediction['answers'] else "Sorry, I couldn't find any relevant information."
        await dispatcher.utter_message(text=answer)
        return []
```

#### Step 5: Define NLU Data
Update your `data/nlu/nlu.yml` file to include the `ask_question` intent with sample questions.

**data/nlu/nlu.yml**:
```yaml
version: "3.0"

nlu:
  - intent: ask_question
    examples: |
      - What is the capital of France?
      - Who is the President of the United States?
      - Tell me about the Great Wall of China.
      - How does photosynthesis work?
      - Explain quantum mechanics.
```

#### Step 6: Define the Domain
Update your `domain.yml` file to include the intent and action.

**domain.yml**:
```yaml
version: "3.0"

intents:
  - ask_question

responses:
  utter_ask_question:
    - text: "What would you like to know?"

actions:
  - action_search_document
```

#### Step 7: Train Your Rasa Model
Train your Rasa model using the command:

```bash
rasa train
```

#### Step 8: Run Your Rasa Bot
Run your Rasa bot and the actions server:

```bash
rasa run actions
```

In a separate terminal, start the Rasa shell to interact with your bot:

```bash
rasa shell
```

Now, your Rasa bot should be able to answer questions based on the content of the `document.txt` file using Haystack for document search. Adjust the configurations and scripts as per your specific use case and requirements.

This setup ensures compatibility with Rasa 3.1 and utilizes Haystack for efficient document retrieval and answering within your chatbot application.
from haystack import Finder
from haystack.document_store.faiss import FAISSDocumentStore
from haystack.retriever.dense import DensePassageRetriever
from haystack.reader.farm import FARMReader
from haystack.utils import print_answers

# Initialize document store
document_store = FAISSDocumentStore()

# Add documents to document store
document_store.add_eval_data(
    filename="path/to/your/document.pdf",
    doc_index="document",
    doc_source="local"
)

# Initialize retriever and reader
retriever = DensePassageRetriever(document_store=document_store)
reader = FARMReader(model_name_or_path="deepset/bert-base-cased-squad2")

# Initialize Finder
finder = Finder(reader, retriever)

# Define your question
question = "What is the capital of France?"

# Perform the search
prediction = finder.get_answers(question=question, top_k_retriever=10, top_k_reader=5)

# Print out answers
print_answers(prediction, details="minimal")
