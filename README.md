<!----
layout: page
title: Product review generation using conditional generative language model
description: CSCI 566 2019 Fall Project (NLP TextGen)
img: /assets/img/12.jpg
----->
**_Taejin Park, Yongwan Lim, Yichen Zhou, Kaixi Wang_**  
The rise of deep neural-network based approaches have significantly improved natural dialog with machines in the past few years. While conditional generative models have been successfully deployed in image/video applications, there is still much that can be done with generative language models such as GAN and VAE in text and language applications. 
### **Goal of this project**
The goal of this project is to artificially generate semantically and syntactically correct sentences given human inputted keyword prompts. Specifically, we are trying to address the question like *Can we generate text while controlling the output?* If we can control the output of generated text, we can apply this technique to many of real life applications, including chat-bot, AI speaker, predictive text, and so on. 
<!--<div class="img_row">
<img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig1.png" alt="" title="fig1"/>
</div>-->
We expect this project to have the following features:
* Generative language model
* Keyword prompts input: sentiment (rating), subject (product name), aggressiveness (vocabulary)
* Grammatically correct sentence output that contains a distinct context
### **Problem Definition**
To artificially generate semantically and syntactically correct review sentences given human inputted keyword prompts.  
* Training input: review texts, rating 1~5, subject category
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
<div class="img_row">
<img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig2.png" alt="" title="fig2"/>
</div>
<div class="col three caption">
Training mode CVAE
</div>
* Inference
<div class="img_row">
<img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig3.png" alt="" title="fig3"/>
</div>
<div class="col three caption">
Inference mode CVAE
</div>
* Conditional VAE system that uses keyword/sentiment as conditional input.
* Both encoder and decoder take the keyword input during training. 
* Decoder outputs a few sentences of review about a product driven by keyword input.
* Random noise input can work as a seed for generated review
* **Main problem**: the decoder ignores conditional input (mode collapse)
    * Example: 
        * 1-star input, 100 noise samples ➝  44 positive, 56 negative output 
        * 5-star input, 100 noise samples ➝  61 positive, 39 negative output 
### **Proposed Method: Improved CVAE**
* Training (CVAE + **Discriminator**) 
<div class="img_row">
<img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig4.png" alt="" title="fig4"/>
</div>
<div class="col three caption">
Training: CVAE + "Discriminator"
</div>
* Inference (**Conditional** Decoder + **Discriminator** + **Filtering**)
    <div class="img_row">
    <img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig5.png" alt="" title="fig5"/>
    </div>
    <div class="col three caption">
    "Conditional"" Decoder + "Discriminator"
    </div>
    <div class="img_row">
    <img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig6.png" alt="" title="fig6"/>
    </div>
    <div class="col three caption">
    "Conditional" Decoder + "Discriminator" + "Filtering"
    </div>
    
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
    <div class="img_row">
    <img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig8.png" alt="" title="fig8"/>
    </div>
    <div class="col three caption">
    Conditional decoder output with a range of 𝜶
    </div>
#### **Improvement of sentiment accuracy by conditional decoder and output filtering**
<div class="img_row">
<img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig9.png" alt="" title="fig9"/>
</div>
<div class="col two caption">
Sentiment Accuracy 
(*Sentiment accuracy is evaluated with different 𝜶 and 𝜶=0.65 gives the best performance.)
</div>
<!--|         | No Conditional Decoder (𝜶=0) | Conditional Decoder (𝜶=0.65*) | -->
<!--    | :-------------: |-------------| -----|| ------------- |-->
<!--    | Sentiment Accuracy | right-aligned | $1600 |-->
#### **Task 1: Real vs Generated**
1. Evaluation by Humans:
    **Accuracy (F1 score): 70.83% (73.31%)**
    <div class="img_row">
    <img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig10.png" alt="" title="fig10"/>
    </div>
    <div class="col three caption">
    Accuracy of 50% means humans cannot tell the difference.
    </div>
    * *Highlights* of evaluation by humans
    <div class="img_row">
    <img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig11.png" alt="" title="fig11"/>
    </div>
        
<!--            **GOOD** -->
<!--            (*More than 90% are fooled*)-->
<!--        -->
<!--            `these speakers are not the best sounding radio i have tried . i have had it for a few months and it stopped working . i am very disappointed in the quality of the product .`-->
<!--            -->
<!--            `this thing rocks ! ! ! ! easy to carry and works great . i can even see the difference in the picture and the sound quality is good .`-->
<!--            -->
<!--            `it has great sound , and the volume was quite adequate . easy to use and it was easy to set up . i like the way it is supposed to be .`-->
<!---->
<!--            **BAD** -->
<!--            (*Nobody was fooled*)-->
<!--            -->
<!--            `i have had this battery for a few years now and i **have no complaints** . so far this is a terrible product .`-->
<!--            -->
<!--            `pro is a great product . the only problem is that the unit is not compatible with the unit , but the battery is dead . i have to recharge it with a different charger .`-->
<!--            -->
<!--            `i have had this tv for over a year and have had it for a month and then just stopped working . i have had it for a month and now it has a great picture . `-->
      
#### **Task 2: Sentiment Accuracy**
1. Evaluation by humans:
 **Accuracy (F1 score):  87.5% (88.00%)**
    * Ground truth: majority vote of annotated sentiment scores (0 generatd or 1 real) from humans 
    * *Highlights* of evaluation by humans
    <div class="img_row">
    <img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig12.png" alt="" title="fig12"/>
    </div>
    
2. Evaluation by machine:
 **Accuracy (F1 score): 77.08% (80.70%)**
      * Softmax value of positive/negative sentiment class
3. Sentiment Score Comparison Machine vs Humans
  <div class="img_row">
  <img class="col three left" src="{{ site.baseurl }}/assets/img/project1_fig7.png" alt="" title="fig7"/>
  </div>
 
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
 
 
<!--Some preliminary results from the prototype models-->
<!---->
<!--1. CVAE output examples: Trained on 800,000 musical instrument section comments. Star rating of 5 was given as condition for CVAE model.-->
<!---->
<!--    * *Example 1*<br/>-->
<!--        `<sos> i have been using this for a few months now and i am very pleased with the quality of the sound . the sound is very good and the sound is very good . i have a pair of these in my studio and they are very comfortable  . <eos>`-->
<!---->
<!--    * *Example 2*<br/>-->
<!--       `<sos> i bought this for my son for christmas . he loves it . he has a lot of fun with it  . <eos> `-->
<!---->
<!--    * *Example 3*<br/>-->
<!--        `<sos> i love this microphone . i use it for my karaoke and it works great . i have a lot of fun with it . <eos>`-->
<!---->
<!--2.  GPT-2 language model examples: (unconditioned baseline)-->
<!---->
<!--    * *Example 1*<br/>-->
<!--        `The Contour Brush is a makeup brush that features majorly angled bristles and highly curated fine-sprinkle trays. Its medium size bristles on outward strokes makes it ideal for applied color and volumizing/contour techniques. Its single-handler mode allows for easy control and produces clean product, while elegant design accentuates the product's natural beauty. The contour brush requires a firm grip on the handle, utilized to control the crooked strokes that are most desired by makeup artists.`-->
<!--        -->
<!--    * *Example 2*<br/>-->
<!--        `Using this brush comes with a few different application techniques. First, you should apply your face powder using a straight, wide, upward stroke over the center, and play around with type of product to get the correct color. Then, use the medium type of powder brushes to apply powders to remove excess product with the downward strokes. Lastly, use the contour brush to apply the eyeshadow. Don't forget to dry the blush brush after to prevent rolling and contamination!`-->
<!---->
<!--    * Observations: -->
<!--        * Sampling methods have significant effects on results and intermediate training does not necessarily entail better accuracy-->
<!--        * Focus on language modeling has greatest potential to increase performance-->
