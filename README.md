<!----
layout: page
title: Product review generation using conditional generative language model
description: CSCI 566 2019 Fall Project (NLP TextGen)
img: /assets/img/12.jpg
----->

# Product review generation using conditional generative language model
### CSCI 566 2019 Fall Project 
### Team Name: NLP_TextGen

**_Taejin Park, Yongwan Lim, Yichen Zhou, Kaixi Wang_**  
  
### Motivation  
The rise of deep neural-network based approaches have significantly improved natural dialog with machines in the past few years. While conditional generative models have been successfully deployed in image/video applications, there is still much that can be done with generative language models such as GAN and VAE in text and language applications. 

### **Goal of this project**
The goal of this project is to artificially generate semantically and syntactically correct produc review comments given human inputted keyword prompts. Specifically, we are trying to address the question: *Can we generate text while controlling the output?* If we can control the output of generated text, we can apply this technique to many of real life applications, including chat-bot, AI speaker, predictive text, and many others. 

![fig1](https://yongwanlim.github.io/assets/img/project1_fig1.png)

We expect this project to have the following features:  
* Generative language model
* Keyword prompts input: sentiment (rating), subject (product name), aggressiveness (vocabulary)
* Grammatically correct sentence output that contains a distinct context


### **Problem Definition**

To artificially generate semantically and syntactically correct review sentences given human inputted keyword prompts.  
* Training input: review texts, rating 1 or 5 
* Inference
    * Input: review rating 1~5, keywords (subject category,...)
    * Output: review sentences containing and/or reflecting the given distinct context and sentiment
    
This would require us to being able to have randomness and controllability at the same time.
The main challenges of this problem would be that:
* Output is often generated independent of the conditioning input (mode collapse).
* Quality of generated sentence (repetitive phrases, too general output) 

### **Previous Method**
#### Conditional Variational Auto-Encoder (VAE)
* Training
![fig2](https://yongwanlim.github.io/assets/img/project1_fig2.png)

* Inference
![fig3](https://yongwanlim.github.io/assets/img/project1_fig3.png)
* Conditional VAE system that uses keyword/sentiment as conditional input.
* Both encoder and decoder take the keyword input during training. 
* Decoder outputs a few sentences of review about a product driven by keyword input.
* Random noise input can work as a seed for generated review
* **Limitation of conventional CVAE **: the decoder ignores conditional input (mode collapse)
    * Example: 
        * 1-star input, 100 noise samples ➝  44 positive, 56 negative output 
        * 5-star input, 100 noise samples ➝  61 positive, 39 negative output 
        
        
### **Proposed Method: Improved CVAE**
* Training (CVAE + **Discriminator**) 
![fig5](https://yongwanlim.github.io/assets/img/project1_fig4.png)

* Inference (**Conditional** Decoder + **Discriminator** + **Filtering**)
![fig5](https://yongwanlim.github.io/assets/img/project1_fig5.png)
![fig6](https://yongwanlim.github.io/assets/img/project1_fig6.png)

* Output filtering by discriminator’s softmax value
    
### **Network Training**

* Training Dataset:  Amazon review dataset [3]
    * Train with a subset of 5 major categories (Electronics, mobile electronics, major appliances, and etc)
    * Train on 0.6M reviews
    * Vocabulary size of 60K words
    * Limit sentence length between 20 and 60 words, including punctuations.
    * Only use 1-star (negative) and 5-star (positive) ratings
    * 66% negative / 33% positive data
    * Example  
    
        `Reviewer ID: R1KKOXHNI8MSXU`  
        `Product category: Apparel`  
        `Review text: “This is the second leggings I have ordered, I wear both of them often. They wash well and I receive many compliments on them!”`  
        `5-star rating of the product: 5` 
        `Helpful votes: 3`
        
* Environment
    * Pytorch 0.4.1, CUDA 10.0, Python 3.6
### **Experiments**
We test the effectiveness of the proposed methods in terms of sentiment accuracy measure:
1. Conditional decoder
    * The conditional output accuracy is dependent on 𝜶.
    * Check how output text varies over different 𝜶.  
1. Output filtering
    * Rejects the output text with low discriminator output softmax probability
        
Sentiment accuracy is used as a performance metric, which is measured by BERT (Transformer) + LSTM sentiment classifier trained on IMDB dataset. Note that for sanity check, accuracy of 92.31% for IMDB test set.
### **Evaluation**
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
    * BERT model based Bi-LSTM sentiment classifier
    * Trained on IMDB data using BERT embeddings (IMDB test set accuracy: 92.31%)
    * **Task 2**: Sentiment Classification 
        * 48 comments; 24 1-star, 24 5-star
### **Results**
#### **Control between sentiment and syntex**   
* Example of conditional decoder output of negative condition,  star rating 1.
![fig8](https://yongwanlim.github.io/assets/img/project1_fig8.png)
Conditional decoder output with a range of 𝜶

#### **Improvement of sentiment accuracy by conditional decoder and output filtering**
![fig9](https://yongwanlim.github.io/assets/img/project1_fig9.png)
Sentiment accuracy is evaluated with different 𝜶 and 𝜶=0.65 gives the best performance.

#### **Task 1: Real vs Generated**
1. Evaluation by Humans:
    **Accuracy (F1 score): 70.83% (73.31%)**
    ![fig10](https://yongwanlim.github.io/assets/img/project1_fig10.png)

    * *Highlights* of evaluation by humans
    ![fig11](https://yongwanlim.github.io/assets/img/project1_fig11.png)

       
     
#### **Task 2: Sentiment Accuracy**
1. Evaluation by humans:
 **Accuracy (F1 score):  87.5% (88.00%)**
    * Ground truth: majority vote of annotated sentiment scores (0 generatd or 1 real) from humans 
    * *Highlights* of evaluation by humans
    ![fig12](https://yongwanlim.github.io/assets/img/project1_fig12.png)
    
2. Evaluation by machine:
 **Accuracy (F1 score): 77.08% (80.70%)**
      * Softmax value of positive/negative sentiment class
3. Sentiment Score Comparison Machine vs Humans
![fig7](https://yongwanlim.github.io/assets/img/project1_fig7.png)
 
### **Conclusion**
1.  The challenge of ignored condition
    * Input condition to CVAE can be ignored and lead to mode collapse.
    * Conditional generative model should be carefully designed to avoid ignored condition problem.
 
1. Conditional decoder and output filtering 
    * Conditional decoder leverages the discriminator’s ability to force the condition input.
    * Output filtering also increases the quality of generated text.
 
 
### **Future Work**
1. Consistency of sentiment:
    * Time varying conditional decoder’s 𝜶 value that controls the condition 
    * Self attention algorithm to focus on the certain part of the generated text.
 
1. Generative method coupled with CVAE+Discriminator
    * A way to modify the random input to prevent the condition ignoring problem
 
