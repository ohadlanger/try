# Evaluation metric for SignWriting
### Introduction
The code presented here implements a new metric for evaluating the similarity of two given phrases written in FSW.
The goal of it's creation was to replace the generic and less task-speceific evaluation methods used to assess the quality of FSW transcription by default, such as
BLEU, CHRF and more, by a signwriting-concentrated approach that knows to evaluate similarity based on the language's rules and principles (that were otherwise tested by general string comperations)

### Evaluation Method
The evaluation method we created attempts to bridge the gap that opens when using general methods that were implemented to support strings, but not specfically signwriting strings.
By using a method that is not aware of the field that it is used in, it cannot propperly focus on the right attributes and guidelines, all the do and don'ts that were developed when creating this language.

The method we created take into account the following important notes and is conditioned to treat them correctly, as well as other features:
- Symbols in the FSW dictinary are orgenized nicley to portray their seperation to types such as hand signals, motion, touch and more. The closer in the table, usually the closer they are in their representation and meaning.
- Symbols that construct a sign can be written in different orders, yet represent the same visual output.
- Each part of a symbol has different meaning, and therefore different importance. Specifically: "S12345600x600" can be broken into aspecs such as kind of symbol ("123"), facing direction ("4"), angle ("5"), position ("600x600") and so on for different kinds of symbols.

### Main concept
The evaluation process we developed is built on four main stages, each with it's own intent and purposes:
1. **Symbol distance function**: this function is in chaarge of receiving two symbols (certain parts of signs), and calculate a distance value to represent the similarity or lack of it bewtween
   the two according to the language and sign table. It's grade and actual factors are based on the ones stated above, custom weights we chose to define importance of different differences and more.
2. **Distance Normalization**: the return value of the function does have a maximum value (the distance between the two polars of the table) but is not between 0-1. Therefore, we devide by the         maximumm distance and add an additional step of inserting the value into a non-linear function (x in the power of 1/5), which makes the value more clearly representating the ratio of the            distance.
   
   <img src="https://github.com/ohadlanger/try/assets/118103585/3dab6c81-272a-48e3-8f04-9f7fed840c38" width="20%" height="20%">


3. **Matching and grading**: now that we have established our ability to quantify the similarity of two symbols, we want to utilize this ability to generalize it for whole signs (series of             continuous symbols), and hopefully understand better the resemblence and common parts of two given inputs. The way we do that is by breaking down the sign into it's constructing symbols, and        looking for similar parts that can be assumed to be close in meaning. We do that by iterating over all couples of the two symbols arrays and creating a distance matrix, where each [i,j] slot        represent the output of the distance function on the i'th element of the first group, and the j'th of the second. Now, we pass this matrix to a Hungarian algorithm provided by the scipy library     for matching pairs of close parts in each sign. The next step is to take the distance from all of those ideal matchings, calculating the mean, and lastly, for taking into account the fact that      the inputs may differ in length (and their lenght difference is part of their similarity), we input this difference into another non-linear function to decide a weight for the mean difference       between the inputs, according to the lengths, and so get a better representation of a grade, which is the final value we return.
   
   <img src="https://github.com/ohadlanger/try/assets/118103585/3b706a19-a627-4b2e-bd9e-209506e81565" width="20%" height="20%">


### Examples

(These examples are taken from the DSGS Vokabeltrainer)

|              |                                   00004                                    |                                   00007                                    |                                   00015                                    |
|:------------:|:--------------------------------------------------------------------------:|:--------------------------------------------------------------------------:|:--------------------------------------------------------------------------:|
|    Video     |  <img src="assets/examples/00004.gif" width="150px">   |  <img src="assets/examples/00007.gif" width="150px">   |  <img src="assets/examples/00015.gif" width="150px">   |
| SignWriting  |   <img src="assets/examples/00004.png" width="50px">   |   <img src="assets/examples/00007.png" width="50px">   |   <img src="assets/examples/00015.png" width="50px">   |


## Tokenization

SignWriting can be tokenized using the [SignWriting Tokenizer](https://github.com/sign-language-processing/signbank-plus/blob/main/signbank_plus/signwriting/signwriting_tokenizer.py).

## Data

For this study, there are two notable lexicons, containing isolated sign language videos with SignWriting transcriptions.

- [Sign2MINT](https://sign2mint.de/) is a lexicon of German Signed Language (DGS) focusing on natural science subjects. It features 5,263 videos with SignWriting transcriptions. 
- [SignSuisse](https://signsuisse.sgb-fss.ch/) is a Swiss Signed Languages Lexicon that covers Swiss-German Sign Language (DSGS), French Sign Language (LSF), and Italian Sign Language (LIS). The lexicon includes approximately 4,500 LSF videos with [SignWriting transcriptions in SignBank](https://www.signbank.org/signpuddle2.0/index.php?ui=4&sgn=49).

(can also add around 2300 videos from the Vokabeltrainer)
