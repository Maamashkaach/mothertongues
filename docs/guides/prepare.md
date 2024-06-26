---
comments: true
---

# Preparing your data

*This guide is required for anyone creating a Mother Tongues Dictionary, regardless of which path they take ([no code](../nocode/install.md) or [advanced developer](../developer/prerequisites.md)).*

The most time-intensive part of building your dictionary, depending on how you want to publish it, is preparing your data.

## Project folder structure

To generate a sample project that you can edit to suit your needs, use the `mothertongues new-project` command. For example `mothertongues new-project --outdir . --name 'DanishDictionary'` will build a folder called 'DanishDictionary in the current directory'. It will include some sample data ('data.xlsx') and a [MTD configuration file](#mtd-language-configuration-file) ('config.mtd.json').

```raw
📦DanishDictionary
 ┣ 📜data.xlsx
 ┗ 📜config.mtd.json
```

## MTD Configuration file

!!! tip
    In the [starter](https://github.com/MotherTongues/mtd-starter), the language configuration file is found here: `mtd-starter/config.mtd.json`


Every dictionary must have a configuration file, which helps define which data belongs to your dictionary and meta data like your dictionary's [alphabet](#customized-alphabet)


## Sorting your data

Your data will get sorted based on your defined [alphabet](#customized-alphabet). By default, the sorting form is created from the "word" key from each entry and characters that aren't defined in your alphabet will be sorted at the end. If you want ignore certain characters from being sorted, or sort using different data, you can change this in the configuration:

```json hl_lines="7-8"
{
    "config":{
        "L1": "Danish",
        "L2": "English",
        ...
        "alphabet": ["a", "b", "c"],
        "no_sort_characters": [",", "(", ")"],
        "sorting_field": "word",
    },
    ...
}
```

### Customized Alphabet

Adding your custom alphabet allows your entries to be sorted based on that alphabet. If you don't use a custom alphabet, the English alphabet will be used instead.

In your Language Configuration file, set the `alphabet`
key equal to an array containing the letters in your language's alphabet in alphabetical order:

```json hl_lines="6"
{
    "config":{
        "L1": "Danish",
        "L2": "English",
        ...
        "alphabet": ["a", "b", "c"]
    },
    ...
}
```

You can also reference a file that contains a csv of your alphabet:

```json hl_lines="6"
{
    "config":{
        "L1": "Danish",
        "L2": "English",
        ...
        "alphabet": "../alphabet.csv"
    },
    ...
}
```

!!! tip
    In the [starter](https://github.com/MotherTongues/mtd-starter), an alphabet file is used at `mtd-starter/alphabet.csv`

### Custom Sort Form (Advanced)

It's possible that you want to sort based on a form other than the `word` or display form by changing the word in some way. To do this, you must adjust the [configuration for your data](#parsing-your-data) to include a transducer. In this example, we are adding a `sort_form` which is just a lowercased version of the `word` and then defining that as the field to sort. You can apply any of the operations from the "RestrictedTransducer" class if you need to do more than just lowercase. See [here](./search.md#index-changing-which-fields-get-searched) for more details.


```json hl_lines="5 13-18"
{
  "config": {
    "L1": "Danish",
    "L2": "English",
    "sorting_field": "sort_form",
    ...
  },
  "data": [
    {
      "manifest": {
        "file_type": "xlsx",
        "name": "YourData",
        "transducers": [
          {
            "input_field": "word",
            "output_field": "sort_form",
            "lower": true
          }
        ],
        "targets": {
          "word": "A",
          "definition": "B"
        }
      },
      "resource": "data.xlsx"
    }
  ]
}
```

## Parsing your data

Every source of data must have a configuration, which defines how the data should be parsed.

This configuration file describes a spreadsheet (data.xlsx) dictionary resource that only has two columns where the first column includes the word in the target L1 language and the second column includes the 'definition' of that word or 'gloss' in the L2 language. The "manifest" describes how to parse the resource.

```json
{
    "manifest": {
        "file_type": "xlsx",
        "name": "YourData",
        "targets": {
            "word": "A",
            "definition": "B",
        }
    },
    "resource": "data.xlsx"
}
```

### Optional Information

To add information that can be optionally displayed in the UI, you must
point to it in your Data Resource configuration file. For example, if
you wanted to add "Part of Speech" information that could be displayed
optionally and that was present in column "F" of an Excel spreadsheet, you
would add the following to your Data Resource configuration file:


```json hl_lines="7-9"
{
    ...
    "name": "2018 Spreadsheet",
    "targets": {
        "word": "A",
        "definition": "B",
         "optional": {
                "Part of Speech": "F"
            }
    }
}
```

### Multimedia

To add images and audio, you must have the filenames of your files in
your dictionary data resource. Then, change your Data Resource
configuration files to point to the location of the filenames.

#### Images

For images, just add a target for the `img` key. Take the following
example for an Excel spreadsheet with image filenames in column "D":


```json hl_lines="10"
{
    "manifest": {
        "file_type": "xlsx",
        "name": "YourData",
        "skip_header": false,
        "transducers": [],
        "targets": {
            "word": "A",
            "definition": "B",
            "img": "D"
        }
    },
    "resource": "data.xlsx"
}
```

#### Audio

For audio, you minimally have to add the filename, but you can also add
a description for the audio (such as a speaker name). You can also choose between `audio` for audio files in
the target language, `definition_audio` for audio files of the
definition, `example_sentence_audio` for audio files corresponding to an
example sentence and `example_sentence_definition_audio` for audio files
corresponding to the definitions of example sentences.

Take the following example for an Excel spreadsheet with audio in
columns "B" & "C" and example sentence audio in column "D". The
descriptions for audio files are in columns "E", "F", and "G"
respectively.

```json hl_lines="10-27"
{
    "manifest": {
        "file_type": "xlsx",
        "name": "YourData",
        "skip_header": false,
        "transducers": [],
        "targets": {
            "word": "A",
            "definition": "H",
            "audio": [
                {
                    "filename": "B",
                    "description": "E"
                },
                {
                    "filename": "C",
                    "description": "F"
                }
            ],
            "example_sentence_audio": [
                [
                    {
                        "filename": "D",
                        "description": "G"
                    }
                ]
            ]
        }
    },
    "resource": "data.xlsx"
}
```

### Semantic Categories

To add semantic categories to your entries, you can make use of both the
`theme` and `secondary_theme` keys in the Data Resource configuration
file. Using these will allow your entries to be sorted based on semantic
categories like "colours", or "animals" etc.

For example, suppose you have an Excel spreadsheet where column "C"
has main categories like "Animals", and column "D" has
sub-categories like "- Fish", and "- Reptiles". Your Data Resource
congfiguration file would have to add the following targets:

```json hl_lines="10-11"
{
    "manifest": {
        "file_type": "xlsx",
        "name": "YourData",
        "skip_header": false,
        "transducers": [],
        "targets": {
            "word": "A",
            "definition": "B",
            "theme": "C",
            "seconday_theme": "D"
        }
    },
    "resource": "data.xlsx"
}
```

## Adding Information to the About Page

To populate the About Page in a Mother Tongues UI, you can add an image by providing a URL to an image in `aboutPageImg`, and you can also provide a description of your dictionary as seen below. The description can be plain text or HTML.

```json hl_lines="7-8"
{
    "config":{
        "L1": "Danish",
        "L2": "English",
        ...
        "alphabet": ["a", "b", "c"],
        "aboutPageImg": "https://placehold.co/600x400",
        "aboutPageDescription": "Here is a description of the dictionary. I can also use <b>HTML</b> here.",
    },
    ...
}
```
