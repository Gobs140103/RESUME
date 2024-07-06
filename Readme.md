To create a receipt parser that outputs the detailed JSON structure you provided, we will follow these steps:

1. **OCR to extract text and layout information from the receipt image using PaddleOCR.**
2. **NER model to identify and extract relevant entities from the OCR text.**
3. **Combine OCR and NER results to structure the output JSON accordingly.**

Here’s a step-by-step guide, including the necessary code:

### Step 1: Set Up the Environment

1. **Install Required Libraries**:

   Create a `requirements.txt` file:
   ```plaintext
   paddlepaddle
   paddleocr
   spacy
   numpy
   pandas
   opencv-python
   ```
   Install the dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. **Download spaCy Base Model**:
   ```bash
   python -m spacy download en_core_web_sm
   ```

### Step 2: Annotation and Data Preprocessing

1. **Annotate Data Using LabelImg**:
   - Download and install LabelImg: [LabelImg GitHub](https://github.com/tzutalin/labelImg)
   - Annotate your receipt images by labeling different entities like `DATE`, `TOTAL`, `ITEM`, `PRICE`, etc.
   - Save the annotations in Pascal VOC format.

### Step 3: Convert Annotations to spaCy Format

Create `utils.py`:

```python
import os
import xml.etree.ElementTree as ET
import json

def convert_voc_to_spacy_format(annotations_dir, output_file):
    train_data = []

    for filename in os.listdir(annotations_dir):
        if not filename.endswith('.xml'):
            continue

        tree = ET.parse(os.path.join(annotations_dir, filename))
        root = tree.getroot()

        text = root.find('filename').text
        entities = []
        for obj in root.findall('object'):
            label = obj.find('name').text
            bbox = obj.find('bndbox')
            start_x = int(bbox.find('xmin').text)
            start_y = int(bbox.find('ymin').text)
            end_x = int(bbox.find('xmax').text)
            end_y = int(bbox.find('ymax').text)
            entities.append(((start_x, start_y), (end_x, end_y), label))

        train_data.append((text, {"entities": entities}))

    with open(output_file, 'w') as f:
        json.dump(train_data, f)

# Example usage
convert_voc_to_spacy_format('data/annotations', 'data/train_data.json')
```

### Step 4: Preprocess Images

Create `scripts/preprocess.py`:

```python
import cv2
import os

def preprocess_image(image_path):
    image = cv2.imread(image_path)
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    _, thresh = cv2.threshold(gray, 150, 255, cv2.THRESH_BINARY)
    return thresh

def preprocess_images(input_dir, output_dir):
    if not os.path.exists(output_dir):
        os.makedirs(output_dir)

    for filename in os.listdir(input_dir):
        if not filename.endswith(('.jpg', '.png')):
            continue
        image_path = os.path.join(input_dir, filename)
        preprocessed_image = preprocess_image(image_path)
        cv2.imwrite(os.path.join(output_dir, filename), preprocessed_image)

# Example usage
preprocess_images('data/images', 'data/preprocessed_images')
```

### Step 5: Perform OCR Using PaddleOCR

Create `scripts/ocr.py`:

```python
from paddleocr import PaddleOCR
import os
import json

def perform_ocr(image_path):
    ocr = PaddleOCR(use_angle_cls=True, lang='en')
    result = ocr.ocr(image_path, cls=True)
    texts = []
    for line in result[0]:
        texts.append(line[1][0])
    return result

def ocr_images(input_dir, output_dir):
    if not os.path.exists(output_dir):
        os.makedirs(output_dir)

    for filename in os.listdir(input_dir):
        if not filename.endswith(('.jpg', '.png')):
            continue
        image_path = os.path.join(input_dir, filename)
        ocr_result = perform_ocr(image_path)
        with open(os.path.join(output_dir, filename + '.json'), 'w') as f:
            json.dump(ocr_result, f)

# Example usage
ocr_images('data/preprocessed_images', 'data/ocr_results')
```

### Step 6: Train NER Model

Create `scripts/train_ner.py`:

```python
import spacy
import random
import json

def load_train_data(file_path):
    with open(file_path, 'r') as f:
        train_data = json.load(f)
    return train_data

def train_ner_model(train_data, output_dir):
    nlp = spacy.blank("en")
    if "ner" not in nlp.pipe_names:
        ner = nlp.create_pipe("ner")
        nlp.add_pipe(ner, last=True)
    else:
        ner = nlp.get_pipe("ner")

    for _, annotations in train_data:
        for ent in annotations.get("entities"):
            ner.add_label(ent[2])

    optimizer = nlp.begin_training()
    for itn in range(30):
        print(f"Iteration {itn}")
        random.shuffle(train_data)
        losses = {}
        for text, annotations in train_data:
            nlp.update([text], [annotations], drop=0.5, losses=losses)
        print(losses)

    if not os.path.exists(output_dir):
        os.makedirs(output_dir)
    nlp.to_disk(output_dir)

# Example usage
train_data = load_train_data('data/train_data.json')
train_ner_model(train_data, 'models/ner_model')
```

### Step 7: Parse Receipts Using OCR and NER

Create `scripts/parse_receipt.py`:

```python
from paddleocr import PaddleOCR
import spacy
import os
import json
from datetime import datetime

def perform_ocr(image_path):
    ocr = PaddleOCR(use_angle_cls=True, lang='en')
    result = ocr.ocr(image_path, cls=True)
    texts = []
    for line in result[0]:
        texts.append(line[1][0])
    return result, "\n".join(texts)

def parse_receipt(image_path, ner_model_path, output_file):
    ocr_result, ocr_text = perform_ocr(image_path)
    nlp = spacy.load(ner_model_path)
    doc = nlp(ocr_text)
    
    parsed_entities = {}
    items = []

    for ent in doc.ents:
        if ent.label_ in parsed_entities:
            parsed_entities[ent.label_].append(ent.text)
        else:
            parsed_entities[ent.label_] = [ent.text]
    
    for token in doc:
        if token.ent_type_ == "ITEM":
            items.append({"description": token.text, "amount": None, "flags": None, "qty": None, "remarks": None, "unitPrice": None})
    
    result_json = {
        "ocr_type": "receipts",
        "request_id": "P_0-0-0-0-0-0-0-1_kr3awxez_810",
        "ref_no": "AspDemo_1626256135747_535",
        "file_name": os.path.basename(image_path),
        "request_received_on": int(datetime.now().timestamp() * 1000),
        "success": True,
        "image_width": None,
        "image_height": None,
        "image_rotation": 0.0,
        "recognition_completed_on": int(datetime.now().timestamp() * 1000),
        "receipts": [{
            "merchant_name": parsed_entities.get("merchant_name", [None])[0],
            "merchant_address": parsed_entities.get("merchant_address", [None])[0],
            "merchant_phone": parsed_entities.get("merchant_phone", [None])[0],
            "merchant_website": None,
            "merchant_tax_reg_no": None,
            "merchant_company_reg_no": None,
            "region": None,
            "mall": None,
            "country": "US",
            "receipt_no": parsed_entities.get("receipt_no", [None])[0],
            "date": parsed_entities.get("date", [None])[0],
            "time": parsed_entities.get("time", [None])[0],
            "items": items,
            "currency": "USD",
            "total": parsed_entities.get("total", [None])[0],
            "subtotal": parsed_entities.get("subtotal", [None])[0],
            "tax": parsed_entities.get("tax", [None])[0],
            "service_charge": None,
            "tip": None,
            "payment_method": parsed_entities.get("payment_method", [None])[0],
            "payment_details": None,
            "credit_card_type": None,
            "credit_card_number": None,
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
