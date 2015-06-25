- title : Build Yourself A Google
- description : How to find stuff quickly in a big pile of text
- author : Daniel Bradley
- theme : serif
- transition : default

***

## Build Yourself A Google

![DIY](images/meme-diy.jpg)

***

## A brief history of finding stuff on the internet

---

### 1995 - AltaVista

- First full-text search service for the internet.
- Information was gathered by crawlers.
- Ran on 20 machines, each had:
  - 130GB of RAM
  - 500GB of storage
- *10 terrabytes total storage!*

---

### 1997 - Ask Jeeves

- Ordered results by popularity

---

### 1998 - Google
- "PC Magazine" reports that Google *"has an uncanny knack for returning extremely relevant results"*
- **Page rank** - ranking by factors such as number of links from other pages.

---

### Google grows
- 10 years on - 2008:
 - 1 trillion pages indexed
- Today:
 - 30 trillion
 - 30,000,000,000,000

<br><br>
<small>Still got 87 zeros to add until they reach a googol</small>

---

## Differentiators
Ranking

## Commonalities
Find information quickly and efficiently

---

Let's focus on commonalities that make search possible

All this material revolves around the Apache Lucene library.

![Lucene](https://lucene.apache.org/images/lucene_logo_green_300.png)

***

<small>revised title</small>

## 'How to find stuff quickly in a big pile of text'

---

## What's the problem
- Why can't you just do `ctrl`+`F`?

---

![Will it scale](images/will-it-scale.jpg)

---

## What if we ...

#### Double the text in each document?
#### Double number of documents?
#### Double number of users?

---

![Blender explosion](images/blender-explode.gif)

***

## What's the alternative?

*We need to 'invert the index'*

---

### 3 documents with 1 sentence in each

    [lang=js]
    var documents = {
      "docA": "it is what it is",
      "docB": "what is it",
      "docC": "it is a banana"
    };

---

### List the words in each document

    [lang=js]
    var documents = {
      "docA": "it is what it is",
      "docB": "what is it",
      "docC": "it is a banana"
    };

    var wordsInDocuments = {
      "docA": ["it", "is", "what", "it", "is"],
      "docB": ["what", "is", "it"],
      "docC": ["it", "is", "a", "banana"]
    };

' Tokenisation

---

### Invert the lookup

    [lang=js]
    var wordsInDocuments = {
      "docA": ["it", "is", "what", "it", "is"],
      "docB": ["what", "is", "it"],
      "docC": ["it", "is", "a", "banana"]
    };

    var documentsWithWords = {
      "a": ["docC"],
      "banana": ["docC"],
      "is": ["docA", "docB", "docC"],
      "it": ["docA", "docB", "docC"],
      "what": ["docA", "docB"]
    };

***

### Now we've finished the intro...

' In the real world, there's a bit more to it than just words

***

### Querying - Part 1

    [lang=js]
    var index = {
      "a": ["docC"],
      "banana": ["docC"],
      "is": ["docA", "docB", "docC"],
      "it": ["docA", "docB", "docC"],
      "what": ["docA", "docB"]
    };

Which documents match the following searches:

1. "banana"
2. "what AND banana"
3. "what OR banana"
4. "it AND NOT what"

' Boolean operators

***

## Fields
A document might be more than a single piece of text e.g.

    [lang=js]
    var email = {
      from: "bob@hotmail.com",
      to: "dan@gmail.com",
      sentAt: "2015-06-25T16:52:35Z"
      subject: "re: that thing"
      body: "What's up?"
    };

' -

    [lang=js]
    var event = {
      start: "2015-07-03T10:30:00Z",
      end: "2015-07-03T11:30:00Z"
      location: "The Angel",
      attendees: "All of FundApps",
      subject: "Build Yourself A Google"
    };

---

## What do we do with our inverted index?

---

We create an index per field

    [lang=js]
    var index = {
      "body": {
        "a": ["docC"],
        "banana": ["docC"],
        "is": ["docA", "docB", "docC"],
        "it": ["docA", "docB", "docC"],
        "what": ["docA", "docB"]
      },
      subject: {
        "a": ["docA"],
        "build": ["docA"],
        "google": ["docA"],
        "yourself": ["docA"]
      }
    };

***

<small><s>Tolkienisation</s></small>

## Tokenisation

![gollum](images/gollum.png)

' Turning a single big blob of text
' into lots of small blobs

---

How do we extract words?

## Start simple

Just split on whitespace (spaces and tabs)

---

## What about punctuation?

*"He finally answered (after taking five minutes to think) that he didn't understand the question."*

---

### What about other languages?

*"구미의 남서쪽에 있는 금오산도립공원은 영남 팔경의 하나이다."*

*"不过，影评人对电影的剧情褒贬不一，他们的批评认为，可以预见到电影不会对其所提出的问题加以解决。"*

' There are general purpose & per-language

---

### Special cases

e.g. Path hierarchy

`"/home/foo/bar"`

⇩

 `["/home/foo/bar", "/home/foo", "/home", "/"]`

' Email addresses
' URLs

***

## Analysis (transformation)

`Tokenisation` ⇨ `Analysis`

' Taking the small blobs and changing them

---

## Casing

`SomeThing` ⇨ `something`

' Need to change what we search for

---

## Stop words

    [lang=js]
    var before = ["the", "banana", "is", "yellow"];
    var after = ["banana", "yellow"];

---

## Stemming

> **Stemming** is the term used in linguistic morphology and information retrieval to describe the process for reducing inflected (or sometimes derived) words to their word stem, base or root form—generally a written word form.

---

### Stemming (cont.)

#### Normalise endings

- `walked` ⇨ `walk`
- `talking` ⇨ `talk`
- `ran`/`run` ⇨ ...

![I'm sorry Dave, I'm afraid I can't do that](http://www.king-royal-design.com/wp-content/uploads/2011/10/i-m-sorry-dave-e1369058177230.jpg)

' Not easy.
' Porter stemmer (Martin Porter '80)
' Snowball language

---

## Soundex

W300, C640, S512, B634, P362, S534

N350, B200, M635, S530, P252, M236

' Remove vouls, y, h, w
' Replace similar letters with a number (d|t=3, m|n=5)
' Remove duplicates

***

## Querying Part 2

---

### Relevance

- Rank document higher with more matches

' Maybe we should count how many times each word is used in each document?

---

### More uses for fields

- Hidden from users
- Can duplicate information

---

### Fields example

`something`

⇩

`original:something OR stemmed:someth OR soundex:S535`

***

### Scoring & Boosting

---

### Position Offsets
