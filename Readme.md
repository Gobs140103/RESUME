If your JSON data format for training is structured as `[[ocr_text, {"entities": [[start1, end1, label1], [start2, end2, label2], ...]}], ...]`, where each sublist contains OCR text and corresponding entity annotations, you'll need to modify the script to handle this specific format. Here’s how you can adjust the `train_ner.py` script accordingly:

### Example Training Data Format

Assuming your `train_data.json` looks like this:

```json
[
    [
        "OCR text from image 1",
        {
            "entities": [
                [49, 129, "MERCHANT_NAME"],
                [143, 160, "ADDRESS"]
            ]
        }
    ],
    [
        "OCR text from image 2",
        {
            "entities": [
                [30, 68, "MERCHANT_NAME"],
                [72, 95, "ADDRESS"]
            ]
        }
    ]
]
```

### Updated `train_ner.py` Script

Here's how you can modify the script to train a spaCy NER model using this format:

```python
import spacy
import random
import json
from spacy.training import Example
from paddleocr import PaddleOCR

# Initialize PaddleOCR
ocr = PaddleOCR(use_angle_cls=True, lang='en')

def get_text_from_image(image_path):
    # Run OCR on the image
    result = ocr.ocr(image_path, cls=True)
    
    # Extract text from the OCR result
    text_lines = []
    for line in result:
        for res in line:
            text_lines.append(res[1][0])  # res[1][0] contains the recognized text

    # Join all text lines into a single string
    text = '\n'.join(text_lines)
    return text

# Load spaCy English model with blank pipeline
nlp = spacy.blank("en")

# Add NER component to the pipeline using the string name
ner = nlp.add_pipe("ner")

# Load your training data from JSON
with open("train_data.json", "r") as f:
    train_data = json.load(f)

# Define function to convert data to spaCy format
def convert_data_to_spacy(data):
    ner_data = []
    for entry in data:
        ocr_text = entry[0]
        entities = entry[1]['entities']
        entities_spacy = [(start, end, label) for start, end, label in entities]
        ner_data.append((ocr_text, {"entities": entities_spacy}))
    return ner_data

# Convert data to spaCy format
formatted_data = convert_data_to_spacy(train_data)

# Add labels to the NER component
for entry in train_data:
    for ent in entry[1]['entities']:
        ner.add_label(ent[2])

# Initialize the optimizer
optimizer = nlp.begin_training()

# Train the NER model
for itn in range(10):  # Example: Train for 10 iterations
    random.shuffle(formatted_data)
    losses = {}
    
    for text, annotations in formatted_data:
        doc = nlp.make_doc(text)
        example = Example.from_dict(doc, annotations)
        nlp.update([example], drop=0.5, sgd=optimizer, losses=losses)
    
    print(f"Iteration {itn+1}: Losses - {losses}")

# Save the trained model
nlp.to_disk("trained_ner_model")

# Example usage: Test the trained model
test_image_path = "path_to_test_receipt_image.jpg"
test_text = get_text_from_image(test_image_path)
doc = nlp(test_text)
print("Entities:", [(ent.text, ent.label_) for ent in doc.ents])
```

### Explanation

1. **Extract Text**:
   - The `get_text_from_image` function processes the image using PaddleOCR and extracts the text. The text lines are concatenated into a single string for NER processing.

2. **Training Data Format**:
   - The `train_data.json` file now contains entries structured with OCR text as the first element and entity annotations under `"entities"` as lists of `[start, end, label]`.

3. **Handling Annotations**:
   - The `convert_data_to_spacy` function converts your JSON data into a format suitable for spaCy training, extracting `start`, `end`, and `label` for each entity within the OCR text.

4. **Training Loop**:
   - The script iterates over each entry in `formatted_data`, creates spaCy `Example` objects, and updates the NER model with the annotations.

5. **Saving and Testing**:
   - After training, the model is saved to disk. The script includes an example of how to test the trained model with a new receipt image.

### Running the Project

1. **Annotate Receipts**:
   - Annotate a diverse set of receipts to cover different formats using tools like LabelImg.

2. **Convert Annotations**:
   - Use `utils.py` to convert XML annotations to JSON if needed.

3. **Train the Model**:
   - Update and run the `train_ner.py` script to train the model with the annotated data.

4. **Evaluate and Refine**:
   - Test the model with new receipt texts.
   - Continue refining the model by annotating more data and adjusting training parameters as needed.

This approach ensures that your NER model is trained with correct annotations from your OCR text data, using the specific format where entities are defined by their `start`, `end`, and `label`. Adjust paths and specific details according to your project requirements. If you have any further questions or encounter issues, feel free to ask!
            "ocr_text": ocr_text,
            "ocr_confidence": None,
            "width": None,
            "height": None,
            "avg_char_width": None,
            "avg_line_height": None,
            "source_locations": {}
        }]
    }
    
    with open(output_file, 'w') as f:
        json.dump(result_json, f, indent=4)

# Example usage
image_path = 'data/preprocessed_images/receipt.jpg'
ner_model_path = 'models/ner_model'
output_file = 'data/parsed_receipt.json'
parse_receipt(image_path, ner_model_path, output_file)
```
receipt-parser/
│
├── data/
│   ├── images/        # Receipt images for annotation
│   └── annotations/   # Annotations in JSON format
│
├── scripts/
│   ├── preprocess.py  # Preprocessing images
│   ├── ocr.py         # Running OCR using PaddleOCR
│   ├── train_ner.py   # Training NER model using spaCy
│   ├── parse_receipt.py # Combining OCR and NER results
│   └── utils.py       # Utility functions
│
├── models/
│   └── ner_model/     # Directory to save the trained NER model
│
└── requirements.txt   # Project dependencies


### Summary
This project structure and code snippets provide
