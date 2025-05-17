# LM-response-quality
method and software to improve quality of the response you get from your language model

Consider the observation: 
> *a given LM (language model) may forget to include some important items in the solution it generates (False Negative rate, FNR), BUT if we provide “hints” in the prompt, then the probability of FNR goes down to almost 0%.*

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
<br>
<br>

Here is the quality-improvement algorithm:

1. given a question Q, say "Which big tech stock has the largest year-to-date gain this year? How much is the gain?"
2. generate a set of prompts, call it set-P, using the "prompt perturbation methods" outlined above {synonymy, adding phrases, rephrasing, whitespace insertion, typo insertion}, where all of the prompts are asking the same Q, but with slightly different wording
3. take a set of LMs, call it set-L, for example {qwq-32b-q8_0.gguf, DeepSeek-R1-Distill-Qwen-14B-f16, Llama-4-Scout-17B-16E-Instruct}, and generate responses by sending set-P to set-L;
4. 



