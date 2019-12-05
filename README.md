
# Product review generation using conditional generative language model
### CSCI 566 2019 Fall Project 
### Team Name: NLP_TextGen

**_Taejin Park, Yongwan Lim, Yichen Zhou, Kaixi Wang_**  
  
### Motivation  

The rise of deep neural-network based approaches have significantly improved natural dialog with machines in the past few years. While conditional generative models have been successfully deployed in image/video applications, there is still much that can be done with generative language models such as VAE[1] in text and language applications. 

### Goal of this project  

The goal of this project is to artificially generate semantically and syntactically correct product review comments given human inputted keyword prompts. Specifically, we are trying to address the question: *Can we generate text while controlling the output?* If we can control the output of generated text, we can apply this technique to many of real life applications, including chat-bot, AI speaker, predictive text, and many others. 

![fig1](https://yongwanlim.github.io/assets/img/project1_fig1.png)

We expect this project to have the following features:  
* Generative language model
* Keyword prompts input: sentiment (rating), subject (product name), aggressiveness (vocabulary)
* Grammatically correct sentence output that contains a distinct context


### Problem Definition

To artificially generate semantically and syntactically correct review sentences given human inputted keyword prompts.  
* Training input: review texts, rating 1 or 5 
* Inference
    * Input: review rating 1 or 5
    * Output: review sentences containing and/or reflecting the given distinct context and sentiment
    
This would require us to being able to have randomness and controllability at the same time.
The main challenges of this problem would be that:

* Output is often generated independent of the conditioning input (mode collapse).
* Quality of generated sentence (repetitive phrases, too general output) 

### Traditional Method

#### Conditional Variational Auto-Encoder (VAE) [1]

#### Training
![fig2](https://yongwanlim.github.io/assets/img/project1_fig2.png)
* Conditional VAE system that uses keyword/sentiment as conditional input.
* Both encoder and decoder take the keyword input during training. 

#### Inference
![fig3](https://yongwanlim.github.io/assets/img/project1_fig3.png)

* Decoder outputs a few sentences of review about a product driven by keyword input.
* Random noise input can work as a seed for generated review

* **Limitation of conventional CVAE** : the decoder ignores conditional input (mode collapse)
    * Example: 
        * 1-star input, 100 noise samples ➝  44 positive, 56 negative output 
        * 5-star input, 100 noise samples ➝  61 positive, 39 negative output 
        
    * Even if we provide conditioning input to decoder, the sentiment is heavily dependent on random signal and conditioning input is not able to  change the sentiment already ingrained in random noise.
    * Thus, we need more powerful and efficient way to enforce the sentiment to the training system and inferencing system.
    
### Proposed Method: Improved CVAE 
#### Training (CVAE + **Discriminator**) 
![fig5](https://yongwanlim.github.io/assets/img/project1_fig4.png)

* Discriminator is a classifier that classifies sentiment into two different categories.
* Attaching discriminator enables the model to backpropagate the error from the sentiment (star rating) labels thus lead to more accurate sentiment.

#### Inference (**Conditional** Decoder + **Discriminator** + **Filtering**)

![fig5](https://yongwanlim.github.io/assets/img/project1_fig5.png)

* Since conventional CVAE system ignores the conditioning input, we can force the conditioned word output by leveraging the discriminator.
* The alpha value balances between the softmax value from discriminator and the softmax value from the decoder.
* If the alpha value is close to 0, the model outputs very plausible and grammatically correct sentence but inaccurate sentiment.
* If the alpha value is close to 1, the model outputs more accurate sentiment but with poor grammar and semantic.

![fig6](https://yongwanlim.github.io/assets/img/project1_fig6.png)

* If text output is not giving certain amount of confidence in terms of softmax value of the discriminator, we drop the output text and regenerate it.
    
### Neural Network Training

* Training Dataset:  Amazon review dataset [2]
    * Trained on a subset of 5 major categories (Electronics, mobile electronics, major appliances, and etc)
    * Trained on total 0.6M reviews
    * Vocabulary size of 60K words
    * Limit sentence length between 20 and 60 words, including punctuations.
    * Only use 1-star (negative) and 5-star (positive) ratings
    * 66% negative / 33% positive data (*Since negative comments tend to have more variability, doubling the dataset of negative training data balances the variability of output text's sentiment)
    * Example of training dataset: 
    
        `**Reviewer ID:** R1KKOXHNI8MSXU`  
        `Product category: Apparel`  
        `Review text: “This is the second leggings I have ordered, I wear both of them often. They wash well and I receive many compliments on them!”`  
        `5-star rating of the product: 5` 
        `Helpful votes: 3`
        
* Environment
    * Pytorch 0.4.1, CUDA 10.0, Python 3.6
    
### Experiments
We test the effectiveness of the proposed methods in terms of sentiment accuracy measure:

1. Conditional decoder
    * The conditional output accuracy is dependent on 𝜶.
    * Check how output text varies over different 𝜶.  
    
1. Output filtering
    * Rejects the output text with low discriminator output softmax probability
        
Sentiment accuracy is used as a performance metric, which is measured by BERT (Transformer) + LSTM sentiment classifier trained on IMDB dataset. Note that for sanity check, accuracy of 92.31% for IMDB test set.

### Evaluation
We evaluate the quality of artificially generated sentences along the following two dimensions:
1. Evaluation by humans:
    * 15 human participants   
    * **Task 1**: Real vs Generated 
        * 48 comments; 24 real, 24 generated
        * Machine generated texts are shuffled with human generated text.
        * Evaluators were asked whether they think the review is human or machine generated
    * **Task 2**: Sentiment Classification 
        * 48 comments; 24 1-star, 24 5-star
        * Evaluators were asked whether they think the review is positive or negative
    
1. Evaluation by algorithm:
    * BERT model based Bi-LSTM sentiment classifier (the same system as in experiment session)
    * Trained on IMDB data using BERT embeddings (IMDB test set accuracy: 92.31%)
    * **Task 2**: Sentiment Classification 
        * 48 comments; 24 1-star, 24 5-star
        
        
### Results
#### Control between sentiment and syntax   
* Example of conditional decoder output of negative condition,  star rating 1.
![fig8](https://yongwanlim.github.io/assets/img/project1_fig8.png)
* If alpha is 0, there is no influence of discriminator, and it sometimes generates sentiment that is not correspond to the given sentiment (The given star rating is 1, but alpha=0 sentence says "she loves it")
* Higher alpha values show grammatical errors (e.g. These are a terrible product) or sentences do not make any sense (e.g. no distortion when it goes)

#### Improvement of sentiment accuracy by conditional decoder and output filtering
![fig9](https://yongwanlim.github.io/assets/img/project1_fig9.png)
* Sentiment accuracy is evaluated with different 𝜶 and 𝜶=0.65 gives the best performance while showing a good balance between grammar and accurate sentiment.
* All the following evaluation is done with alpha value of 0.65.

#### Task 1: Real vs Generated
1. Evaluation by Humans:
    **Accuracy (F1 score): 70.83% (73.31%)**
    ![fig10](https://yongwanlim.github.io/assets/img/project1_fig10.png)
    
    * 50% is chance probablity and this means that some texts are very plausible while some are not.

    * *Highlights* of evaluation by humans
    ![fig11](https://yongwanlim.github.io/assets/img/project1_fig11.png)

    * We picked three sentences that all the human annotators said "Real" comments. These sentences show very consistent sentiment.
    * We also picked three sentences that all the human annotators said "Generated", which means failed output. These sentences show lots of conflicting sentiment and semantically incorrect phrases.

#### Task 2: Sentiment Accuracy
1. Evaluation by humans:
 **Accuracy (F1 score):  87.5% (88.00%)**
    * Ground truth: majority vote of annotated sentiment scores (0 generatd or 1 real) from humans 
    * *Highlights* of evaluation by humans
    ![fig12](https://yongwanlim.github.io/assets/img/project1_fig12.png)
    
    * We picked the sentences with coherent sentiment outcome from human annotators. These sentences have very consistent sentiment in each sentence.
    * We also picked the sentences which have conflicting annotations. These sentences have inconsistent sentiment which means failed output.
    
2. Evaluation by machine:
 **Accuracy (F1 score): 77.08% (80.70%)**
      * This score is from the softmax value of positive/negative sentiment class in BERT + BiLSTM model trained on IMDB datset.
      * BERT + BiLSTM model tends to output negative sentence as positive sentence since there are plenty of negative expressions that do not exist in IMDB dataset. (e.g. This plastic cover do not fit into my camera and very cheaply made.)
      
3. Sentiment Score Comparison Machine vs Humans
![fig7](https://yongwanlim.github.io/assets/img/project1_fig7.png)

* This correlation reflects the credibility of algorithm (machine) based evaluation.
* There are few highly conflicting outcomes: Human annotators are far better at capturing semantics from the text to judge the actual sentiment.
 
### Conclusion
1.  The challenge of ignored condition
    * Input condition to CVAE can be ignored and lead to mode collapse.
    * Conditional generative model should be carefully designed to avoid ignored condition problem.
 
1. Conditional decoder and output filtering 
    * Conditional decoder leverages the discriminator’s ability to force the condition input.
    * Output filtering also increases the quality of generated text.
 
 
### Future Work
1. Consistency of sentiment:
    * Time varying conditional decoder’s 𝜶 value that controls the condition 
    * Self attention algorithm to focus on the certain part of the generated text.
 
1. Generative method coupled with CVAE+Discriminator
    * A way to modify the random input to prevent the condition ignoring problem
 
### **Reference**
[1] Sohn, Kihyuk, Honglak Lee, and Xinchen Yan. “Learning Structured Output Representation using Deep Conditional Generative Models.” Advances in Neural Information Processing Systems. 2015.
[2] Amazon review dataset: http://jmcauley.ucsd.edu/data/amazon/ 
