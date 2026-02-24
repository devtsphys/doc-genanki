# genanki Python API — Complete Reference Card

> **genanki** lets you programmatically create Anki decks (`.apkg` files) in Python.
> Install: `pip install genanki`  |  GitHub: https://github.com/kerrickstaley/genanki

-----

## Table of Contents

1. [Quick Start](#quick-start)
1. [Core Classes](#core-classes)
1. [Method Reference Table](#method-reference-table)
1. [Model (Note Type)](#model-note-type)
1. [Note](#note)
1. [Deck](#deck)
1. [Package](#package)
1. [Built-in Model Presets](#built-in-model-presets)
1. [Card Templates & Styling](#card-templates--styling)
1. [Media Files](#media-files)
1. [GUIDs & ID Generation](#guids--id-generation)
1. [Advanced Techniques](#advanced-techniques)
1. [Common Patterns & Recipes](#common-patterns--recipes)
1. [Troubleshooting](#troubleshooting)

-----

## Quick Start

```python
import genanki

# 1. Define a Model (note type)
my_model = genanki.Model(
    1607392319,                      # unique model ID
    'Simple Model',
    fields=[
        {'name': 'Question'},
        {'name': 'Answer'},
    ],
    templates=[
        {
            'name': 'Card 1',
            'qfmt': '{{Question}}',
            'afmt': '{{FrontSide}}<hr id=answer>{{Answer}}',
        },
    ]
)

# 2. Create a Deck
my_deck = genanki.Deck(2059400110, 'Country Capitals')

# 3. Add Notes
my_deck.add_note(genanki.Note(
    model=my_model,
    fields=['Capital of France?', 'Paris']
))

# 4. Write to .apkg
genanki.Package(my_deck).write_to_file('output.apkg')
```

-----

## Core Classes

|Class            |Purpose                                  |
|-----------------|-----------------------------------------|
|`genanki.Model`  |Defines note type: fields, templates, CSS|
|`genanki.Note`   |A single flashcard (one or more cards)   |
|`genanki.Deck`   |Collection of notes; maps to an Anki deck|
|`genanki.Package`|Bundles decks + media into a `.apkg` file|

-----

## Method Reference Table

### `genanki.Model`

|Method / Attribute|Signature                                                       |Description                           |
|------------------|----------------------------------------------------------------|--------------------------------------|
|`__init__`        |`Model(model_id, name, fields, templates, css='', model_type=0)`|Create a note type                    |
|`.model_id`       |`int`                                                           |Unique integer ID for the model       |
|`.name`           |`str`                                                           |Display name in Anki                  |
|`.fields`         |`list[dict]`                                                    |List of `{'name': str}` dicts         |
|`.templates`      |`list[dict]`                                                    |List of template dicts (see below)    |
|`.css`            |`str`                                                           |CSS applied to all cards of this model|
|`.model_type`     |`int`                                                           |`0` = standard, `1` = cloze           |

### `genanki.Note`

|Method / Attribute|Signature                                                          |Description                                 |
|------------------|-------------------------------------------------------------------|--------------------------------------------|
|`__init__`        |`Note(model, fields, sort_field=None, tags=None, guid=None, due=0)`|Create a note                               |
|`.model`          |`Model`                                                            |The model this note uses                    |
|`.fields`         |`list[str]`                                                        |Field values (must match model field count) |
|`.sort_field`     |`int | None`                                                       |Index of field used for sorting (default: 0)|
|`.tags`           |`list[str]`                                                        |List of tag strings                         |
|`.guid`           |`str | int`                                                        |Globally unique identifier for the note     |
|`.due`            |`int`                                                              |Due order (for new cards)                   |
|`.cards`          |property                                                           |Returns list of `Card` objects generated    |

### `genanki.Deck`

|Method / Attribute|Signature                            |Description                       |
|------------------|-------------------------------------|----------------------------------|
|`__init__`        |`Deck(deck_id, name, description='')`|Create a deck                     |
|`.deck_id`        |`int`                                |Unique integer ID for the deck    |
|`.name`           |`str`                                |Deck name (use `::` for sub-decks)|
|`.description`    |`str`                                |Optional deck description         |
|`.notes`          |`list[Note]`                         |Notes added to this deck          |
|`add_note(note)`  |`Deck.add_note(Note) -> None`        |Add a Note to the deck            |

### `genanki.Package`

|Method / Attribute   |Signature                                 |Description                     |
|---------------------|------------------------------------------|--------------------------------|
|`__init__`           |`Package(deck_or_decks, media_files=None)`|Create a package                |
|`.decks`             |`list[Deck]`                              |Decks included in the package   |
|`.media_files`       |`list[str]`                               |Paths to media files to bundle  |
|`write_to_file(path)`|`Package.write_to_file(str) -> None`      |Write `.apkg` to given file path|

### Utility Functions

|Function                   |Signature               |Description                       |
|---------------------------|------------------------|----------------------------------|
|`genanki.guid_for(*values)`|`guid_for(*args) -> str`|Deterministic GUID from any values|

-----

## Model (Note Type)

### Fields Definition

```python
fields = [
    {'name': 'Front'},
    {'name': 'Back'},
    {'name': 'Extra'},   # optional extra fields
    {'name': 'Audio'},
]
```

Each field dict **must** have a `'name'` key. Additional optional keys:

|Key       |Type  |Description                 |
|----------|------|----------------------------|
|`'name'`  |`str` |Field label (required)      |
|`'ord'`   |`int` |Field order (auto-set)      |
|`'sticky'`|`bool`|Retain value in card editor |
|`'rtl'`   |`bool`|Right-to-left text direction|
|`'font'`  |`str` |Font family for the editor  |
|`'size'`  |`int` |Font size for the editor    |

### Templates Definition

```python
templates = [
    {
        'name': 'Card 1',           # template name (required)
        'qfmt': '{{Front}}',        # question side HTML (required)
        'afmt': '{{FrontSide}}<hr id=answer>{{Back}}',  # answer side HTML (required)
        'bqfmt': '',                # browser question format (optional)
        'bafmt': '',                # browser answer format (optional)
        'did': None,                # deck override (optional)
    },
]
```

### CSS Styling

```python
my_model = genanki.Model(
    ...,
    css="""
    .card {
        font-family: arial;
        font-size: 20px;
        text-align: center;
        color: black;
        background-color: white;
    }
    .hint { color: #aaa; font-size: 14px; }
    """
)
```

### Full Model Example

```python
VOCABULARY_MODEL = genanki.Model(
    1380120064,
    'Vocabulary Card',
    fields=[
        {'name': 'Word'},
        {'name': 'Definition'},
        {'name': 'Example Sentence'},
        {'name': 'Image'},
    ],
    templates=[
        {
            'name': 'Word to Definition',
            'qfmt': '<b>{{Word}}</b><br>{{Image}}',
            'afmt': '{{FrontSide}}<hr id=answer>{{Definition}}<br><i>{{Example Sentence}}</i>',
        },
        {
            'name': 'Definition to Word',
            'qfmt': '{{Definition}}',
            'afmt': '{{FrontSide}}<hr id=answer><b>{{Word}}</b>',
        },
    ],
    css='.card { font-family: Georgia; font-size: 18px; text-align: center; }'
)
```

-----

## Note

### Basic Note

```python
note = genanki.Note(
    model=my_model,
    fields=['What is 2+2?', '4'],
)
```

### Note with Tags

```python
note = genanki.Note(
    model=my_model,
    fields=['Bonjour', 'Hello'],
    tags=['french', 'greetings', 'A1'],
)
```

### Note with Custom GUID

```python
# Deterministic GUID — same values always produce same GUID
note = genanki.Note(
    model=my_model,
    fields=['Paris', 'France'],
    guid=genanki.guid_for('Paris', 'France'),
)

# Hard-coded integer GUID (must be globally unique)
note = genanki.Note(
    model=my_model,
    fields=['Berlin', 'Germany'],
    guid=1234567890,
)
```

### Note with Custom Sort Field

```python
# Sort by the 2nd field (index 1) instead of the default first field
note = genanki.Note(
    model=my_model,
    fields=['Q: What year?', '1789', 'French Revolution'],
    sort_field=1,
)
```

### Note with Audio and Image

```python
note = genanki.Note(
    model=my_model,
    fields=[
        'What does "Bonjour" sound like?',
        '[sound:bonjour.mp3]',       # Anki sound tag
        '<img src="flag_fr.jpg">',   # Anki image tag
    ]
)
# Remember to include the media files in the Package!
```

-----

## Deck

### Simple Deck

```python
deck = genanki.Deck(
    deck_id=2059400110,
    name='My Deck',
    description='A deck for learning things.'
)
deck.add_note(note1)
deck.add_note(note2)
```

### Sub-Decks (Nested)

```python
# Use :: to create hierarchy
parent_deck = genanki.Deck(1111111111, 'Languages')
french_deck  = genanki.Deck(2222222222, 'Languages::French')
german_deck  = genanki.Deck(3333333333, 'Languages::German')
advanced_fr  = genanki.Deck(4444444444, 'Languages::French::Advanced')

french_deck.add_note(some_note)
german_deck.add_note(another_note)

# Package all decks together
pkg = genanki.Package([parent_deck, french_deck, german_deck, advanced_fr])
pkg.write_to_file('languages.apkg')
```

-----

## Package

### Single Deck

```python
genanki.Package(my_deck).write_to_file('deck.apkg')
```

### Multiple Decks

```python
pkg = genanki.Package([deck1, deck2, deck3])
pkg.write_to_file('all_decks.apkg')
```

### With Media Files

```python
pkg = genanki.Package(my_deck)
pkg.media_files = [
    'audio/hello.mp3',
    'audio/goodbye.mp3',
    'images/cat.jpg',
    '/absolute/path/to/image.png',
]
pkg.write_to_file('deck_with_media.apkg')
```

-----

## Built-in Model Presets

genanki ships with ready-made models you can use directly:

|Constant                                    |Description                                 |
|--------------------------------------------|--------------------------------------------|
|`genanki.BASIC_MODEL`                       |Simple front/back card                      |
|`genanki.BASIC_AND_REVERSED_CARD_MODEL`     |Generates 2 cards per note (both directions)|
|`genanki.BASIC_OPTIONAL_REVERSED_CARD_MODEL`|Reversed card only if 3rd field is non-empty|
|`genanki.CLOZE_MODEL`                       |Cloze deletion cards                        |

```python
# Use a built-in model directly
note = genanki.Note(
    model=genanki.BASIC_MODEL,
    fields=['Front text', 'Back text']
)

# Cloze example
note = genanki.Note(
    model=genanki.CLOZE_MODEL,
    # Fields: [Text, Back Extra]
    fields=['The capital of France is {{c1::Paris}}.', 'Extra info here'],
)
```

> **Note:** Do **not** create new IDs when using built-in models. Their IDs are already set.

-----

## Card Templates & Styling

### Special Template Variables

|Variable            |Description                                        |
|--------------------|---------------------------------------------------|
|`{{FieldName}}`     |Renders the named field’s value                    |
|`{{FrontSide}}`     |Inserts full rendered front (answer templates only)|
|`{{Tags}}`          |Renders the note’s tags                            |
|`{{Type}}`          |Note type name                                     |
|`{{Deck}}`          |Current deck name                                  |
|`{{Subdeck}}`       |Current subdeck name                               |
|`{{Card}}`          |Card template name                                 |
|`{{hint:FieldName}}`|Renders field as a collapsible hint button         |
|`{{text:FieldName}}`|Renders field with HTML stripped                   |
|`{{c1::text}}`      |Cloze deletion (cloze model only)                  |

### Conditional Display

```python
# Show content only if field is non-empty
qfmt = '{{#HintField}}Hint: {{HintField}}<br>{{/HintField}}{{MainField}}'

# Show content only if field IS empty
qfmt = '{{^HintField}}No hint available{{/HintField}}'
```

### HTML in Fields

```python
note = genanki.Note(
    model=my_model,
    fields=[
        '<b>Bold</b> and <i>italic</i> text',
        '<ul><li>Item 1</li><li>Item 2</li></ul>',
    ]
)
```

-----

## Media Files

### Audio

```python
# In the field value, use Anki's sound tag:
fields = ['Word: Cat', '[sound:cat_meow.mp3]']

# Bundle the file in the package:
pkg = genanki.Package(deck)
pkg.media_files = ['cat_meow.mp3']
pkg.write_to_file('output.apkg')
```

### Images

```python
# In the field value, use an img tag:
fields = ['What animal is this?', '<img src="cat.jpg">']

pkg = genanki.Package(deck)
pkg.media_files = ['cat.jpg']
pkg.write_to_file('output.apkg')
```

### Collecting Media with Glob

```python
import glob

media = glob.glob('assets/audio/*.mp3') + glob.glob('assets/images/*.jpg')
pkg = genanki.Package(deck, media_files=media)
pkg.write_to_file('output.apkg')
```

-----

## GUIDs & ID Generation

### Why GUIDs Matter

Anki uses GUIDs to identify notes across imports:

- **Same GUID** → existing card is updated (scheduling preserved)
- **Different GUID** → duplicate card is created

### GUID Strategies

```python
import genanki

# Strategy 1: Deterministic from field content (RECOMMENDED)
guid = genanki.guid_for('Paris', 'France')

# Strategy 2: Deterministic from a stable unique key in your source data
guid = genanki.guid_for('vocab_item_042')

# Strategy 3: Hard-coded integer (must be globally unique)
guid = 987654321

# Strategy 4: Auto-generated (DEFAULT if not set)
# WARNING: changes each run — creates duplicates on re-import
```

### Generating Safe Deck/Model IDs

```python
import random

# Generate once, then hard-code the result
model_id = random.randrange(1 << 30, 1 << 31)
deck_id  = random.randrange(1 << 30, 1 << 31)

# Or derive deterministically from a name string
import hashlib
def id_from_name(name: str) -> int:
    return int(hashlib.md5(name.encode()).hexdigest()[:8], 16)

model_id = id_from_name('MyVocabularyModel')
```

### ID Safe Ranges

|Resource       |Recommended Range                    |
|---------------|-------------------------------------|
|Model ID       |`1073741824` – `2147483648`          |
|Deck ID        |`1073741824` – `2147483648`          |
|Note GUID (int)|Any positive integer, globally unique|

-----

## Advanced Techniques

### Building from CSV

```python
import csv, genanki

MODEL = genanki.Model(1234567890, 'CSV Model',
    fields=[{'name': 'Q'}, {'name': 'A'}],
    templates=[{
        'name': 'Card 1',
        'qfmt': '{{Q}}',
        'afmt': '{{FrontSide}}<hr id=answer>{{A}}'
    }]
)

deck = genanki.Deck(9876543210, 'From CSV')

with open('cards.csv', newline='', encoding='utf-8') as f:
    for row in csv.DictReader(f):
        deck.add_note(genanki.Note(
            model=MODEL,
            fields=[row['question'], row['answer']],
            guid=genanki.guid_for(row['question']),
            tags=row.get('tags', '').split(),
        ))

genanki.Package(deck).write_to_file('from_csv.apkg')
```

### Building from a Pandas DataFrame

```python
import pandas as pd, genanki

df = pd.read_csv('vocab.csv')   # columns: word, definition, example

deck = genanki.Deck(1122334455, 'Vocabulary')
for _, row in df.iterrows():
    deck.add_note(genanki.Note(
        model=MY_MODEL,
        fields=[row['word'], row['definition'], row['example']],
        guid=genanki.guid_for(row['word']),
    ))

genanki.Package(deck).write_to_file('vocab.apkg')
```

### Multiple Templates (Bidirectional Cards)

```python
BIDIRECTIONAL_MODEL = genanki.Model(
    5678901234,
    'Bidirectional',
    fields=[{'name': 'Front'}, {'name': 'Back'}],
    templates=[
        {
            'name': 'Front to Back',
            'qfmt': '{{Front}}',
            'afmt': '{{FrontSide}}<hr id=answer>{{Back}}',
        },
        {
            'name': 'Back to Front',
            'qfmt': '{{Back}}',
            'afmt': '{{FrontSide}}<hr id=answer>{{Front}}',
        },
    ]
)
# Each Note generates TWO cards automatically
```

### Cloze Deletions

```python
CLOZE = genanki.CLOZE_MODEL

notes = [
    '{{c1::Mitosis}} produces {{c2::two}} identical daughter cells.',
    'The speed of light is {{c1::299,792,458}} meters per second.',
    '{{c1::Python}} was created by {{c2::Guido van Rossum}} in {{c3::1991}}.',
]

deck = genanki.Deck(1357924680, 'Cloze Facts')
for text in notes:
    deck.add_note(genanki.Note(
        model=CLOZE,
        fields=[text, ''],          # second field is 'Back Extra'
        guid=genanki.guid_for(text),
    ))

genanki.Package(deck).write_to_file('cloze.apkg')
```

### Deck with Images and Audio

```python
import os

deck = genanki.Deck(2468013579, 'Animals with Sound')
media_files = []

animals = [
    ('Cat',  'cat.jpg',  'cat.mp3'),
    ('Dog',  'dog.jpg',  'dog.mp3'),
    ('Bird', 'bird.jpg', 'bird.mp3'),
]

for name, img, audio in animals:
    deck.add_note(genanki.Note(
        model=MY_MODEL,
        fields=[
            f'<img src="{img}">',
            f'{name}<br>[sound:{audio}]',
        ],
        guid=genanki.guid_for(name),
    ))
    media_files.extend([f'images/{img}', f'audio/{audio}'])

pkg = genanki.Package(deck)
pkg.media_files = [f for f in media_files if os.path.exists(f)]
pkg.write_to_file('animals.apkg')
```

### Dynamic Decks from JSON

```python
import json, genanki

with open('deck_data.json') as f:
    data = json.load(f)

model = genanki.Model(
    data['model_id'], data['model_name'],
    fields=[{'name': n} for n in data['fields']],
    templates=data['templates'],
    css=data.get('css', '')
)

deck = genanki.Deck(data['deck_id'], data['deck_name'])
for card in data['cards']:
    deck.add_note(genanki.Note(
        model=model,
        fields=card['fields'],
        tags=card.get('tags', []),
        guid=genanki.guid_for(*card['fields']),
    ))

genanki.Package(deck).write_to_file(data['output'])
```

### Stable Re-import (Idempotent Updates)

```python
# Running this script multiple times will NOT create duplicates
# because guid_for() produces the same stable ID each time.
# Anki will UPDATE the fields but PRESERVE scheduling history.

for item in data_source:
    note = genanki.Note(
        model=model,
        fields=[item.front, item.back],
        guid=genanki.guid_for(item.stable_id),  # stable key from source
    )
    deck.add_note(note)
```

-----

## Common Patterns & Recipes

### Class-Based Deck Builder (Chainable)

```python
import genanki

class DeckBuilder:
    def __init__(self, deck_id: int, name: str, model: genanki.Model):
        self.deck = genanki.Deck(deck_id, name)
        self.model = model
        self.media: list[str] = []

    def add(self, fields: list[str], tags: list[str] = None, media: list[str] = None):
        self.deck.add_note(genanki.Note(
            model=self.model,
            fields=fields,
            tags=tags or [],
            guid=genanki.guid_for(*fields),
        ))
        if media:
            self.media.extend(media)
        return self  # enables method chaining

    def save(self, path: str):
        pkg = genanki.Package(self.deck)
        pkg.media_files = self.media
        pkg.write_to_file(path)

# Usage with chaining
(DeckBuilder(1234567890, 'Spanish Vocab', MY_MODEL)
    .add(['Hola', 'Hello'])
    .add(['Gracias', 'Thank you'])
    .add(['Por favor', 'Please'])
    .save('spanish.apkg'))
```

### Hierarchical Tags

```python
# Anki supports :: for tag hierarchy (shows as tree in browser)
tags = ['language::french', 'level::A1', 'topic::greetings']

note = genanki.Note(model=model, fields=['Bonjour', 'Hello'], tags=tags)
```

### Multi-Deck Package Helper

```python
def build_package(deck_configs: list[dict], output: str):
    """
    Each dict in deck_configs should have: id, name, notes (list of Note objects)
    """
    decks = []
    for cfg in deck_configs:
        deck = genanki.Deck(cfg['id'], cfg['name'])
        for note in cfg['notes']:
            deck.add_note(note)
        decks.append(deck)
    genanki.Package(decks).write_to_file(output)
```

-----

## Troubleshooting

|Problem                          |Likely Cause                   |Solution                                                        |
|---------------------------------|-------------------------------|----------------------------------------------------------------|
|Duplicate cards on re-import     |Random/missing GUIDs           |Use `guid_for()` with a stable key                              |
|`AssertionError` on Note creation|Field count mismatch           |Ensure `len(fields) == len(model.fields)`                       |
|Media not appearing              |File missing from `media_files`|Add all file paths to `pkg.media_files`                         |
|Media not appearing              |Filename mismatch              |Filename in `[sound:x]` / `<img src="x">` must match exactly    |
|Sub-decks not showing            |Missing parent deck in package |Include all deck levels in `Package([...])`                     |
|Cards not updating on re-import  |GUID changed between runs      |Hard-code or deterministically derive GUIDs                     |
|`OverflowError` on ID            |ID out of valid range          |Use IDs in range `1<<30` to `1<<31`                             |
|Styling not applied              |CSS or specificity issue       |Target `.card` class; check template HTML                       |
|Cloze card not generated         |Wrong model or wrong syntax    |Use `CLOZE_MODEL`; field must contain `{{c1::...}}`             |
|`KeyError` in template           |Field name typo                |Field names in templates must exactly match `model.fields` names|

-----

## Minimal Project Template

```python
import genanki

# Hard-code these after generating once with random.randrange(1<<30, 1<<31)
MODEL_ID = 1234567890
DECK_ID  = 9876543210

MODEL = genanki.Model(
    MODEL_ID,
    'My Model',
    fields=[
        {'name': 'Front'},
        {'name': 'Back'},
    ],
    templates=[{
        'name': 'Card 1',
        'qfmt': '{{Front}}',
        'afmt': '{{FrontSide}}<hr id=answer>{{Back}}',
    }],
    css='.card { font-family: Arial; font-size: 20px; text-align: center; }'
)

DECK = genanki.Deck(DECK_ID, 'My Deck')

ITEMS = [
    ('Question 1', 'Answer 1'),
    ('Question 2', 'Answer 2'),
]

for q, a in ITEMS:
    DECK.add_note(genanki.Note(
        model=MODEL,
        fields=[q, a],
        guid=genanki.guid_for(q),   # stable, deterministic
    ))

genanki.Package(DECK).write_to_file('my_deck.apkg')
print('Done! Import my_deck.apkg into Anki.')
```

-----

*Reference card for genanki — verified against genanki v0.13+*
