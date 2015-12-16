<pre>
     ___        __                               __             ___      
    /\_ \    __/\ \                             /\ \__         /\_ \     
    \//\ \  /\_\ \ \____  _____     ___     ____\ \ ,_\    __  \//\ \    
      \ \ \ \/\ \ \ '__`\/\ '__`\  / __`\  /',__\\ \ \/  /'__`\  \ \ \   
       \_\ \_\ \ \ \ \L\ \ \ \L\ \/\ \L\ \/\__, `\\ \ \_/\ \L\.\_ \_\ \_ 
       /\____\\ \_\ \_,__/\ \ ,__/\ \____/\/\____/ \ \__\ \__/.\_\/\____\
       \/____/ \/_/\/___/  \ \ \/  \/___/  \/___/   \/__/\/__/\/_/\/____/
                            \ \_\                                        
                             \/_/                                        
    ---------------------------------------------------------------------
</pre>

**N.B.**: libpostal is not publicly released yet and the APIs may change. We
encourage folks to hold off on including it as a dependency for now.
Stay tuned...

---------------------------------------------------------------------------

:jp: :us: :gb: :ru: :fr: :kr: :it: :es: :cn: :de:

libpostal is a fast, multilingual, all-i18n-everything NLP library for 
normalizing and parsing physical addresses.

Addresses and the geographic coordinates they represent are essential for any
location-based application (map search, transportation, on-demand/delivery
services, check-ins, reviews). Yet even the simplest addresses are packed with
local conventions, abbreviations and context, making them difficult to
index/query effectively with traditional full-text search engines, which are
designed for document indexing. This library helps convert the free-form
addresses that humans use into clean normalized forms suitable for machine
comparison and full-text indexing.

libpostal is not itself a full geocoder, but should be a ubiquitous
preprocessing step before indexing/searching with free text geographic strings.
It is written in C for maximum portability and performance.

Examples of normalization
-------------------------

Address normalization may sound trivial initially, especially when thinking
only about the US (if that's where you happen to reside), but it only takes
a few examples to realize how complicated natural language addresses are
internationally. Here's a short list of some less straightforward normalizations
in various languages. The left/right columns in this table are equivalent
strings under libpostal, the left column being user input and the right column
being the indexed (normalized) string.

| Input                               | Output (may be multiple in libpostal) |
| ----------------------------------- |---------------------------------------|
| One-hundred twenty E 96th St        | 120 east 96th street                  |
| C/ Ocho, P.I. 4                     | calle 8 polígono industrial 4         |
| V XX Settembre, 20                  | via 20 settembre 20                   |
| Quatre vignt douze R. de l'Église   | 92 rue de l' église                   |
| ул Каретный Ряд, д 4, строение 7    | улица каретныи ряд дом 4 строение 7   |
| ул Каретный Ряд, д 4, строение 7    | ulica karetnyj rad dom 4 stroenie 7   |
| Marktstrasse 14                     | markt straße 14                       |

libpostal currently supports these types of normalization in *over 60 languages*,
and you can add more (without having to write any C!)

Now, instead of trying to bake address-specific conventions into traditional
document search engines like Elasticsearch using giant synonyms files, scripting,
custom analyzers, tokenizers, and the like, geocoding can be as simple as:

1. Run the addresses in your index through libpostal's expand_address
2. Store the normalized string(s) in your favorite search engine, DB, 
   hashtable, etc.
3. Run your user queries or fresh imports through libpostal and search
   the existing database using those strings

In this way, libpostal can perform fuzzy address matching in constant time.

For further reading and some bizarre address edge-cases, see:
[Falsehoods Programmers Believe About Addresses](https://www.mjt.me.uk/posts/falsehoods-programmers-believe-about-addresses/).

Examples of parsing
-------------------

libpostal implements the first truly international statistical address parser,
trained on ~50 million addresses in over 100 countries speaking over 60
languages. We use OpenStreetMap (anything with an addr:* tag) and the OpenCage
address format templates at: https://github.com/OpenCageData/address-formatting
to construct the training data, supplementing with containing polygons and
perturbing the inputs in a number of ways to make the parser as robust as possible
to messy real-world input.

These example parses are taken from the interactive address_parser program 
that builds with libpostal on make. Note that the parser doesn't care about commas
vs. no commas, casing, or different permutations of components (if components are
left out e.g. just city or just city/postcode).

```
> 781 Franklin Ave Crown Heights Brooklyn NYC NY 11216 USA 

Result:
{
  "house_number": "781"
  "road": "franklin ave"
  "suburb": "crown heights"
  "city_district": "brooklyn"
  "city": "nyc"
  "state": "ny"
  "postcode": "11216"
  "country": "usa",
} 

> The Book Club 100-106 Leonard St, Shoreditch, London, Greater London, England, EC2A 4RH, United Kingdom
 
Result: 
 
{
  "house": "the book club"
  "house_number": "100-106"
  "road": "leonard st"
  "suburb": "shoreditch"
  "city": "london"
  "state_district": "greater london"
  "state": "england"
  "postcode": "ec2a 4rh"
  "country": "united kingdom",
}
 
> Eschenbräu Bräurei Triftstraße 67, 13353 Berlin, Deutschland
 
Result:

{
  "house": "eschenbraeu braeurei"
  "road": "triftstrasse"
  "house_number": "67"
  "postcode": "13353"
  "city": "berlin"
  "country": "deutschland",
}

 > Double Shot Tea & Coffee 15 Melle St. Braamfontein Johannesburg, 2000, South Africa
 
Result:

{
  "house": "double shot tea & coffee"
  "house_number": "15"
  "road": "melle st."
  "suburb": "braamfontein"
  "city": "johannesburg"
  "postcode": "2000"
  "country": "south africa",
}
 
> Le Polikarpov 24 cours Honoré d'Estienne d'Orves, 13001 Marseille, France  
 
Result:

{
  "house": "le polikarpov"
  "house_number": "24"
  "road": "cours honoré d'estienne d'orves"
  "postcode": "13001"
  "city": "marseille"
  "country": "france",
}
 
> Государственный Эрмитаж Дворцовая наб., 34 191186, Saint Petersburg, Russia 

Result:

{
  "house": "государственный эрмитаж"
  "road": "дворцовая наб."
  "house_number": "34"
  "postcode": "191186"
  "city": "saint petersburg"
  "country": "russia",
}
```

The parser achieves very high accuracy on held-out data, currently 98.9%
correct full parses (meaning a 1 in the numerator for getting *every* token
in the address correct).

Installation
------------

Before you install, make sure you have the following prerequisites:

**On Linux (Debian)**
```
sudo apt-get install libsnappy-dev autoconf automake libtool
```

**On Mac OSX**
```
sudo brew install snappy autoconf automake libtool
```

For C/C++ users or those writing bindings (if you've written a
language binding, please let us know!):

```
git clone https://github.com/openvenues/libpostal
cd libpostal
./bootstrap.sh
./configure --datadir=[...some dir with a few GB of space...]
make
sudo make install

# On Linux it's probably a good idea to run
sudo ldconfig
```

To install via Python, you should first install the C library and then run:

```
python setup.py install
```

Python usage
------------

After installing:

```python
from postal.expand import expand_address
expand_address('Quatre vignt douze Ave des Champs-Élysées', languages=['fr'])

from postal.parser import parse_address
parse_address('The Book Club 100-106 Leonard St, Shoreditch, London, Greater London, EC2A 4RH, United Kingdom')
```

**Note**: for expand_address, we currently default to English if no languages parameter is passed. When the language classifier is complete we'll remove this requirement and libpostal will predict the language automatically if none is specified.


Command-line usage (expand)
---------------------------

After building libpostal:

```
cd src/

./libpostal "12 Three-hundred and forty-fifth ave, ste. no 678" en
```

Currently libpostal requires two input strings, the address text and a language
code (ISO 639-1).

Command-line usage (parser)
---------------------------

After building libpostal:

```
cd src/

./address_parser
```

address_parser is an interactive shell. Just type addresses and libpostal will
parse them and print the result.

Data files
----------

libpostal needs to download some data files from S3. The basic files are on-disk
representations of the data structures necessary to perform expansion. For address
parsing, since model training takes about a day, we publish the fully trained model 
to S3 and will update it automatically as new addresses get added to OSM.

Data files are automatically downloaded when you run make. To check for and download
any new data files, run:

```
libpostal_data download all $YOUR_DATA_DIR/libpostal
```

And replace $YOUR_DATA_DIR with whatever you passed to configure during install.

Features
--------

- **Abbreviation expansion**: e.g. expanding "rd" => "road" but for almost any
language. libpostal supports > 50 languages and it's easy to add new languages
or expand the current dictionaries. Ideographic languages (not separated by
whitespace e.g. Chinese) are supported, as are Germanic languages where
thoroughfare types are concatenated onto the end of the string, and may
optionally be separated so Rosenstraße and Rosen Straße are equivalent.

- **International address parsing**: sequence model which parses
"123 Main Street New York New York" into {"house_number": 123, "road":
"Main Street", "city": "New York", "state": "New York"}. Unlike the majority
of parsers out there, it works for a wide variety of countries and languages,
not just US/English. The model is trained on > 50M OSM addresses, using the
templates in the [OpenCage address formatting repo](https://github.com/OpenCageData/address-formatting) to construct formatted,
tagged traning examples for most countries around the world.

- **Language classification (coming soon)**: multinomial logistic regression
trained on all of OpenStreetMap ways, addr:* tags, toponyms and formatted
addresses. Labels are derived using point-in-polygon tests in Quattroshapes
and official/regional languages for countries and admin 1 boundaries
respectively. So, for example, Spanish is the default language in Spain but
in different regions e.g. Catalunya, Galicia, the Basque region, regional
languages are the default. Dictionary-based disambiguation is employed in
cases where the regional language is non-default e.g. Welsh, Breton, Occitan.

- **Numeric expression parsing** ("twenty first" => 21st, 
"quatre-vignt-douze" => 92, again using data provided in CLDR), supports > 30
languages. Handles languages with concatenated expressions e.g.
milleottocento => 1800. Optionally normalizes Roman numerals regardless of the
language (IX => 9) which occur in the names of many monarchs, popes, etc.

- **Geographic name aliasing**: New York, NYC and Nueva York alias to New York
City. Uses the crowd-sourced GeoNames (geonames.org) database, so alternate
names added by contributors can automatically improve libpostal.

- **Geographic disambiguation (coming soon)**: There are several equally
likely Springfields in the US (formally known as The Simpsons problem), and
some context like a state is required to disambiguate. There are also > 1200
distinct San Franciscos in the world but the term "San Francisco" almost always
refers to the one in California. Williamsburg can refer to a neighborhood in
Brooklyn or a city in Virginia. Geo disambiguation is a subset of Word Sense
Disambiguation, and attempts to resolve place names in a string to GeoNames
entities. This can be useful for city-level geocoding suitable for polygon/area
lookup. By default, if there is no other context, as in the San Francisco case,
the most populous entity will be selected.

- **Ambiguous token classification (coming soon)**: e.g. "dr" => "doctor" or
"drive" for an English address depending on the context. Multiclass logistic
regression trained on OSM addresses, where abbreviations are discouraged,
giving us many examples of fully qualified addresses on which to train.

- **Fast, accurate tokenization/lexing**: clocked at > 1M tokens / sec,
implements the TR-29 spec for UTF8 word segmentation, tokenizes East Asian
languages chracter by character instead of on whitespace.

- **UTF8 normalization**: optionally decompose UTF8 to NFD normalization form,
strips accent marks e.g. à => a and/or applies Latin-ASCII transliteration.

- **Transliteration**: e.g. улица => ulica or ulitsa. Uses all
[CLDR transforms](http://www.unicode.org/repos/cldr/trunk/common/transforms/), the exact same as used by ICU,
though libpostal doesn't require pulling in all of ICU (might conflict 
with your system's version). Note: some languages, particularly Hebrew, Arabic
and Thai may not include vowels and thus will not often match a transliteration 
done by a human. It may be possible to implement statistical transliterators
for some of these languages.

- **Script detection**: Detects which script a given string uses (can be
multiple e.g. a free-form Hong Kong or Macau address may use both Han and
Latin scripts in the same address). In transliteration we can use all
applicable transliterators for a given Unicode script (Greek can for instance
be transliterated with Greek-Latin, Greek-Latin-BGN and Greek-Latin-UNGEGN).

Non-goals
---------

- Verifying that a location is a valid address
- Street-level geocoding

Raison d'être
-------------

libpostal was created as part of the [OpenVenues](https://github.com/openvenues/openvenues) project to solve 
the problem of venue deduping. In OpenVenues, we have a data set of millions of
places derived from terabytes of web pages from the [Common Crawl](http://commoncrawl.org/).
The Common Crawl is published monthly, and so even merging the results of
two crawls produces significant duplicates.

Deduping is a relatively well-studied field, and for text documents like web
pages, academic papers, etc. there exist pretty decent approximate
similarity methods such as [MinHash](https://en.wikipedia.org/wiki/MinHash). 

However, for physical addresses, the frequent use of conventional abbreviations
such as Road == Rd, California == CA, or New York City == NYC complicates
matters a bit. Even using a technique like MinHash, which is well suited for
approximate matches and is equivalent to the Jaccard similarity of two sets, we
have to work with very short texts and it's often the case that two equivalent
addresses, one abbreviated and one fully specified, will not match very closely
in terms of n-gram set overlap. In non-Latin scripts, say a Russian address and
its transliterated equivalent, it's conceivable that two addresses referring to
the same place may not match even a single character.

As a motivating example, consider the following two equivalent ways to write a
particular Manhattan street address with varying conventions and degrees
of verbosity:

- 30 W 26th St Fl #7
- 30 West Twenty-sixth Street Floor Number 7

Obviously '30 W 26th St Fl #7 != '30 West Twenty-sixth Street Floor Number 7'
in a string comparison sense, but a human can grok that these two addresses
refer to the same physical location.

libpostal aims to create normalized geographic strings, parsed into components,
such that we can more effectively reason about how well two addresses
actually match and make automated server-side decisions about dupes.

So it's not a geocoder?
-----------------------

If the above sounds a lot like geocoding, that's because it is in a way,
only in the OpenVenues case, we do it without a UI or a user to select the
correct address in an autocomplete. Given a database of source addresses
such as OpenAddresses or OpenStreetMap (or all of the above), libpostal
can be used to implement things like address deduping and server-side
batch geocoding in settings like MapReduce.

Why C?
------

libpostal is written in C for three reasons (in order of importance):

1. **Portability/ubiquity**: libpostal targets higher-level languages that
people actually use day-to-day: Python, Go, Ruby, NodeJS, etc. The beauty of C
is that just about any programming language can bind to it and C compilers are
everywhere, so pick your favorite, write a binding, and you can use libpostal
directly in your application without having to stand up a separate server. We
support Mac/Linux (Windows is not a priority but happy to accept patches), have
a standard autotools build and an endianness-agnostic file format for the data
files. The Python bindings, are maintained as part of this repo since they're
needed to construct the training data.

2. **Memory-efficiency**: libpostal is designed to run in a MapReduce setting
where we may be limited to < 1GB of RAM per process depending on the machine
configuration. As much as possible libpostal uses contiguous arrays, tries
(built on contiguous arrays), bloom filters and compressed sparse matrices to
keep memory usage low. It's conceivable that libpostal could even be used on
a mobile device, although that's not an explicit goal of the project.

3. **Performance**: this is last on the list for a reason. Most of the
optimizations in libpostal are for memory usage rather than performance.
libpostal is quite fast given the amount of work it does. It can process
10-30k addresses / second in a single thread/process on the platforms we've
tested (that means processing every address in OSM planet in a little over
an hour). Check out the simple benchmark program to test on your environment
and various types of input. In the MapReduce setting, per-core performance
isn't as important because everything's being done in parallel, but there are
some streaming ingestion applications at Mapzen where this needs to
run in-process.

C codebase
----------

libpostal is written in modern, legible, C99 and uses the following conventions:

- Roughly object-oriented, as much as allowed by C
- Almost no pointer-based data structures, arrays all the way down
- Uses dynamic character arrays (inspired by [sds](https://github.com/antirez/sds)) for safer string handling
- Confines almost all mallocs to *name*_new and all frees to *name*_destroy
- Efficient existing implementations for simple things like hashtables
- Generic containers (via [klib](https://github.com/attractivechaos/klib)) whenever possible
- Data structrues take advantage of sparsity as much as possible
- Efficient double-array trie implementation for most string dictionaries
- Tries to stay cross-platform as much as possible, particularly for *nix

Python codebase
---------------

There are actually two Python packages in libpostal.

1. **geodata**: generates C files and data sets used in the C build
2. **pypostal**: Python bindings for libpostal

geodata is simply a confederation of scripts for preprocessing the various geo
data sets and building input files for the C lib to use during model training.
Said scripts shouldn't be needed  for most users unless you're rebuilding data
files for the C lib.

Language dictionaries
---------------------

It's easy to add new languages/synonyms to libpostal by modifying a few text
files. The format of each dictionary file roughly resembles a
Lucene/Elasticsearch synonyms file:

```
drive|dr
street|st|str
road|rd
```

The leftmost string is treated as the canonical/normalized version. Synonyms
if any, are appended to the right, delimited by the pipe character.

The supported languages can be found in the [resources/dictionaries](https://github.com/openvenues/libpostal/tree/master/resources/dictionaries).

Each language can define one or more dictionaries (sometimes called "gazetteers" in NLP) to help with address parsing, and normalizing abbreviations. The dictionary types are:

- **academic_degrees.txt**: for post-nominal strings like "M.D.", "Ph.D.", etc.
- **ambiguous_expansions.txt**: e.g. "E" could be expanded to "East" or could
be "E Street", so if the string it encountered, it can either be left alone or expanded
- **building_types.txt**: strings indicating a building/house
- **company_types.txt**: company suffixes like "Inc" or "GmbH"
- **concatenated_prefixes_separable.txt**: things like "Hinter..." which can
be written either concatenated or as separate tokens
- **concatenated_suffixes_inseparable.txt**: Things like "...bg." => "...burg"
where the suffix cannot be separated from the main token, but either has an
abbreviated equivalent or simply can help identify the token in parsing as,
say, part of a street name
- **concatenated_suffixes_separable.txt**: Things like "...straße" where the
suffix can be either concatenated to the main token or separated
- **directionals.txt**: strings indicating directions (cardinal and
lower/central/upper, etc.)
- **level_types.txt**: strings indicating a particular floor
- **no_number.txt**: strings like "no fixed address"
- **nulls.txt**: strings meaning "not applicable"
- **personal_suffixes.txt**: post-nominal suffixes, usually generational
like Jr/Sr
- **personal_titles.txt**: civilian, royal and military titles
- **place_names.txt**: strings found in names of places e.g. "theatre",
"aquarium", "restaurant". See [Nominatim Special Phrases](http://wiki.openstreetmap.org/wiki/Nominatim/Special_Phrases)
- **post_office.txt**: strings like "p.o. box"
- **qualifiers.txt**: strings like "township"
- **stopwords.txt**: prepositions and articles mostly, very common words
which may be ignored in some contexts
- **street_types.txt**: words like "street", "road", "drive" which indicate
a thoroughfare and their respective abbreviations.
- **synonyms.txt**: any miscellaneous synonyms/abbreviations e.g. "bros"
expands to "brothers", etc. These have no special meaning and will essentially
just be treated as string replacement.
- **toponyms.txt**: abbreviations for certain abbreviations relating to
toponyms like regions, places, etc. Note: GeoNames covers most of these.
In most cases better to leave these alone
- **unit_types.txt**: strings indicating an apartment or unit number

Most of the dictionaries have been derived with the following process:

1. Tokenize every street name in OSM for language x
2. Count the most common N tokens
3. Optionally use frequent item set techniques to exctract phrases
4. Run the most frequent words/phrases through Google Translate
5. Add the ones that mean "street" to dictionaries
6. Augment by researching addresses in countries speaking language x

In the future it might be beneficial to move the dictionaries to a wiki
so they can be crowdsourced by native speakers regardless of whether or not
they use git.

Address parser accuracy
-----------------------

On held-out test data (meaning labeled parses that the model has _not_ seen
before), the address parser achieves 98.9% full parse accuracy.

For some tasks like named entity recognition it's preferable to use something
like an F1 score or variants, mostly because there's a class bias problem (most
tokens are non-entities, and a system that simply predicted non-entity for
every token would actually do fairly well in terms of accuracy). That is not
the case for address parsing. Every token has a label and there are millions
of examples of each class in the training data, so accuracy 

We prefer to evaluate on full parses (at the sentence level in NER nomenclature),
so that means that 98.9% of the time, the address parser gets every single token
in the address correct, which is quite good performance.

Improving the address parser
----------------------------

There are four primary ways the address parser can be improved even further
(in order of difficulty):

1. Contribute addresses to OSM. Anything with an addr:housenumber tag will be
   incorporated automatically into the parser next time it's trained.
2. If the address parser isn't working well for a particular country, language
   or style of address, chances are that some name variations or places being
   missed/mislabeled during training data creation. Sometimes the fix is to
   add more countries at: https://github.com/OpenCageData/address-formatting,
   and in many other cases there are relatively simple tweaks we can make
   when creating the training data that will ensure the model is trained to
   handle your use case without you having to do any manual data entry.
   If you see a pattern of obviously bad address parses, post an issue to
   Github and we'll tr
3. We currently don't have training data for things like flat numbers.
   The tags are fairly uncommon in OSM and the address-formatting templates
   don't use floor, level, apartment/flat number, etc. This would be a slightly
   more involved effort, but would be like to begin a discussion around it.
4. We use a greedy averaged perceptron for the parser model. Viterbi inference
   using a linear-chain CRF may improve parser performance on certain classes
   of input since the score is the argmax over the entire label sequence not
   just the token. This may slow down training significantly.

Todos
-----

[ ] Port language classification from Python, train and publish model
[ ] Publish tests (currently not on Github) and set up continuous integration
[ ] Hosted documentation
