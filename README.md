# LM-response-quality
method and software to improve quality of the response you get from your language model

**Reducing False Negatives**

Consider the observation: 
> *a given LM (language model) may forget to include some important items in the solution it generates (False Negative rate, FNR), BUT if we provide “hints” in the prompt, then the probability of FNR goes down to almost 0%.*

In other words, if we can reduce FNR, we will improve the quality of the LM response.

A practical question is, where do we get such useful 'hints' in the first place, so that we include them in the prompt we are sending to the LM!
<br>
<br>



**Perturbing the prompt**

Another observation is that:
> perturbing the LM prompt, will change the answer the LM generates, sometimes significantly.

For example, methods we use to modify/perturb the prompt include:
+ Synonym Replacement: Substitute words with their synonyms to observe if the change in vocabulary affects the output.
+ Adding/Removing Words or Phrases: Introduce or delete words, phrases, or even punctuation marks to test the model's sensitivity to these alterations.
+ Rephrasing Sentences: Rewrite sentences while preserving the core meaning to see how the model responds to different sentence structures.
+ insert/reduce whitespaces (amount, location, order, type)
+ Adding Typos or Grammatical Errors: Introduce small errors to see if the model can still understand the prompt and provide a relevant response.
+ besides above NLP methods, we may have an LM to rewrite a given prompt into different styles. For example the topic of 'meta prompting' where we use an advanced LM to optimize prompts for simpler ones.
+ also there are python libs that diversify prompts, for example 'https://github.com/MadcowD/ell' and 'https://github.com/stanfordnlp/dspy' and 'https://github.com/meta-llama/llama-prompt-ops'

The topic of "prompt engineering" attests to the above observation, that modifying the prompt we send to LM will affect the response we get from LM.
<br>
<br>

**Few-Shot-Learning Prompt-Engineering**

One approach is, instead of focusing to craft an excellent prompt to send to LM, we use our effort to generate a set of various versions of the initial prompt. We then get LM responses for the prompts. Finally we get the LM to compose a final answer that is the "central tendency" of the set of answers we just got from the LM.

The set of LM answers are the "hints" that we insert in the LM prompt for the final 'central tendency' answer.

Here is the quality-improvement algorithm:

1. given a question Q, say "Which big tech stock has the largest year-to-date gain this year? How much is the gain?"
2. generate a set of prompts, call it set-P, using the "prompt perturbation methods" outlined above {synonymy, adding phrases, rephrasing, whitespace insertion, typo insertion}, where all of the prompts are asking the same Q, but with slightly different wording / format;
3. take a set of LMs, call it set-L, for example {qwq-32b-q8_0.gguf, DeepSeek-R1-Distill-Qwen-14B-f16, Llama-4-Scout-17B-16E-Instruct}, and generate responses by sending set-P to set-L;
4. Generate prompt for a final LM, where the prompt includes both the original question Q plus the 'setP x setL' responses generated from above steps.
5. This final LM response should benefit from the "few-shot learning" method for prompt-engineering explained in this algorithm. The quality of the final LM response is lower-bounded by the maximum quality observed within set-P.
<br>
<br>

The following is excerpt of the code that implements the above algorithm, "few-shot-learning prompt-engineering".

**Sensitivity of LM to whitespaces in prompt**

To demonstrate effect of prompt-perturbation on language-model LM response, we use just extra whitespace in the form of blank lines added to the given prompt to make a new version of the prompt. Then we query the same LM (DeepSeek-R1-Distill-Qwen-14B-f16.gguf) with the two prompt. The following shows how significantly the LM answer changes.

When using regular/original prompt:

```
<｜begin▁of▁sentence｜>
You are a planning agent. If you do not know the answer, reply I don't know, don't make things up.
<｜User｜>
You need to follow these instructions strictly:
1.Generate a plan of actions, 'Action Plan', do not take any action.
2.What 'Action Plan' is: a numbered list of steps to take in order to answer the question marked up by the tags <Question> and </Question>.
3.Mark up your answer by tags <ActionPlan> and </ActionPlan>.
4.You need to use a Tool per step in your 'Action Plan'. Each tool can be used in more than one step, or in no step at all.
5.The Tools at your disposal are:
 5.1.'PythonCodeExecutor': it accepts a python code, executes it in python, returns the execution output back. Install packages first as a separate step (before executing any other python code), using the method: subprocess.check_call([sys.executable, "-m", "pip", "install", package])
 5.2.'ResponseComposer': it takes the Question, the ActionPlan, and the answers to the steps of the ActionPlan, then it composes the final response to the end-user who asked the Question.
 5.3.'HumanInTheLoop': after certain steps (like PythonCodeExecutor steps), add a step with the tool HumanInTheLoop where the human supervisor will verify you can continue with the next step in the plan. Otherwise HumanInTheLoop will terminate the execution of this ActionPlan.
6.For steps that use PythonCodeExecutor, generate the python code to execute such step, and include it in the tags <PyCode> and </PyCode>.
7.Add a final step for composing response to the end-user who asked the Question.

<Question>
What are the weather alerts in New York, Virginia, California, and Alaska?
/Question>
<｜Assistant｜>
```

then the LM answer is:

```
import requests
import pandas as pd

# Replace with your actual API key
API_KEY = 'your_api_key'

# List of states to check
states = ['New York', 'Virginia', 'California', 'Alaska']

# Function to get weather alerts
def get_weather_alerts(state):
    base_url = f"http://api.openweathermap.org/data/2.5/weather?q={state}&appid={API_KEY}"
    response = requests.get(base_url)
    data = response.json()
    if 'alerts' in data:
        return data['alerts']
    else:
        return f"No alerts for {state}"

# Fetch and store data
alerts_data = []
for state in states:
    alerts = get_weather_alerts(state)
    alerts_data.append({'State': state, 'Alerts': alerts})

# Convert to DataFrame and save to CSV
df = pd.DataFrame(alerts_data)
df.to_csv('weather_alerts.csv', index=False)

print("Weather alerts have been fetched and saved to weather_alerts.csv.")
```


Whereas when using prompt with extra newlines:

```
<｜begin▁of▁sentence｜>

You are a planning agent. If you do not know the answer, reply I don't know, don't make things up.

<｜User｜>

You need to follow these instructions strictly:

1.Generate a plan of actions, 'Action Plan', do not take any action.

2.What 'Action Plan' is: a numbered list of steps to take in order to answer the question marked up by the tags <Question> and </Question>.

3.Mark up your answer by tags <ActionPlan> and </ActionPlan>.

4.You need to use a Tool per step in your 'Action Plan'. Each tool can be used in more than one step, or in no step at all.

5.The Tools at your disposal are:

 5.1.'PythonCodeExecutor': it accepts a python code, executes it in python, returns the execution output back. Install packages first as a separate step (before executing any other python code), using the method: subprocess.check_call([sys.executable, "-m", "pip", "install", package])

 5.2.'ResponseComposer': it takes the Question, the ActionPlan, and the answers to the steps of the ActionPlan, then it composes the final response to the end-user who asked the Question.

 5.3.'HumanInTheLoop': after certain steps (like PythonCodeExecutor steps), add a step with the tool HumanInTheLoop where the human supervisor will verify you can continue with the next step in the plan. Otherwise HumanInTheLoop will terminate the execution of this ActionPlan.

6.For steps that use PythonCodeExecutor, generate the python code to execute such step, and include it in the tags <PyCode> and </PyCode>.

7.Add a final step for composing response to the end-user who asked the Question.



<Question>

What are the weather alerts in New York, Virginia, California, and Alaska?

/Question>

<｜Assistant｜>
```

the LM answer uses a completely different internet resource (using www.weather.gov instead of api.openweathermap.org), in addition to a different code structure (printing versus saving as file, also using a loop versus repeatedly calling the same function):

1. Using different internet resource, www.weather.gov instead of api.openweathermap.org
2. Saving the output into a file versus printing it on monitor
3. Using loop to process each item in the list (names of US states) versus repeatedly calling the same function on every item

```
<PyCode>
import requests
from bs4 import BeautifulSoup

def get_weather_alerts(state):
    url = f"https://www.weather.gov/alerts/{state}"
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    alerts = soup.find_all('div', class_='weather-alert')
    return [alert.get_text() for alert in alerts]

# Example usage:
ny_alerts = get_weather_alerts('new-york')
va_alerts = get_weather_alerts('virginia')
ca_alerts = get_weather_alerts('california')
ak_alerts = get_weather_alerts('alaska')

print("New York Weather Alerts:", ny_alerts)
print("Virginia Weather Alerts:", va_alerts)
print("California Weather Alerts:", ca_alerts)
print("Alaska Weather Alerts:", ak_alerts)
</PyCode>

<HumanInTheLoop>
Please verify that the Python code is correct and that the weather alerts can be fetched properly.
</HumanInTheLoop>

```

Note above changes in the LM behavior was the result of using exactly the same prompt (exactly the same words used, in exactly the same sequence) but only adding extra newlines.

One may see how other 'substantial' perturbation methods, such as synonym replacement or rewriting the sentences, might affect the LM response further.
