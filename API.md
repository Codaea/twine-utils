# Twine Utils API Documentation

A comprehensive guide to using twine-utils for working with Twine stories programmatically.

## Table of Contents

1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Story Class](#story-class)
5. [Passage Class](#passage-class)
6. [StoryFormat Class](#storyformat-class)
7. [Path Module](#path-module)
8. [Complete Examples](#complete-examples)
9. [Best Practices](#best-practices)

## Introduction

`twine-utils` is a Node.js library that enables you to work with Twine stories programmatically. You can:

- Create stories from scratch or load them from various formats (Twee, HTML, TWS)
- Manipulate story content, passages, and metadata
- Combine multiple stories or passages
- Add custom JavaScript and CSS
- Publish stories to playable HTML using story formats
- Convert between different Twine formats

This library is designed for Node.js environments and is ideal for:
- Building custom Twine editing tools
- Automating story generation
- Creating build pipelines for Twine projects
- Converting between different Twine formats

## Installation

```bash
npm install twine-utils
```

## Quick Start

### Creating a Simple Story

```javascript
import {Story, Passage, StoryFormat} from 'twine-utils';

// Create a new story
const story = new Story({
  attributes: {
    name: 'My First Story',
    ifid: 'D674C58C-DEFA-4F70-B7A2-27742230C0FC'
  }
});

// Add passages
story.passages.push(new Passage({
  attributes: {
    name: 'Start',
    tags: []
  },
  source: 'Welcome to my story! [[Continue->Next]]'
}));

story.passages.push(new Passage({
  attributes: {
    name: 'Next',
    tags: []
  },
  source: 'This is the next passage.'
}));

// Set the starting passage
story.setStartByName('Start');

console.log('Story created with', story.passages.length, 'passages');
```

### Publishing a Story

```javascript
import {readFile} from 'fs/promises';
import {Story, StoryFormat} from 'twine-utils';

// Load a story format (e.g., Harlowe)
const formatSource = await readFile('./harlowe-3.3.0.js', 'utf8');
const format = new StoryFormat();
await format.load(formatSource);

// Create or load your story
const story = new Story({
  attributes: {name: 'My Published Story'}
});

// Add passages to your story
// ... (add your passages here)

// Publish to HTML
const publishedHtml = format.publish(story);

// Save to file
await writeFile('story.html', publishedHtml);
console.log('Story published to story.html');
```

## Story Class

The `Story` class represents a complete Twine story with all its passages, attributes, and custom code.

### Constructor

```javascript
new Story(options?: StoryOptions)
```

Creates a new Story instance.

**Parameters:**
- `options.attributes` (Record<string, unknown>): Story attributes like `name`, `ifid`, `startnode`, `zoom`, etc.
- `options.javascript` (string): Custom JavaScript code for the story
- `options.stylesheet` (string): Custom CSS for the story
- `options.passages` (Passage[]): Array of passages in the story

**Example:**

```javascript
const story = new Story({
  attributes: {
    name: 'Adventure Game',
    ifid: 'E1C2D3A4-B5F6-4A7B-8C9D-0E1F2A3B4C5D',
    zoom: 1.0,
    format: 'Harlowe',
    'format-version': '3.3.0'
  },
  javascript: 'console.log("Story loaded!");',
  stylesheet: 'body { background-color: #222; color: #fff; }',
  passages: []
});
```

### Properties

#### `attributes`

```javascript
story.attributes: Record<string, unknown>
```

General attributes of the story. Common attributes include:
- `name` (string): Story title
- `ifid` (string): Interactive Fiction ID (unique identifier)
- `startnode` (string): PID of the starting passage
- `creator` (string): Tool that created the story (defaults to 'twine-utils')
- `creator-version` (string): Version of the creator tool
- `zoom` (number): Zoom level of the story map
- `format` (string): Story format name
- `format-version` (string): Story format version

**Example:**

```javascript
story.attributes.name = 'My Epic Tale';
story.attributes.ifid = crypto.randomUUID();
story.attributes.zoom = 1.5;
```

#### `passages`

```javascript
story.passages: Passage[]
```

Array of all passages in the story.

**Example:**

```javascript
console.log(`Story has ${story.passages.length} passages`);

// Add a passage
story.passages.push(new Passage({
  attributes: {name: 'NewPassage'},
  source: 'New content'
}));

// Find a passage by name
const passage = story.passages.find(p => p.attributes.name === 'Start');
```

#### `startPassage`

```javascript
story.startPassage?: Passage
```

The starting passage of the story. This should always be a member of the `passages` array.

**Example:**

```javascript
if (story.startPassage) {
  console.log('Story starts at:', story.startPassage.attributes.name);
}
```

#### `javascript`

```javascript
story.javascript: string
```

Custom JavaScript code that runs when the story loads.

**Example:**

```javascript
story.javascript = `
  // Custom story JavaScript
  window.storyData = {
    version: '1.0',
    author: 'Your Name'
  };
`;
```

#### `stylesheet`

```javascript
story.stylesheet: string
```

Custom CSS stylesheet for the story.

**Example:**

```javascript
story.stylesheet = `
  tw-story {
    font-family: 'Georgia', serif;
    font-size: 18px;
  }
  
  tw-passage {
    color: #333;
  }
`;
```

### Static Methods

#### `Story.fromHTML()`

```javascript
Story.fromHTML(source: string, twineVersion?: number, silent?: boolean): Story
```

Creates a Story from HTML source code.

**Parameters:**
- `source` (string): HTML source code containing a Twine story
- `twineVersion` (number): Version of Twine (1 or 2, defaults to 2)
- `silent` (boolean): If true, suppresses console warnings (defaults to false)

**Returns:** A new Story instance

**Example:**

```javascript
import {readFile} from 'fs/promises';

// Load a Twine 2 HTML story
const html = await readFile('story.html', 'utf8');
const story = Story.fromHTML(html, 2);

console.log('Loaded story:', story.attributes.name);
console.log('Passages:', story.passages.length);

// Load a Twine 1 HTML story
const twine1Html = await readFile('old-story.html', 'utf8');
const oldStory = Story.fromHTML(twine1Html, 1);
```

#### `Story.fromTwee()`

```javascript
Story.fromTwee(source: string, tweeVersion?: number, silent?: boolean): Story
```

Creates a Story from Twee source code.

**Parameters:**
- `source` (string): Twee source code
- `tweeVersion` (number): Version of Twee spec to use (defaults to 1)
- `silent` (boolean): If true, suppresses console warnings (defaults to false)

**Returns:** A new Story instance

**Twee Format:**

```
:: Passage Name [tag1 tag2] {"position": "100,100"}
Passage content goes here.
You can include [[links->OtherPassage]].

:: OtherPassage
More content here.

:: StoryTitle
My Story Title

:: StoryData
{
  "ifid": "D674C58C-DEFA-4F70-B7A2-27742230C0FC",
  "format": "Harlowe",
  "format-version": "3.3.0"
}
```

**Example:**

```javascript
const tweeSource = `
:: Start [intro]
Welcome to the story!
Do you want to [[explore]] or [[rest]]?

:: explore [adventure]
You venture into the unknown!

:: rest
You take a moment to rest.

:: StoryTitle
My Twee Adventure

:: StoryData
{
  "ifid": "12345678-1234-1234-1234-123456789012"
}
`;

const story = Story.fromTwee(tweeSource, 3);
console.log('Story:', story.attributes.name);
console.log('Passages:', story.passages.map(p => p.attributes.name));
```

#### `Story.fromTWS()`

```javascript
Story.fromTWS(buffer: Uint8Array | Int8Array | Uint8ClampedArray): Story
```

Creates a Story from TWS (Twine 1 story file) format.

**Parameters:**
- `buffer`: Binary buffer containing TWS data

**Returns:** A new Story instance

**Example:**

```javascript
import {readFile} from 'fs/promises';

const twsBuffer = await readFile('story.tws');
const story = Story.fromTWS(twsBuffer);
console.log('Loaded TWS story:', story.attributes.name);
```

### Instance Methods

#### `mergeStory()`

```javascript
story.mergeStory(otherStory: Story): Story
```

Merges another story into this one, combining passages, JavaScript, CSS, and attributes.

**Parameters:**
- `otherStory` (Story): Story to merge into this one (will not be modified)

**Returns:** This story instance (for chaining)

**Example:**

```javascript
// Create main story
const mainStory = new Story({
  attributes: {name: 'Main Story'},
  passages: [
    new Passage({
      attributes: {name: 'Start'},
      source: 'Beginning of the main story.'
    })
  ]
});

// Create additional content
const additionalContent = new Story({
  passages: [
    new Passage({
      attributes: {name: 'ExtraScene'},
      source: 'Additional scene content.'
    })
  ],
  javascript: 'console.log("Extra JS");'
});

// Merge them
mainStory.mergeStory(additionalContent);

console.log('Combined story has', mainStory.passages.length, 'passages');
// Output: Combined story has 2 passages
```

#### `mergeJavaScript()`

```javascript
story.mergeJavaScript(source: string): Story
```

Adds JavaScript code to the story's existing JavaScript.

**Parameters:**
- `source` (string): JavaScript code to add

**Returns:** This story instance (for chaining)

**Example:**

```javascript
const story = new Story();

story.mergeJavaScript('window.customVar = 42;');
story.mergeJavaScript('console.log("Story loaded:", window.customVar);');

console.log(story.javascript);
// Output:
// 
// window.customVar = 42;
// console.log("Story loaded:", window.customVar);
```

#### `mergeStylesheet()`

```javascript
story.mergeStylesheet(source: string): Story
```

Adds CSS to the story's existing stylesheet.

**Parameters:**
- `source` (string): CSS code to add

**Returns:** This story instance (for chaining)

**Example:**

```javascript
const story = new Story();

story.mergeStylesheet('body { margin: 0; padding: 20px; }');
story.mergeStylesheet('.highlight { color: red; font-weight: bold; }');

console.log(story.stylesheet);
// Output:
// 
// body { margin: 0; padding: 20px; }
// .highlight { color: red; font-weight: bold; }
```

#### `setStartByName()`

```javascript
story.setStartByName(name: string): Story
```

Sets the starting passage by name. Throws an error if the passage doesn't exist.

**Parameters:**
- `name` (string): Name of the passage to set as start

**Returns:** This story instance (for chaining)

**Throws:** Error if passage with the given name doesn't exist

**Example:**

```javascript
const story = new Story();

story.passages.push(
  new Passage({attributes: {name: 'Intro'}, source: 'Welcome!'}),
  new Passage({attributes: {name: 'Start'}, source: 'The story begins...'})
);

story.setStartByName('Intro');
console.log('Start passage:', story.startPassage.attributes.name);
// Output: Start passage: Intro

// This would throw an error:
// story.setStartByName('NonExistent'); // Error!
```

#### `toHTML()`

```javascript
story.toHTML(): string
```

Returns a Twine 2 HTML fragment for this story. This is not a complete HTML page—use `StoryFormat.publish()` to create a playable HTML file.

**Returns:** HTML string containing the story data

**Example:**

```javascript
const story = new Story({
  attributes: {name: 'Test Story'},
  passages: [
    new Passage({
      attributes: {name: 'Start'},
      source: 'Hello, world!'
    })
  ]
});

const htmlFragment = story.toHTML();
console.log(htmlFragment);
// Output: <tw-storydata name="Test Story" ...>...</tw-storydata>
```

#### `toTwee()`

```javascript
story.toTwee(tweeVersion?: number, passageSpacer?: string): string
```

Converts the story to Twee source code format.

**Parameters:**
- `tweeVersion` (number): Version of Twee spec to use (defaults to 3)
- `passageSpacer` (string): Text between passages (defaults to '\n\n')

**Returns:** Twee source code string

**Note:** Converting to Twee versions less than 3 is lossy (some metadata may be lost).

**Example:**

```javascript
const story = new Story({
  attributes: {
    name: 'My Story',
    ifid: 'ABC-123'
  }
});

story.passages.push(
  new Passage({
    attributes: {name: 'Start', tags: ['intro']},
    source: 'Welcome to the adventure!'
  }),
  new Passage({
    attributes: {name: 'Next'},
    source: 'The story continues...'
  })
);

const tweeSource = story.toTwee(3);
console.log(tweeSource);
// Output:
// :: Start [intro]
// Welcome to the adventure!
//
// :: Next
// The story continues...
//
// :: StoryTitle
// My Story
//
// :: StoryData
// {
//   "ifid": "ABC-123",
//   ...
// }
```

## Passage Class

The `Passage` class represents a single passage (scene/node) in a Twine story.

### Constructor

```javascript
new Passage(options?: PassageOptions)
```

Creates a new Passage instance.

**Parameters:**
- `options.attributes` (Record<string, unknown>): Passage attributes
- `options.source` (string): Source text/content of the passage

**Example:**

```javascript
const passage = new Passage({
  attributes: {
    name: 'Forest',
    tags: ['outdoor', 'exploration'],
    position: '100,200',
    size: '100,100'
  },
  source: 'You find yourself in a dark forest.\n[[Go north->Cave]]\n[[Go south->Village]]'
});
```

### Properties

#### `attributes`

```javascript
passage.attributes: Record<string, unknown>
```

Passage attributes that appear in the `<tw-passagedata>` element. Common attributes:
- `name` (string): Passage name (required)
- `tags` (string[]): Array of tags for categorizing passages
- `pid` (number/string): Passage ID
- `position` (string): Position in editor (format: "x,y")
- `size` (string): Size in editor (format: "width,height")

**Example:**

```javascript
const passage = new Passage();

passage.attributes.name = 'Dungeon Entrance';
passage.attributes.tags = ['dungeon', 'combat'];
passage.attributes.position = '250,150';
passage.attributes.size = '100,100';

// Custom attributes
passage.attributes.difficulty = 'hard';
passage.attributes.author = 'John Doe';
```

#### `source`

```javascript
passage.source: string
```

The text content of the passage, including any Twine markup.

**Example:**

```javascript
const passage = new Passage({attributes: {name: 'Test'}});

passage.source = `
You stand at a crossroads.

(if: $hasKey)[
  [[Unlock the door->SecretRoom]]
]

[[Go back->Start]]
`;
```

### Static Methods

#### `Passage.fromHTML()`

```javascript
Passage.fromHTML(source: string, silent?: boolean): Passage
```

Creates a Passage from an HTML fragment containing a `<tw-passagedata>` element.

**Parameters:**
- `source` (string): HTML source containing passage data
- `silent` (boolean): If true, suppresses warnings (defaults to false)

**Returns:** A new Passage instance

**Example:**

```javascript
const html = `
  <tw-passagedata pid="1" name="Start" tags="intro" position="10,10">
    Welcome to the story!
  </tw-passagedata>
`;

const passage = Passage.fromHTML(html);
console.log('Loaded passage:', passage.attributes.name);
console.log('Tags:', passage.attributes.tags);
console.log('Content:', passage.source);
```

### Instance Methods

#### `toHTML()`

```javascript
passage.toHTML(pid?: number): string
```

Converts the passage to an HTML fragment.

**Parameters:**
- `pid` (number): Optional passage ID to set

**Returns:** HTML string

**Example:**

```javascript
const passage = new Passage({
  attributes: {
    name: 'TestPassage',
    tags: ['test']
  },
  source: 'This is a test passage.'
});

const html = passage.toHTML(1);
console.log(html);
// Output: <tw-passagedata name="TestPassage" tags="test" pid="1">This is a test passage.</tw-passagedata>
```

#### `toTwee()`

```javascript
passage.toTwee(tweeVersion?: number): string
```

Converts the passage to Twee format.

**Parameters:**
- `tweeVersion` (number): Twee specification version (defaults to 3)

**Returns:** Twee source string

**Note:** Versions less than 3 may lose some metadata.

**Example:**

```javascript
const passage = new Passage({
  attributes: {
    name: 'Cave',
    tags: ['dark', 'dangerous'],
    position: '150,200'
  },
  source: 'The cave is dark and foreboding.'
});

const twee = passage.toTwee(3);
console.log(twee);
// Output:
// :: Cave [dark dangerous] {"position":"150,200"}
// The cave is dark and foreboding.
```

## StoryFormat Class

The `StoryFormat` class represents a Twine story format (like Harlowe, SugarCube, Snowman) and is used to publish stories to playable HTML.

### Constructor

```javascript
new StoryFormat(rawSource?: string)
```

Creates a new StoryFormat instance.

**Parameters:**
- `rawSource` (string): Optional JSONP source to load immediately

**Example:**

```javascript
import {readFile} from 'fs/promises';

// Create empty format
const format = new StoryFormat();

// Or create and load in one step
const formatSource = await readFile('harlowe.js', 'utf8');
const loadedFormat = new StoryFormat(formatSource);
```

### Properties

#### `attributes`

```javascript
storyFormat.attributes: Record<string, unknown>
```

Parsed attributes from the story format. Common attributes:
- `name` (string): Format name (e.g., "Harlowe", "SugarCube")
- `version` (string): Format version (e.g., "3.3.0")
- `author` (string): Format author
- `description` (string): Format description
- `source` (string): The HTML template for publishing

**Example:**

```javascript
const format = new StoryFormat();
await format.load(formatSource);

console.log('Format:', format.attributes.name);
console.log('Version:', format.attributes.version);
console.log('Author:', format.attributes.author);
```

### Instance Methods

#### `load()`

```javascript
storyFormat.load(rawSource: string): Promise<void>
```

Loads a story format from JSONP source. The format must follow the Twine story format specification.

**Parameters:**
- `rawSource` (string): JSONP source code of the format

**Returns:** Promise that resolves when loading is complete

**Throws:** Error if format is malformed or another format is currently loaded

**Example:**

```javascript
import {readFile} from 'fs/promises';

const format = new StoryFormat();

// Load Harlowe format
const harlowe = await readFile('./formats/harlowe-3.3.0.js', 'utf8');
await format.load(harlowe);

console.log('Loaded:', format.attributes.name, format.attributes.version);

// Now you can publish stories
const html = format.publish(myStory);
```

#### `publish()`

```javascript
storyFormat.publish(story: Story): string
```

Publishes a story to playable HTML using this format.

**Parameters:**
- `story` (Story): The story to publish

**Returns:** Complete HTML page as a string

**Throws:** Error if format has no source or source is not a string

**Example:**

```javascript
import {writeFile} from 'fs/promises';

// Create a story
const story = new Story({
  attributes: {name: 'My Game'}
});

story.passages.push(
  new Passage({
    attributes: {name: 'Start'},
    source: 'Welcome to my game!'
  })
);

story.setStartByName('Start');

// Load format and publish
const format = new StoryFormat();
await format.load(harloweSrc);

const html = format.publish(story);

// Save to file
await writeFile('my-game.html', html);
console.log('Game published to my-game.html');
```

#### `toJSONP()`

```javascript
storyFormat.toJSONP(callbackName?: string): string
```

Converts the story format to JSONP representation.

**Parameters:**
- `callbackName` (string): Name of the callback function (defaults to 'storyFormat')

**Returns:** JSONP string

**Example:**

```javascript
import {writeFile} from 'fs/promises';

const format = new StoryFormat();
// ... set format.attributes ...

const jsonp = format.toJSONP('storyFormat');
await writeFile('my-format.js', jsonp);
console.log('Format saved');
```

## Path Module

The Path module provides functions to locate Twine's story directory on the user's system.

### Functions

#### `storyDirectory()`

```javascript
Path.storyDirectory(): Promise<string>
```

Returns a promise that resolves to the absolute path of the user's Twine Stories directory.

**Returns:** Promise<string> - Absolute path to Stories directory

**Throws:** Error if Twine directory or Stories directory cannot be found

**Example:**

```javascript
import * as Path from 'twine-utils/path';
import {readdir, readFile} from 'fs/promises';
import {join} from 'path';

try {
  const storiesPath = await Path.storyDirectory();
  console.log('Stories directory:', storiesPath);
  
  // List all story files
  const files = await readdir(storiesPath);
  const storyFiles = files.filter(f => f.endsWith('.html'));
  
  console.log('Found stories:', storyFiles);
  
  // Load a story
  if (storyFiles.length > 0) {
    const storyPath = join(storiesPath, storyFiles[0]);
    const storyHtml = await readFile(storyPath, 'utf8');
    const story = Story.fromHTML(storyHtml);
    console.log('Loaded:', story.attributes.name);
  }
} catch (error) {
  console.error('Could not find Twine directory:', error.message);
}
```

#### `storyDirectorySync()`

```javascript
Path.storyDirectorySync(): string
```

Synchronous version of `storyDirectory()`.

**Returns:** Absolute path to Stories directory

**Throws:** Error if Twine directory or Stories directory cannot be found

**Example:**

```javascript
import * as Path from 'twine-utils/path';
import {readdirSync, readFileSync} from 'fs';
import {join} from 'path';

try {
  const storiesPath = Path.storyDirectorySync();
  const files = readdirSync(storiesPath);
  
  console.log('Stories found:', files.length);
} catch (error) {
  console.error('Cannot access Twine directory:', error.message);
}
```

## Complete Examples

### Example 1: Creating and Publishing a Complete Story

```javascript
import {Story, Passage, StoryFormat} from 'twine-utils';
import {readFile, writeFile} from 'fs/promises';

async function createAndPublishStory() {
  // Create the story
  const story = new Story({
    attributes: {
      name: 'The Mysterious Island',
      ifid: 'A1B2C3D4-E5F6-4789-A012-B3C4D5E6F789',
      format: 'Harlowe',
      'format-version': '3.3.0'
    }
  });

  // Add custom styling
  story.mergeStylesheet(`
    tw-story {
      background-color: #1a1a1a;
      color: #e0e0e0;
      font-family: 'Georgia', serif;
    }
    
    tw-link {
      color: #4a9eff;
    }
    
    tw-link:hover {
      color: #7bb8ff;
    }
  `);

  // Add custom JavaScript
  story.mergeJavaScript(`
    // Track player progress
    window.gameState = {
      visitedLocations: [],
      inventory: []
    };
  `);

  // Create passages
  const passages = [
    new Passage({
      attributes: {
        name: 'Start',
        tags: ['intro'],
        position: '100,100'
      },
      source: `
You wake up on a mysterious island, with no memory of how you got here.

The sun is setting, and you need to find shelter soon.

[[Explore the beach->Beach]]
[[Head into the jungle->Jungle]]
      `.trim()
    }),
    
    new Passage({
      attributes: {
        name: 'Beach',
        tags: ['location'],
        position: '250,100'
      },
      source: `
The beach stretches out before you, with white sand and crystal blue water.

You spot some debris from a shipwreck nearby.

[[Investigate the debris->Shipwreck]]
[[Return to the starting point->Start]]
      `.trim()
    }),
    
    new Passage({
      attributes: {
        name: 'Jungle',
        tags: ['location'],
        position: '100,250'
      },
      source: `
The jungle is dense and humid. Strange sounds echo through the trees.

You notice a path leading deeper into the vegetation.

[[Follow the path->Cave]]
[[Go back->Start]]
      `.trim()
    }),
    
    new Passage({
      attributes: {
        name: 'Shipwreck',
        tags: ['location', 'discovery'],
        position: '400,100'
      },
      source: `
Among the wreckage, you find a waterproof container with supplies:
- A flashlight
- A knife
- A compass

(set: $hasSupplies to true)

[[Take the supplies and continue->Beach]]
      `.trim()
    }),
    
    new Passage({
      attributes: {
        name: 'Cave',
        tags: ['location', 'discovery'],
        position: '100,400'
      },
      source: `
You discover a cave entrance. It looks like it could provide shelter for the night.

(if: $hasSupplies)[
  With your flashlight, you can safely explore inside.
  [[Enter the cave->CaveInterior]]
]
(else:)[
  It's too dark to enter safely without light.
  [[Go back->Jungle]]
]
      `.trim()
    }),
    
    new Passage({
      attributes: {
        name: 'CaveInterior',
        tags: ['location', 'ending'],
        position: '250,400'
      },
      source: `
Inside the cave, you find ancient drawings on the walls and a hidden passage that leads to civilization!

You've survived the mysterious island!

**THE END**

[[Start over->Start]]
      `.trim()
    })
  ];

  // Add all passages to the story
  passages.forEach(p => story.passages.push(p));

  // Set the starting passage
  story.setStartByName('Start');

  console.log(`Created story "${story.attributes.name}" with ${story.passages.length} passages`);

  // Load the Harlowe format
  const formatSource = await readFile('./harlowe-3.3.0.js', 'utf8');
  const format = new StoryFormat();
  await format.load(formatSource);

  console.log(`Loaded format: ${format.attributes.name} ${format.attributes.version}`);

  // Publish the story
  const publishedHtml = format.publish(story);

  // Save to file
  const outputFile = 'the-mysterious-island.html';
  await writeFile(outputFile, publishedHtml);

  console.log(`Story published to ${outputFile}`);
  console.log(`File size: ${(publishedHtml.length / 1024).toFixed(2)} KB`);
}

// Run the example
createAndPublishStory().catch(console.error);
```

### Example 2: Loading and Modifying an Existing Story

```javascript
import {Story, Passage} from 'twine-utils';
import {readFile, writeFile} from 'fs/promises';

async function modifyExistingStory() {
  // Load an existing story
  const html = await readFile('original-story.html', 'utf8');
  const story = Story.fromHTML(html);

  console.log(`Loaded: ${story.attributes.name}`);
  console.log(`Passages: ${story.passages.length}`);

  // Add a new passage
  story.passages.push(new Passage({
    attributes: {
      name: 'SecretEnding',
      tags: ['ending', 'secret']
    },
    source: 'Congratulations! You found the secret ending!'
  }));

  // Find and modify a specific passage
  const startPassage = story.passages.find(
    p => p.attributes.name === 'Start'
  );

  if (startPassage) {
    // Add a link to the secret ending
    startPassage.source += '\n\n[[Hidden path->SecretEnding]]';
    console.log('Modified Start passage');
  }

  // Add some custom CSS
  story.mergeStylesheet(`
    .secret-text {
      color: gold;
      font-style: italic;
    }
  `);

  // Update story metadata
  story.attributes['modified-date'] = new Date().toISOString();

  // Convert to Twee for easy editing
  const tweeOutput = story.toTwee(3);
  await writeFile('story-modified.twee', tweeOutput);

  console.log('Modified story saved as Twee');
}

modifyExistingStory().catch(console.error);
```

### Example 3: Combining Multiple Stories

```javascript
import {Story} from 'twine-utils';
import {readFile, writeFile} from 'fs/promises';

async function combineStories() {
  // Load multiple story files
  const story1Html = await readFile('chapter1.html', 'utf8');
  const story2Html = await readFile('chapter2.html', 'utf8');
  const story3Html = await readFile('chapter3.html', 'utf8');

  // Parse them
  const chapter1 = Story.fromHTML(story1Html);
  const chapter2 = Story.fromHTML(story2Html);
  const chapter3 = Story.fromHTML(story3Html);

  // Create a new combined story
  const combined = new Story({
    attributes: {
      name: 'Complete Story: All Chapters',
      ifid: crypto.randomUUID()
    }
  });

  // Merge all chapters
  combined.mergeStory(chapter1);
  combined.mergeStory(chapter2);
  combined.mergeStory(chapter3);

  // Set the starting passage (from chapter 1)
  if (chapter1.startPassage) {
    const startName = chapter1.startPassage.attributes.name;
    combined.setStartByName(startName);
  }

  console.log('Combined story created');
  console.log(`Total passages: ${combined.passages.length}`);

  // Save as Twee for review
  const tweeOutput = combined.toTwee(3);
  await writeFile('combined-story.twee', tweeOutput);

  console.log('Combined story saved');
}

combineStories().catch(console.error);
```

### Example 4: Converting Between Formats

```javascript
import {Story} from 'twine-utils';
import {readFile, writeFile} from 'fs/promises';

async function convertFormats() {
  console.log('=== Format Conversion Tool ===\n');

  // Load a Twine 1 HTML file
  console.log('Loading Twine 1 HTML...');
  const twine1Html = await readFile('old-story.html', 'utf8');
  const story = Story.fromHTML(twine1Html, 1);
  console.log(`Loaded: ${story.attributes.name}`);

  // Convert to Twee v3
  console.log('\nConverting to Twee v3...');
  const tweeSource = story.toTwee(3);
  await writeFile('converted.twee', tweeSource);
  console.log('Saved as: converted.twee');

  // Convert to Twine 2 HTML fragment
  console.log('\nConverting to Twine 2 HTML...');
  const twine2Fragment = story.toHTML();
  await writeFile('converted-fragment.html', twine2Fragment);
  console.log('Saved as: converted-fragment.html');

  // You could also load the Twee and convert back
  console.log('\nRound-trip test...');
  const reloadedStory = Story.fromTwee(tweeSource, 3);
  console.log(`Reloaded: ${reloadedStory.attributes.name}`);
  console.log(`Passages: ${reloadedStory.passages.length}`);

  console.log('\nConversion complete!');
}

convertFormats().catch(console.error);
```

### Example 5: Generating Stories from Data

```javascript
import {Story, Passage} from 'twine-utils';
import {writeFile} from 'fs/promises';

// Example: Generate a story from structured data
function generateStoryFromData(data) {
  const story = new Story({
    attributes: {
      name: data.title,
      ifid: crypto.randomUUID(),
      author: data.author
    }
  });

  // Generate passages from data
  data.scenes.forEach((scene, index) => {
    let source = scene.text + '\n\n';

    // Add choices as links
    if (scene.choices && scene.choices.length > 0) {
      scene.choices.forEach(choice => {
        source += `[[${choice.text}->${choice.target}]]\n`;
      });
    }

    const passage = new Passage({
      attributes: {
        name: scene.id,
        tags: scene.tags || [],
        position: `${100 + (index % 5) * 200},${100 + Math.floor(index / 5) * 200}`
      },
      source: source.trim()
    });

    story.passages.push(passage);
  });

  // Set start passage
  if (data.startScene) {
    story.setStartByName(data.startScene);
  }

  return story;
}

async function example() {
  // Sample data structure
  const gameData = {
    title: 'Data-Driven Adventure',
    author: 'Story Generator',
    startScene: 'intro',
    scenes: [
      {
        id: 'intro',
        text: 'Welcome to this automatically generated story!',
        tags: ['start'],
        choices: [
          {text: 'Begin the adventure', target: 'forest'},
          {text: 'Learn about the world', target: 'info'}
        ]
      },
      {
        id: 'forest',
        text: 'You enter a mysterious forest.',
        tags: ['location'],
        choices: [
          {text: 'Go deeper', target: 'cave'},
          {text: 'Turn back', target: 'intro'}
        ]
      },
      {
        id: 'cave',
        text: 'You discover a hidden cave. You win!',
        tags: ['location', 'ending'],
        choices: []
      },
      {
        id: 'info',
        text: 'This is a procedurally generated story. Have fun!',
        tags: ['info'],
        choices: [
          {text: 'Go back', target: 'intro'}
        ]
      }
    ]
  };

  // Generate the story
  const story = generateStoryFromData(gameData);

  console.log(`Generated: ${story.attributes.name}`);
  console.log(`Passages: ${story.passages.length}`);

  // Save as Twee
  const twee = story.toTwee(3);
  await writeFile('generated-story.twee', twee);

  console.log('Story generated and saved!');
}

example().catch(console.error);
```

### Example 6: Batch Processing Multiple Stories

```javascript
import {Story, StoryFormat} from 'twine-utils';
import {readdir, readFile, writeFile, mkdir} from 'fs/promises';
import {join} from 'path';

async function batchPublishStories() {
  const inputDir = './stories-source';
  const outputDir = './stories-published';

  // Create output directory
  await mkdir(outputDir, {recursive: true});

  // Load story format once
  console.log('Loading story format...');
  const formatSource = await readFile('./harlowe-3.3.0.js', 'utf8');
  const format = new StoryFormat();
  await format.load(formatSource);

  // Get all .twee files
  const files = await readdir(inputDir);
  const tweeFiles = files.filter(f => f.endsWith('.twee'));

  console.log(`Found ${tweeFiles.length} Twee files to process\n`);

  // Process each file
  for (const filename of tweeFiles) {
    const inputPath = join(inputDir, filename);
    const outputFilename = filename.replace('.twee', '.html');
    const outputPath = join(outputDir, outputFilename);

    console.log(`Processing: ${filename}`);

    try {
      // Load and parse Twee
      const tweeSource = await readFile(inputPath, 'utf8');
      const story = Story.fromTwee(tweeSource, 3);

      // Publish to HTML
      const html = format.publish(story);

      // Save output
      await writeFile(outputPath, html);

      console.log(`  ✓ Published: ${outputFilename}`);
      console.log(`  - Passages: ${story.passages.length}`);
      console.log(`  - Size: ${(html.length / 1024).toFixed(2)} KB\n`);
    } catch (error) {
      console.error(`  ✗ Error processing ${filename}:`, error.message, '\n');
    }
  }

  console.log('Batch processing complete!');
}

batchPublishStories().catch(console.error);
```

## Best Practices

### 1. Always Set Story Attributes

When creating stories, always set essential attributes:

```javascript
const story = new Story({
  attributes: {
    name: 'My Story',                          // Required for display
    ifid: crypto.randomUUID(),                 // Unique identifier
    format: 'Harlowe',                         // Story format name
    'format-version': '3.3.0',                 // Format version
    creator: 'My Tool',                        // Your tool name
    'creator-version': '1.0.0'                 // Your tool version
  }
});
```

### 2. Validate Passage Names

Before setting start passages or creating links, verify passage names exist:

```javascript
function safeSetStart(story, passageName) {
  const exists = story.passages.some(
    p => p.attributes.name === passageName
  );
  
  if (exists) {
    story.setStartByName(passageName);
  } else {
    console.warn(`Warning: Passage "${passageName}" not found`);
  }
}
```

### 3. Handle Errors When Loading Files

Always use try-catch when loading external files:

```javascript
async function safeLoadStory(filepath) {
  try {
    const html = await readFile(filepath, 'utf8');
    const story = Story.fromHTML(html);
    return story;
  } catch (error) {
    console.error('Failed to load story:', error.message);
    return null;
  }
}
```

### 4. Use Twee Version 3 for Full Fidelity

When converting to/from Twee, use version 3 to preserve all metadata:

```javascript
// Preserves all attributes
const twee = story.toTwee(3);

// Load with full metadata support
const story = Story.fromTwee(tweeSource, 3);
```

### 5. Organize Passages with Tags

Use tags to categorize and filter passages:

```javascript
// Add tags when creating passages
const passage = new Passage({
  attributes: {
    name: 'BossFight',
    tags: ['combat', 'boss', 'chapter3']
  },
  source: '...'
});

// Filter passages by tag
const combatPassages = story.passages.filter(
  p => p.attributes.tags?.includes('combat')
);

console.log(`Found ${combatPassages.length} combat passages`);
```

### 6. Merge Stories Carefully

When merging stories, be aware of duplicate passage names:

```javascript
function safeMergeStories(main, additional) {
  // Check for conflicts
  const mainNames = new Set(main.passages.map(p => p.attributes.name));
  const conflicts = additional.passages.filter(
    p => mainNames.has(p.attributes.name)
  );

  if (conflicts.length > 0) {
    console.warn('Warning: Duplicate passage names found:');
    conflicts.forEach(p => console.warn(`  - ${p.attributes.name}`));
  }

  main.mergeStory(additional);
  return main;
}
```

### 7. Validate Stories Before Publishing

Check for common issues before publishing:

```javascript
function validateStory(story) {
  const issues = [];

  // Check for story name
  if (!story.attributes.name) {
    issues.push('Story has no name');
  }

  // Check for passages
  if (story.passages.length === 0) {
    issues.push('Story has no passages');
  }

  // Check for start passage
  if (!story.startPassage) {
    issues.push('Story has no start passage');
  }

  // Check for passages without names
  const unnamed = story.passages.filter(p => !p.attributes.name);
  if (unnamed.length > 0) {
    issues.push(`${unnamed.length} passages have no name`);
  }

  // Check for empty passages
  const empty = story.passages.filter(p => !p.source || p.source.trim() === '');
  if (empty.length > 0) {
    issues.push(`${empty.length} passages are empty`);
  }

  return issues;
}

// Usage
const issues = validateStory(myStory);
if (issues.length > 0) {
  console.error('Story validation failed:');
  issues.forEach(issue => console.error(`  - ${issue}`));
} else {
  console.log('Story validation passed!');
}
```

### 8. Optimize Published HTML Size

For web distribution, consider minifying output:

```javascript
async function publishOptimized(story, format) {
  const html = format.publish(story);
  
  // Basic optimization (remove extra whitespace)
  const optimized = html
    .replace(/\n\s+/g, '\n')
    .replace(/>\s+</g, '><')
    .trim();

  console.log(`Original: ${(html.length / 1024).toFixed(2)} KB`);
  console.log(`Optimized: ${(optimized.length / 1024).toFixed(2)} KB`);
  console.log(`Saved: ${(((html.length - optimized.length) / html.length) * 100).toFixed(1)}%`);

  return optimized;
}
```

### 9. Use Position Attributes for Large Stories

For better organization in Twine editors, set passage positions:

```javascript
// Grid layout
function createGridLayout(passages, cols = 5, spacing = 200) {
  passages.forEach((passage, index) => {
    const x = (index % cols) * spacing + 100;
    const y = Math.floor(index / cols) * spacing + 100;
    passage.attributes.position = `${x},${y}`;
  });
}

createGridLayout(story.passages);
```

### 10. Document Your Story Structure

Add comments or special passages for documentation:

```javascript
// Add a documentation passage
story.passages.push(new Passage({
  attributes: {
    name: 'Documentation',
    tags: ['documentation']
  },
  source: `
# Story Structure

## Main Path
Start -> Chapter1 -> Chapter2 -> Ending

## Side Quests
- Quest1 (accessed from Chapter1)
- Quest2 (accessed from Chapter2)

## Special Passages
- GameOver (failure state)
- Credits (after ending)
  `.trim()
}));
```

## Conclusion

This API documentation covers all major functions and classes in twine-utils. The library provides powerful tools for:

- **Creating** stories from scratch
- **Loading** stories from multiple formats (HTML, Twee, TWS)
- **Modifying** existing stories programmatically
- **Publishing** stories to playable HTML
- **Converting** between different formats
- **Automating** story generation and processing

For more information and updates, visit the [GitHub repository](https://github.com/klembot/twine-utils) or check the [TypeDoc API documentation](https://klembot.github.io/twine-utils/).

## Additional Resources

- [Twine Official Site](https://twinery.org)
- [Twee 3 Specification](https://github.com/iftechfoundation/twine-specs/blob/master/twee-3-specification.md)
- [Twine Story Format Documentation](https://github.com/klembot/twinejs/blob/develop/EXTENDING.md)
- [Interactive Fiction Community](https://intfiction.org)
