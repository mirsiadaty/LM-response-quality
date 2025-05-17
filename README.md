# LM-response-quality
method and software to improve quality of the response you get from your language model

Consider the observation: 
> *a given LM may forget to include some important items in the solution it generates (False Negative rate, FNR), BUT if we provide “hints” in the prompt, then the probability of FNR goes down to almost 0%.*

In other words, if we can reduce FNR, we will improve the quality of the LM response.

A practical question is, where do we get such useful 'hints' in the first place, so that we include them in the prompt we are sending to the LM!
<br>
<br>

Another observation is that:
> perturbing the LM prompt, will change the answer the LM generates, sometimes significantly.

For example, methods we use to modify/perturb the prompt include:
+ Synonym Replacement: Substitute words with their synonyms to observe if the change in vocabulary affects the output.
+ Adding/Removing Words or Phrases: Introduce or delete words, phrases, or even punctuation marks to test the model's sensitivity to these alterations.
+ Rephrasing Sentences: Rewrite sentences while preserving the core meaning to see how the model responds to different sentence structures.
+ insert whitespaces, amount location order
+ Adding Typos or Grammatical Errors: Introduce small errors to see if the model can still understand the prompt and provide a relevant response. 

The topic of "prompt engineering" attests to the above observation, that modifying the prompt we send to LM will affect the response we get from LM.

