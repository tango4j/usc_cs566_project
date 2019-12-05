
# Additional Work: Fine-tuning GPT-2

## About GPT-2 ...
GPT-2 is OpenAI's large-scale unsupervised transformer-based language model. According to the team responsible for creating the model:



>GPT-2 generates synthetic text samples in response to the model being primed with an arbitrary input. The model is chameleon-like—it adapts to the style and content of the conditioning text. This allows the user to generate realistic and coherent continuations about a topic of their choosing...

>More about the model:
https://openai.com/blog/better-language-models/

## Fine-tuning GPT-2 for Conditional Review Generation

### Model
Although the full model was released recently, it was not possible to fine-tune its parameters using our current hardware. Therefore, we used the medium-sized version of the model with 355M parameters (still huge).

For full description of the model, see: https://github.com/openai/gpt-2/blob/master/model_card.md)

### Data
While large and complete, the Amazon dataset was not clean. It was not uncommon to find ratings that didn't seem to match the text of the review body. Moreover, the dataset was dominated by short responses like "Good product." In order to ensure the model learned the correct sentiments while also optimizing for longer, coherent reviews, the reviews were filtered using PySpark prior to training. Many traditional NLP techniques were tried including language modeling and sentiment analysis. However, it was found that the simplest and most effective filter to achieve good results was to only use reviews that had been voted as helpful.

### Example: Text Length Stats for Electronic Reviews

#### REVIEWS WITH 1+ HELPFUL VOTES (1002739 total)

 rating | avg review length
 ------------| ----------                  
 1     | 499
 2 | 690
 3 | 763
 4 | 818
 5 | 612


#### ALL REVIEWS:

rating | avg review length
------------| ----------                  
1 | 367
2 | 474
3 | 476
4 | 469
5 | 325

### Training
The original model was simply trained for next word prediction. In order to fine-tune the model for conditionally generating realistic reviews based on user inputted parameters such as rating, category, sentiment, etc, we prefixed each review in the training sample with a unique starting tag in the format of:

```<|startoftext|> [rating] ||| [headline...] ||| [review_body...] <|endoftext|>  ```

The training script for finetuning was adapted from the gpt_2_simple package with parameters:
* steps: 2048
* model_name: '355M'
* sample_length: 500

## Sample of Generated Outputs
### Baseline (no fine-tuning)

sample | output
-------|--------
 1 | All the way from England, I'm supposed to live here in England and I'm supposed to have to pay taxes here in the USA to live here."               They told me to pack my stuff.    I couldn't even find my passport photo to sign up and I was really confused as to what I'm supposed to pack.<br /><br />The passport photo showed me a passport that was over 2 hours in a plane. It's not that it was not, but I can't find it on this passport and I don't want anything but it's the only one that I've seen so far. I didn't even know how to even put on it.<br /><br />I finally made it to the airport about 30 minutes before they open, and the attendants asked me to come in and get some more stuff.       I was pretty shocked when I showed up at the airport.     The passport doesn't work...
2 | If Amazon thinks it's a mistake, I'm willing to pay a small amount of tax to have all my stuff back, but I didn't want them putting a stamp on my passport so they couldn't refund me. <br /> <br /> I would give it 3 stars because of the inconvenience, but it's a good item, and it's working. I would only pay $10-$20 tax in tax to live here in Australia. At least that way I have to pay taxes on this crappy piece of shit stuff they gave me over two hours ago. It's a good item though, and I would suggest others to get something I can use as a cheap travel passport.<br /><br />On the plus side though, I had my passport in my pocket when I came back. I can still see the date I booked it, and the dates I'll be leaving.<br /><br />

### Trained
Score	|    Headline    |	Review
------| ---------------|---------------
5	| Just what I was looking for	| My husband is always wanting to get a Bluetooth headset and has now found the ideal one in my pocket. It holds his music perfectly and is comfortable. It also has great sound quality for listening to music. Great sound, great built-in mic, and the Bluetooth connection works wonderfully. My husband can use his laptop to play his MP3 player while he works. The speakers work just fine on it. It is a bit loud but I get it loud. The headphones are very long and this is a great fit for those who want an outside ear, as well as others who require some sort of earbud. All in all, a great product for such a reasonable price.
2 |	No idea if these are the most comfortable headphones but, the design clearly stands out..| 	After a long run of these headphones, one person told me that the rubber earpieces actually fit the most comfortably, this is not a surprise to me.  They aren't comfortable enough to make a big impression just on the outer surface or even the inside.  Also: the rubber earpieces are designed for extended use.  The rubber earpieces are designed for long use, but for short periods of time (say, if your average run of this set broke, you'd have to try it again).
5	| I would buy those again	| I absolutely love these, even though the price was such that I did not get them in time. The sound is amazing and I would recommend these.
2	| This item never came |	If you are looking at something for cheap, it would be very nice to receive an item that never came.  This item did not come after 2 weeks.  I emailed Amazon who did not even respond to my email.
2	| So far, good. | But, some changes are needed	The first issue is that the unit worked fine from the first time I plugged it in. However, after a few weeks, the voice came on and informed me that the audio from this unit was no longer working, so I had to return it. I have spoken to an attorney and he's offering a new product of this product, but not one that was in the original unit
5	| Great|	This cable is great to watch your HD movies on your TV and DVR.<br />I use it to connect my DVR to a remote, and a laptop at the same time.

#### Example of raw output:
``` <|startoftext|> 1|||Totally useless!|||I have a 5GB Samsung 4x HD Samsung and it does not play in WMA for me.  This device will not play in MP2.  I have tried using my own audio sources but no matter how many times I do it will not playback.  I use my DVR as my audio source but it will only play in MP3 format.<br /><br />I'm going to send it back because it is so stupid not to have a MP3 device which will not play in WMA.  Please make a new unit or something.  For the price it is just a piece of junk.<|endoftext|> ```


(Checkpoint to load and test fine-tuned model included in project source repo)
