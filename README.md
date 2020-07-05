# CS50AI - Week 6 - Questions

## Task:

Write an AI that answers questions, by determining the most relevant document(s) using tf-idf ranking and then extracting the most relevant sentence(s) using idf and a query term density measure.


## Background:

Question Answering (QA) is a field within natural language processing focused on designing systems that can answer questions. Among the more famous question answering systems is Watson, the IBM computer that competed (and won) on Jeopardy!. A question answering system of Watson’s accuracy requires enormous complexity and vast amounts of data, but in this problem, we’ll design a very simple question answering system based on inverse document frequency.

Our question answering system will perform two tasks: document retrieval and passage retrieval. Our system will have access to a corpus of text documents. When presented with a query (a question in English asked by the user), document retrieval will first identify which document(s) are most relevant to the query. Once the top documents are found, the top document(s) will be subdivided into passages (in this case, sentences) so that the most relevant passage to the question can be determined.

How do we find the most relevant documents and passages? To find the most relevant documents, we’ll use tf-idf to rank documents based both on term frequency for words in the query as well as inverse document frequency for words in the query. Once we’ve found the most relevant documents, there many possible metrics for scoring passages, but we’ll use a combination of inverse document frequency and a query term density measure (described in the Specification).

More sophisticated question answering systems might employ other strategies (analyzing the type of question word used, looking for synonyms of query words, lemmatizing to handle different forms of the same word, etc.) but we’ll leave those sorts of improvements as exercises for you to work on if you’d like to after you’ve completed this project!


### Term-Frequency, Inverse Document Frequency, Quantity Tern Density Metrics:

Many different metrics could be used to determine which documents and sentences are most likely to be relevant to a particular query. A simple metric is term frequency (tf) - the number of times a term in the query appears in the document. While this metric is straightforward, it suffers from some drawbacks, including:

  * Both queries and documents will often contain many 'function words' - words such as 'which', 'with', 'yet', 'the', 'and', 'a', 'I', 'of', etc. These words hold little meaning on their own, but grammatically connect other words. When such words are included in the term-frequency analysis, these will often be among the most frequent words and yet are the least relevant words to any query. Often these words will therefore be ignored in a term frequency analysis to avoid this issue, using a precomipled list of these 'function words'.
  * Even having ruled out these 'function words', documents covering different specialist topics of an overall encompassing subject will often still have many similar terms used with high term frequencies. In addition a larger document might have a higher term frequency for certain words, just by virtue of having many more words inside, without being the most relevant document for that specific term.

An improvement on the term-frequency metric can be made by using the inverse document frequency (idf) metric. This metric gives a measuure of how common or rare a word is across a set of documents (or passages/sentences). One way to calculate the idf metric for a word in a corpus of documents is:

<img src="https://render.githubusercontent.com/render/math?math=idf(word) = \ln \frac{Number \hspace{0.1cm} of \hspace{0.1cm} Docs \hspace{0.1cm} in \hspace{0.1cm} Corpus}{Number \hspace{0.1cm} of \hspace{0.1cm} Docs \hspace{0.1cm} Containing \hspace{0.1cm} Word}">

Here, a word that is found in all documents in the corpus is given a score of zero while words that are found in fewer of the documents get a higher score. This gives a better indication of which words are more important keywords for a query or document, and which words are commonly used across more documents and so less relevant. By combining the tf and idf metrics (multiplying their values together), documents can be ranked by the term frequency of the more important words across all documents.

In this project, the most relevant documents are found by ranking them in order of the sum of tf-idf for all the query terms. Once the most relevant document(s) have been found, these are split into sentences and the inverse document frequency for the words in all the sentences are calculated. Sentences are then ranked by the sum of the idf metric for all query terms found in each sentence, ignoring the term frequency. Where two sentences are found to have the same idf score, they are then ranked by query term density - the number of words in the sentence that are in the query, divided by the total number of words in the sentence.


# Specification:

Complete the implementation of load_files, tokenize, compute_idfs, top_files, and top_sentences in questions.py.

*  The load_files function should accept the name of a directory and return a dictionary mapping the filename of each .txt file inside that directory to the file’s contents as a string.
  * Your function should be platform-independent: that is to say, it should work regardless of operating system. Note that on macOS, the / character is used to separate path components, while the \ character is used on Windows. Use os.sep and os.path.join as needed instead of using your platform’s specific separator character.
  * In the returned dictionary, there should be one key named for each .txt file in the directory. The value associated with that key should be a string (the result of reading the corresonding file).
  * Each key should be just the filename, without including the directory name. For example, if the directory is called corpus and contains files a.txt and b.txt, the keys should be a.txt and b.txt and not corpus/a.txt and corpus/b.txt.

* The tokenize function should accept a document (a string) as input, and return a list of all of the words in that document, in order and lowercased.
  * You should use nltk’s word_tokenize function to perform tokenization.
  * All words in the returned list should be lowercased.
  * Filter out punctuation and stopwords (common words that are unlikely to be useful for querying). Punctuation is defined as any character in string.punctuation (after you import string). Stopwords are defined as any word in nltk.corpus.stopwords.words("english").
  * If a word appears multiple times in the document, it should also appear multiple times in the returned list (unless it was filtered out).

* The compute_idfs function should accept a dictionary of documents and return a new dictionary mapping words to their IDF (inverse document frequency) values.
  * Assume that documents will be a dictionary mapping names of documents to a list of words in that document.
  * The returned dictionary should map every word that appears in at least one of the documents to its inverse document frequency value.
  * Recall that the inverse document frequency of a word is defined by taking the natural logarithm of the number of documents divided by the number of documents in which the word appears.

* The top_files function should, given a query (a set of words), files (a dictionary mapping names of files to a list of their words), and idfs (a dictionary mapping words to their IDF values), return a list of the filenames of the the n top files that match the query, ranked according to tf-idf.
  * The returned list of filenames should be of length n and should be ordered with the best match first.
  * Files should be ranked according to the sum of tf-idf values for any word in the query that also appears in the file. Words in the query that do not appear in the file should not contribute to the file’s score.
  * Recall that tf-idf for a term is computed by multiplying the number of times the term appears in the document by the IDF value for that term.
  * You may assume that n will not be greater than the total number of files.

* The top_sentences function should, given a query (a set of words), sentences (a dictionary mapping sentences to a list of their words), and idfs (a dictionary mapping words to their IDF values), return a list of the n top sentences that match the query, ranked according to IDF.
  * The returned list of sentences should be of length n and should be ordered with the best match first.
  * Sentences should be ranked according to “matching word measure”: namely, the sum of IDF values for any word in the query that also appears in the sentence. Note that term frequency should not be taken into account here, only inverse document frequency.
  * If two sentences have the same value according to the matching word measure, then sentences with a higher “query term density” should be preferred. Query term density is defined as the proportion of words in the sentence that are also words in the query. For example, if a sentence has 10 words, 3 of which are in the query, then the sentence’s query term density is 0.3.
  * You may assume that n will not be greater than the total number of sentences.

You should not modify anything else in questions.py other than the functions the specification calls for you to implement, though you may write additional functions, add new global constant variables, and/or import other Python standard library modules.


## Usage:

Requires Python3 and the python package installer pip(3) to run.

First install requirements (nltk):

$pip3 install -r requirements.txt

Run the script with a corpus of .txt documents to search for your query:

python3 questions.py corpus_directory

