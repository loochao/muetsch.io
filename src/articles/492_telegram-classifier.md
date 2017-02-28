## ML: Telegram chat message classification
February 28, 2017

### Intro
First of all, a short disclaimer: I'm not an expert in machine learning at all. In fact I'm in a rather early stage of the learning process, have basic knowledge and this project is kind of my first practical hands-on ML. I've done the [machine learning course](https://www.youtube.com/user/UWCSE/playlists?sort=dd&shelf_id=16&view=50) by [Pedro Domingos](https://homes.cs.washington.edu/~pedrod/) at University of Washington, [Intro to Machine Learning](https://www.udacity.com/course/intro-to-machine-learning--ud120) by Udacity and Google and the [Machine Learning 1 lecture at Karlsruhe Institute Of Technology](https://his.anthropomatik.kit.edu/english/28_315.php), all of which I can really recommend.
After having gathered all that theoretical knowledge, I wanted to try something practical on my own. I decided to learn a simple classifier for chat messages from my [Telegram](https://telegram.com) messenger history. I wanted to learn a program that can, given a chat message, tell who the sender of that message is. I further got inspired after having read the papers related to Facebook's [fastText](https://github.com/facebookresearch/fastText) text classification algorithm. In their examples they classify Wikipedia abstracts / descriptions to [DBPedia](https://dbpedia.org) classes or news article headlines to their respective news categories, only based on plain, natural words. Basically these problems are very similar to mine, so I decided to give it a try. Since I found that many text classifiers are learned using the Naive Bayes algorithm (especially popular in spam detection and part of [SpamAssassin](http://spamassassin.apache.org/)) and it's really easy to understand, I decided to go for that one, too. Inspired by [this article](http://www.laurentluce.com/posts/twitter-sentiment-analysis-using-python-and-nltk/), where the sentiment of tweets is analyzed, I chose to also use the [natural language toolkit](http://www.nltk.org/) for Python. Another option would have been [sklearn](http://scikit-learn.org/), but NLTK also provided some useful utilities beyond the pure ML scope. 

All of my __code is [available on GitHub](https://github.com/n1try/tg-chat-classification/)__.

### Basic Steps
1. The very first step was to download the data, namely the chat messages. Luckily, Telegram has an open API. However it's not a classical REST API, but instead they're using the [MTProto](https://core.telegram.org/mtproto) protocol. I found [vysheng/tg](https://github.com/vysheng/tg) as a cool C++-written commandline client on GitHub as well as [tvdstaaij/telegram-history-dump](https://github.com/tvdstaaij/telegram-history-dump) as a Ruby script to automate the history download for a set of users / chat partners. I told the script (ran in a Docker container, since I didn't want to install Ruby) to fetch at max 40,000 messages for my top three chat partners (let's call them _M_, _P_ and _J_). The outcome were three [JSON Lines](http://jsonlines.org/) files.
2. To pre-process these files as needed for my learning algorithm, I wrote a Python script that extracted only message text and sender from all incoming messages and dumped these data to a JSON file. Additionally I also extracted the same information for all outgoing messages, i.e. all messages where the sender was me. Consequently, there are four classes: __C = { _M_, _P_, _J_, _F_ }__
3. Another data-preprocessing step was to convert the JSON objects with class names as keys for message-arrays to one large list of tuples of the form _(text, label)_, where _label_ is the name of the message's sender and _text_ is the respective message text. In this step I also discarded words with a length of less than 2 characters and converted everything to lower case.
4. Next step was to extract the features. In text classification, there is often one binary (_contains_ / _contains not_) feature for every possible word. So if all messages in total comprise X different words, there will be a X-dimensional feature vector.
5. Last step before actually training the classifier is to compute the feature vector for every messages. For examples if the total feature set is `['in', 'case', 'of', 'fire', 'coffee', 'we', 'trust']`, the resultung feature vector for a message _"in coffee we trust"_ would be `('in'=True, 'case'=False, 'of'=False, 'fire'=False, 'coffee'=True, 'we'=True, 'trust'=True)`.
6. One more minor thing: shuffle the feature set so that the order of messages and message senders is random. Also divide the feature set into training- and test data, where test data contains about 10 % of the number of messages in the train data.
7. Train [nltk.NaiveBayesClassifier](http://www.nltk.org/api/nltk.classify.html) classifier. This is really just one line of code.
8. Use the returned classifier to predict classes for the test messages, validate them and compute the accuracy.

Using that basic initial setup on a set of __37257 messages__, (7931 from M, 9795 from P, 9314 from F and 10217 from J), I ended up with an __accuracy of 0.58__. There seemed to be room for optimization.

### Optimizations
* Inspired by _fastText_, I decided to include n-grams. This seemed resonable to me, because intuitively I'd say that single words a way less characteristic for a person's writing style than certain phrases. I extended the feature list from step 4 by all possible __bi- and tri-grams__, which are easy to compute with NLTK. Actually I'm not taking ALL bi- and tri-grams and I'm not even take all single words as features. Reason for that is that there were approx. 35k different words in the dataset. Plus the n-grams this would make an extremely multi-dimensional feature vector and as it turned out, it was way to complex for my 16 GB MacBook Pro to compute. Consequently, I only took the __top 5000 single words, bigrams and trigrams__, ranked descending by their overall frequency. 
* Since NLTK already provides a corpus of __stopwords__ (like "in", "and", "of", etc.), which are obviously not characteristic for a person's style of chatting, I decided to remove them (the German ones) from the message set in step 2.

With these optimizations, I ended up with an __accuracy of 0.61__ after a training time of 348 seconds (I didn't log testing time at that point).

### Conclusion
Certainly 61 % accuracy isn't really a good classifier, but at least significantly better than random guessing (chance of 1/4 in this case). However, I trained a __fastText__ classifier on my data as a comparison baseline and it even only reached __60 % accuracy__ (but with a much better __training time of only 0.66 seconds__). 
My intuitive explanation for these rather bad results is the complexity of the problem itself. Given only a set of words without any context and semantics, it's not only hard for a machine to predict the message's sender but also for a human. 
Moreover, given more training data (I'd need a longer message history) and more computing power to handle larger feature sets, the accuracy might further improve slightly.
Actually, the practical relevance of this project isn't quit high anyway, but it was a good practice for me to get into the basics of ML and it's really fun!

Please leave me feedback if you like to.