# Deepfake Tweet Author Detection System
> Yujie Song, Jeremy Yang, Ruoyao Yin, Shengjie Zhang

## Abstract
This paper developed a pre-trained model based tweet author detection system to detect who is the author of the tweet (human, GPT-2, RNN, or other text generative models). The paper explored three different strategies to improve the current pre-trained models: a more complex prediction layer, a parallel encoding layer, and feature engineering. Experiments showed that all these three strategies can improve model performance. Specifically, we first tried to fed the BERT embedding into a CNN-linear classifier rather than the Dense linear-linear one. The result shows that CNN can indeed improve the base BERT model for this classification task, even though the improvement is not very significant. Then, we tried to concatenate output of parallel BERT and output of LSTM and then do classification. The results show that this modification also improves the performance. Finally, we do feature engineering on our model with lexical and syntactic diversities feature. It improves the performance either. Crucially, the improvements found in the complex prediction layer and parallel encoding layer disagree with previous work on model complexity for text classification with pre-trained models. On the other hand, the feature engineering experiment confirmed previous efforts in the human-machine binary text classification problem.

## Introduction
Social media platforms like Tweet can be used to manipulate and affect public opinions through fake news. With the rapid development of neural text generative models (TGMs), auto-generated deceptive texts become so indistinguishable from human-written texts that they can be used by adversaries during political election periods to contaminate public debates and affect public opinions. We call these deceptive texts generated by neural models "deepfakes", which are a series of AI-generated media (images, audios, videos, and texts). The malicious application of TGMs facilitates the transmission of deepfakes and weakens the public trust in political institutions and issues. To tackle this problem, it is of vital importance to develop a deepfake social media text detection system that can distinguish human-written tweets from AI-generated tweets. Moreover, since some TGMs (i.e. GPT-2 and RNN) can generate more human-like texts than others, it is also worthwhile to distinguish between different types of TGMs. The detection system can mitigate the spreading of deepfake tweets and strengthen public trust in political issues. On the other hand, when training the text detection system together with the neural text generative model, we can improve the text generative model and generate texts more similar to human-written texts. This is beneficial to the development of systems that require text generation, such as machine translation.

Deepfake tweets detection was first risen by [Tiziano Fagni, et al. (2021)](https://arxiv.org/pdf/2008.00036.pdf), who tried 13 text detection methods and found that pre-trained language model with fine-tuning is the most effective approach for deepfake text detection. They found that RoBERTa-based detector performed best. Based on this, we are wondering what else can be done to improve the performance of the pre-trained models. [Zhao Z., et al. (2021)](https://eprints.whiterose.ac.uk/173371/1/A_Comparative_Study_of_Using_Pre_trained_Language_Models_forToxic_Comment_Classification.pdf) tried different pre-trained language models and downstream structures for toxic comment classification problem and found that complex prediction layers like BiLSTM and CNN can harm model performance. As far as we know, currently there are no studies explored whether the conclusion remains to be true for the Deepfake tweets detection problem. Moreover, [Fröhling, L. & Zubiaga, A. (2021)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8049133/pdf/peerj-cs-07-443.pdf) used feature-based methods to distinguish between human-written texts and machine-generated texts. Therefore, it is worthwhile to explore whether these features can be helpful for classifying texts generated by different machines.

In this project, we developed a deepfake social media text detection system that can classify whether a tweet is written by human or generated by RNN, GPT-2 or other models. We explored three strategies to improve the pre-trained model based text detection system. The first strategy is to replace the linear dense layers in the prediction layer into convolutional neural network (CNN). The second strategy is to add a parallel BiLSTM model and concatenate the vector representation from BiLSTM with that from BERT. The third strategy is to add features that proved useful for distinguishing human-written texts and model-generated texts to baseline BERT model and see if it is helpful for differentiating texts generated by different models.

The rest of the paper is structured as follows. The related work section will provide a detailed literature review about previous works concerned with text detection problem. The dataset section will introduce the dataset that we use. The method & experiments section describes the technical details of our research. The result section provides a summary of our experiment results. Finally, the conclusion section discusses and summarizes our main conclusions based on the experiment results.

## Related Work
Deepfake tweets detection was first risen in [TweepFake: about detecting deepfake tweets](https://arxiv.org/pdf/2008.00036.pdf). First, the tweets are collected from a total of 23 bots, imitating 17 human accounts. The bots are based on several text generation technologies: Markov Chains, RNN, RNN+Markov, LSTM, GPT-2. In total, based on several state-of-the-art approches, they evaluated 13 deepfake text detection methods. These methods can be summarized into 3 categories: machine-learning classifiers with text representations as inputs, deep learning networks and transformer-based classifiers with fine-tuning. According to the results, the pre-trained language models with fine-tuning seem to be the most effective approach. RoBERTa-based detector performed best with nearly 90% accuracy. The architecture of this detector is just a trained language model integrating with a final dense classification layer.

In [A Comparative Study of Using Pre-trained Language Models for
Toxic Comment Classification](https://eprints.whiterose.ac.uk/173371/1/A_Comparative_Study_of_Using_Pre_trained_Language_Models_forToxic_Comment_Classification.pdf), they selected 15,000 to 159,571 comments from four datasets from three different social media platforms, Facebook, Twitter and Wikipedia, with different classification types. To classify toxic comments, they tried different pre-trained language model-based methods based on three popular language model, BERT, RoBERTa and XLM. The structures are shown below. The results suggest that using a basic linear downstream network architecture is a better choice over a complex one, such as CNN and Bi-LSTM.

![structure](./structure.png)


[Feature-based detection of automated language models: tackling GPT-2, GPT-3 and Grover](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8049133/pdf/peerj-cs-07-443.pdf) shed lights on feature-based methods for generated text detection. They produced a feature-based detector, offering a "first line-of-defense" against the abuse of language models, with competitive performance comparing with far more expensive, complex and computationally more restrictive methods. They collected publicly available data either scraped from the Internet or more randomly curated from existing corpora. Different sampling methods lead to different types of flaws in generated text. They summarized the flaws of generated language into 4 categories, lack of syntactic and lexical diversity, repetitiveness, lack of coherence and lack of purpose. To detect lack of syntactic and lexical diversity, they used NE-recognizer and POS-tagger in `spaCy` to find the NE- and POS-tags. Since our datasets are collections from Twitter, most of data are just one sentence, we only consider the lack of syntactic and lexical diversity here. They focused on the evaluation of Neural Networks due to its superior performance on validation set among all classifiers, LR, SVM, Neural Networks and Random Forests. The evaluation was based on the accuracy and the area under curve (AUC) of the receiver operating characteristic curve (ROC). Based on the results of training and evaluating detectors on the individual subsets of features, syntactic, lexical diversity and basic features are the most important feature subsets. Construct a sub-classifier for each dataset and use an ensemble approach to combine the outputs will perform better that single classifiers. They mentioned that the future work can be to systematically evaluate the classifiers' performances in more realistic settings like forum discussions, blog posts or wider social media discourse.

## Dataset

TweepFake is a Twitter deep fake text dataset with both human genereated text and machine generated text from various sources. Deep fake tweets were collected from Twitter accounts with keywords in their description (Twitter account description or Github description) relating to automatic, AI text generation, or model names like "GPT-2". Only accounts that mimicks human Twitter profiles were selected. As shown by the table below, 23 bot accounts were selected and classified into three main groups: GPT-2 (11 accounts, 3861 tweets), RNN (7 accounts, 4181 tweets), and Others (5 accounts, 4876 tweets), where "Others" include methods like Markov Chain, RNN + Markov Chain, LSTM, and CharRNN. On the other hand, 17 human accounts were identified and also included in the dataset.

|   | Human Generated Tweets  | GPT-2 Generated Tweets  | RNN Generated Tweets  | Other Tweets  |
|---|---|---|---|---|
|Tweeter accounts   | 17  | 11  | 7  | 5  | 
|Tweets   | 12786  | 3861  |4181  | 4876  |

The dataset is balanced between human generated tweets and machine generated tweets. However, it is unbalanced between human generated and each of the machine generated tweet group. Despite this unbalance, it is important to note that tweets among different machine generated tweet groups are also reasonably balanced, and we believe this contributed to our model performance.

The dataset is split roughly in 8:1:1 into a training set (20712 tweets, with 10354 bot tweets and 10358 human tweets), a validation set (2302 tweets, with 1152 bot tweets and 1150 human tweets) and a test set (2558 tweets, with 1280 bot tweets and 1278 human tweets). The data is ready-to-use on [kaggle](https://www.kaggle.com/datasets/mtesconi/twitter-deep-fake-text), and we stored a copy of it in their original state (csv) in google drive for later use in Google Colab.

## Methods & Experiments

Under the classic BERT-linear classifier text classification model, we conducted three engineering experiments with different tweaks on the model. For all three experiment models as well as the baseline, we used BertTokenizerFast tokenizer and BERT-base-uncased model.  The baseline model consists of the BERT model and a simple dense feed-forward layer followed by another feed-forward classification layer. The final feed-forward prediction layer produces a softmaxed probability distribution over the four possible labels (human, GPT-2, RNN, Others). For hyperparameters, we used a batch size of 8 and a learning rate (AdamW) of 3e-5, all models were trained on 3 epochs.

As shown in Fig 1 below, we added modifications on the baseline model to conduct each of the three experiments. For experiment one, we fed the BERT embedding into a CNN-linear classifier instead of the Desnse linear-linear one. For the second experiment, we trained a LSTM model parallel to the BERT model. Then the embeddings from the two models were concatenated as one feature vector to be fed into the linear classification layers. 


![image](./COLX585%20Model.jpeg)Fig 1. Flowchart of all three experiment models


Finally, for the third experiment, we tokenized the input text with [SpaCy](https://spacy.io/) en_core_web_sm tokenizer and extracted linguistic features for feature engineering. To be more specific, we extracted syntactic diversity and lexical diversity from the input text data and recorded them as well as an averaged joint diversity. For syntactic and lexical diversity, we use relative distributions of NE-tags and POS-tags as generated by the SpaCy model over tweet. A combined measure was also used which is simply the mean of the two diversity measures. After the linguistic features were extracted, it was concatinated to the BERT embeddings before it was fed into the linear classifier.

We deployed the model on Google Colab, using GPU for computations. For our framework, apart from [Pytorch](https://pytorch.org/), we used [HuggingFace](https://huggingface.co/bert-base-uncased) for Bert pretrained embeddings and [SpaCy]((https://spacy.io/)) for feature engineering.

## Results
### Results: Baseline model
In our project, we first built baseline models with BERT. We employed BERT base model (`bert_base_uncased`) and BERT large model (`bert_large_uncased`) respectively with two dense layers to build classifiers. After 3 epochs, we got the following results:

|   | BERT base model  | BERT large model  |
|---|---|---|
|Epoch 1 train loss:   | 0.2922  | 0.5266  |
|Epoch 1 validation loss:   | 0.4363  | 0.3695  |
|Epoch 1 validation accuracy:  | 0.8658  | 0.8493  |
|Epoch 1 validation f1 | 0.8649 | 0.8205 |
|Epoch 2 train loss:   | 0.1357  | 0.2424  |
|Epoch 2 validation loss:   | 0.5719  | 0.3492  |
|Epoch 2 validation accuracy:  | 0.8810  | 0.8649  |
|Epoch 2 validation f1 | 0.8782 | 0.8648 |
|Epoch 3 train loss:   | 0.0532  | 0.1176  |
|Epoch 3 validation loss:   | 0.6053  | 0.4048  |
|Epoch 3 validation accuracy:  | 0.8727  | 0.8727  |
|Epoch 3 validation f1 | 0.8707 | 0.8660 |

From the results, we discovered that the validation accuracy and validation f1 score for both BERT base model and BERT large model are around 0.87. Even with more Epochs, the difference between those two models for accuracy and validation is within 0.05 (5%). 

Another thing we noticed from the results is about the loss:

|   | BERT base model  | BERT large model  |
|---|---|---|
|Epoch 1 train loss:   | 0.2922  | 0.5266  |
|Epoch 2 train loss:   | 0.1357  | 0.2424  |
|Epoch 3 train loss:   | 0.0532  | 0.1176  |
|Epoch 1 validation loss:   | 0.4363  | 0.3695  |
|Epoch 2 validation loss:   | 0.5719  | 0.3492  |
|Epoch 3 validation loss:   | 0.6053  | 0.4048  |

we noticed that the train losses of BERT base are much smaller than those of BERT large. We assumed that it is because the BERT base model has much fewer parameters than that of BERT large model. It is not as complex as BERT large model as well. Therefore, BERT base model can generalize more easily. Therefore, we decided to use BERT base in our project.

In addition, with the increase of Epoch, the train loss for both models decrease, which means the fitting for train data progresses well. However, the validation loss for both increase. Overfitting happens here since validation data are unseen data for the models. We improved our model in the next week and tried three approaches we discussed in the methods & experiments part above. 

### Experiment results for approach 1: Change dense layer into CNN
For experiment one, we fed the BERT embedding into a CNN-linear classifier instead of the Dense linear-linear one.
We conducted experiments to see if it is true that a complex prediction layer harms the model performance, as the [Toxic Comment Classification](https://eprints.whiterose.ac.uk/173371/1/A_Comparative_Study_of_Using_Pre_trained_Language_Models_forToxic_Comment_Classification.pdf) paper shows. I used the `last_hidden_state` from the output of BERT as the input of CNN and tried different CNN architectures with different kernel sizes in different regions and different number of kernels in each region. The following are the experiment results:

|kernel_sizes    |   kernel_num  |  Validation F1-score   |
|---|----|-----|
|[2,3,4] | 2 |  0.8641  |
|[2,3,4] | 6 |  0.8683 |
|[2,3,4] | 16 | 0.8659 |
|[2,3,4] | 32 | **0.8753** |
|[2,3,4] | 64 | **0.8729** |
|[2,3,4] | 81 | **0.8715** |
|[2,3,4,5] | 6 | 0.8608 |
| [2,3]  |  6 | 0.8683 |
| [2,3]  |  4 | **0.8730** |
|  [4]   |  6 | 0.8649 |
|  [3]   |  6 | 0.8672 |
|  [2]   |  6 | 0.8651 |

Baseline f1-score: 0.8707

Surprisingly, our results are contradicted with the Toxic Comment Classification paper. CNN can indeed improve the base BERT model for this classification task, even though the improvement is not very significant. The best model we got is `kernel_sizes=[2,3,4]`, `kernel_num=32`, with validation f1-score **0.8753**. This may be because CNN can capture some extra information from the output of BERT.

### Experiment results for approach 2: Concatenate the output of LSTM with BERT
We concatenated the contextualized embeddings from BERT, the pooler output, with the output from LSTM. Then passed it to the dense layer for classification. For the output of LSTM, we tried last hidden layer, mean of last hidden layer for bi-directional LSTM, and the mean of all hidden layers. We also tried different values for the hyperparameters, number of layers, embedding size and LSTM hidden size.

| Type  | Output of LSTM | Number of layers | Embedding size | LSTM hidden size | Validation F1-score |
|-------|----------------|------------------|----------------|------------------|---------------------|
| Unidirectional | last hidden layer | 1 | 300 | 512 | 0.8727 |
| Unidirectional | last hidden layer | 1 | 300 | 800 | 0.8803 |
| Bidirectional | concatenated last hidden layer | 1 | 300 | 512 | 0.8760 |
| Bidirectional | concatenated last hidden layer | 1 | 300 | 800 | **0.8814** |
| Bidirectional | mean of last hidden layer | 1 | 300 | 512 | 0.8807 |
| Bidirectional | mean of last hidden layer | 1 | 300 | 800 | 0.8794 |
| Bidirectional | mean of all hidden layers | 1 | 100 | 512 | 0.8722 |
| Bidirectional | mean of all hidden layers | 1 | 300 | 512 | 0.8768 |
| Bidirectional | mean of all hidden layers | 2 | 100 | 512 | 0.8747 |
| Bidirectional | mean of all hidden layers | 2 | 300 | 512 | 0.8740 |
| Bidirectional | mean of all hidden layers | 3 | 100 | 512 | 0.8771 |
| Bidirectional | mean of all hidden layers | 3 | 300 | 512 | 0.8726 |
| Bidirectional | mean of all hidden layers | 5 | 100 | 512 | 0.8803 |
| Bidirectional | mean of all hidden layers | 5 | 300 | 512 | 0.8751 |

From the results listed above, we can see that all results are higher than the base line, but the differences are small. Also, the results of LSTM with different hyperparameters are very close to each other. This may indicate that LSTM does improve the performance but not much. The best model we got is concatenating the last hidden layer of Bi-LSTM with 1 layers, 300 embedding size and 800 hidden size, with highest F1-score, 0.8814, which is about 1% higher than the baseline F1-score 0.8707.

### Experiment results for approach 3: Feature Engineering
The third experiment we conducted was whether feature engineering has an impact on the base BERT text classification model. Its performance and a comparison with the baseline can be found in the table below:

|   | baseline BERT model  | BERT model with feature engineering |
|---|---|---|
|Epoch 1 validation F1:   | 0.8649  |  0.8828  |
|Epoch 2 validation F1:   | 0.8782   | 0.8803  |
|Epoch 3 validation F1:   | 0.8707  | 0.8803  |

We observed a noticeable improvement when we include lexical and syntactic diversity as features for the model. This matches our expectation that human text and machine-generated text differs in their lexical and syntactic diversities. In addition, this suggests that such a differentiation also occurs between different machine text generation methods.

### Test results
After our experiments, we got best models from the above approaches and applied them to the test data. The results can be found in the table below:

|   | baseline BERT model  |  BERT + CNN  |  parallel BERT + LSTM  | BERT + feature engineering | 
|---|---|---|---|---|
|Epoch 1 validation F1:   | 0.8649  | 0.8477 | 0.8474  |  0.8828  |
|Epoch 2 validation F1:   | 0.8782 | 0.8686  |  0.8720  | 0.8803  |
|Epoch 3 validation F1:   | 0.8707  | 0.8753  | 0.8814  | 0.8803  |
|test F1:   |   | 0.8716  | 0.8739  | 0.8715  |

Across our different experiments, the feature engineering model produced the best performance in terms of validation macro-f1 score. However, the model with parallel BERT and LSTM produced the best performance in terms of test macro-f1 score. 

From the results, we can see that all those three approaches we tried got higher scores than that of baseline and improved the performance. Although the differences are small and the improvement is not very apparent. 

Furthermore, comparing the test results with validation results, we can see that test results are lower than that of validation. It is explainable. There may be overfitting when we training and tuning our models and test them on validation data. When we apply our models on test data, which is unseen for models, the performance will decrease a bit. 

## Conclusion
In our project, we developed a deepfake social media text detection system to classify whether a tweet is written by human or generated by RNN, GPT-2 or other models. We also explored three strategies to improve the pre-trained model based text detection system and to see if they are helpful for differentiating texts generated by different models. 

From our experiments and results, we discovered that, unlike some previous works suggest, adding model complexity on a BERT-Linear classification model does improve model performance. Moreover, both a richer representation and a more complex classification architecture can help us to identify and tell the texts generated by different models. 

However, there might have some room to move up in our project. Given more time, we may do classification analysis. In addition, we could explore further about why hyperparameter-tuning does not help a lot in improving performance and why the results of model varied and not very stable.

Further investigations can focuses on an analysis of a combined model. We may also consider to explore how the model performs on the specific machine generated text classes.