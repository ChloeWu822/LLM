## Prompt engineering
In the context of natural language processing and LLMs, a prompt is an input provided to the model to generate a response or prediction. When you write a prompt, you are attempting to set up the LLM to predict the right sequence of tokens. Prompt engineering is the process of designing high-quality prompts that guide LLMs to produce accurate outputs. This process involves tinkering to find the best prompt, optimizing prompt length, and evaluating a prompt’s writing style and structure in relation to the task.

Prompts might need to be optimized for your specific model, regardless of whether you use Gemini language models in Vertex AI, GPT, Claude, or an open source model like Gemma or LLaMA.

## Output Configuration with Parameters
### Output length
An important configuration setting is the **number of tokens** to generate in a response. Generating more tokens requires more computation from the LLM, leading to higher energy consumption and potentially slower response times, which leads to higher costs.

Example parameter:
Claude: max_tokens - the maximum number of tokens to generate before stopping. This parameter only specifies the absolute maximum number of tokens to generate, so Claude may stop before reaching this maximum. Besides, this is a hard stop, meaning that it may cause Claude to stop generating mid-word or mid-sentence.


### Sampling controls
LLMs predict probabilities for what the next token could be, with each token in the LLM’s vocabulary getting a probability, rather than formally predict a single token. Those token probabilities are then sampled to determine what the next produced token will be. Temperature, top-K, and top-P are the most common configuration settings that determine how predicted token probabilities are processed to choose a single output token.

#### Temperature
Temperature controls the degree of randomness in token selection. Lower temperatures are good for prompts that expect a more deterministic response, while higher temperatures can lead to more diverse or unexpected results (e.g. good for experimenting with creative outputs).

A temperature of 0 is deterministic: the highest probability token is always selected. Though note: if two tokens
have the same highest predicted probability, depending on how tiebreaking is implemented you may not always get the same output with temperature 0.

Temperatures close to the max tend to create more random output. And as temperature gets higher and higher, all tokens become equally likely to be the next predicted token.

#### Top-K and top-P
Top-K and top-P are used to restrict the predicted next token to come from tokens with the top predicted probabilities. Like temperature, these sampling settings control the randomness and diversity of generated text.

Top-K sampling selects the top K most likely tokens from the model’s predicted distribution. The higher top-K, the more creative and varied the model’s output; the lower top-K, the more restive and factual the model’s output. A top-K of 1 is equivalent to greedy decoding.

Top-P sampling selects the top tokens whose cumulative probability does not exceed a certain value (P). Values for P range from 0 (greedy decoding) to 1 (all tokens in the LLM’s vocabulary).

#### Interaction mechanism
If temperature, top-K, and top-P are all available, tokens that meet **both** the top-K and top-P criteria are candidates for the next predicted token, and then temperature is applied to sample from the tokens that passed the top-K and top-P criteria. If temperature is not available, whatever tokens meet the top-K and/or top-P criteria are then randomly selected from to produce a single next predicted token.

- If you set temperature to 0, top-K and top-P become irrelevant, and the most probable token becomes the next token predicted. If you set temperature extremely high, temperature becomes irrelevant, and just randomly sample from tokens meet the top-K and/or top-P criteria.
- If you set top-K to 1, temperature and top-P become irrelevant, since only one token passes the top-K criteria. If you set top-K extremely high (e.g., size of the LLM’s vocabulary), then none are selected out excpet tokens with a zero probability.
- If you set top-P to 0, temperature and top-K become irrelevant, and only the most probable token is considered. If you set top-P to 1, any token with a nonzero probability are candidates for the next predicted toke.


General starting point: temperature = .2, top-P = .95, top-K = 30. </br>
Creative results: temperature = .9, top-P = .99, top-K = 40. </br>
Less creative results: temperature = .1, top-P = .9, top-K = 20. </br>
Single answer: temperature = 0. </br>

WARNING: Inappropriate temperature and top-k/top-p settings may cause "repetition loop bug" (response ending with a large amount of filler words), which is a common issue in LLMs where the model gets stuck in a cycle, repeatedly generating the same (filler) word, phrase, or sentence structure. And this can occur at both low and high temperature settings. At low temperatures, the model becomes overly deterministic, sticking rigidly to the highest probability path, which can lead to a loop if that path revisits previously generated text. Conversely, at high temperatures, the model's output becomes excessively random, increasing the probability that a randomly chosen word/phrase will back to a prior state, creating a loop due to the vast number of available options. 

NOTE: With more freedom (higher temperature, top-K, top-P), the LLM might generate text that is less relevant.

## Prompt Techniques

### General prompting / zero shot
A zero-shot prompt is the simplest type of prompt, and it only provides a description of a task and some text for the LLM to get started with.

![Table 1. An example of zero-shot prompting](https://github.com/ChloeWu822/LLM/blob/52ecc595e5e7f6f8a73a91f0a61d8aa11188834c/Table%201.%20An%20example%20of%20zero-shot%20prompting.png)

### One-shot & few-shot
When creating prompts for AI models, it is helpful to provide examples. These examples can help the model understand what you are asking for and especially useful when you want to steer the model to a certain output structure or pattern.
**One-shot** prompt, provides a single example, hence the name one-shot. The idea is the model has an example it can imitate to best complete the task.
**Few-shot** prompt, provides multiple examples to the model. Multiple examples of the desired pattern increases the chance the model follows the pattern.

The number of examples for few-shot prompting depends on a few factors: the complexity of the task, the quality of the examples, and the capabilities of the
generative AI model. As a general rule of thumb, it should use at least three to five examples for few-shot prompting. It may need to use more examples for more complex tasks, or it may need to use fewer due to the input length limitation of the model.

![Table 2. An example of few-shot prompting](https://github.com/ChloeWu822/LLM/blob/main/Table%202.%20An%20example%20of%20few-shot%20prompting.png)

### System, contextual and role prompting
System, contextual and role prompting are all techniques used to guide how LLMs generate text, but they focus on different aspects:

- **System prompting** sets the overall context and purpose for the language model. It defines the ‘big picture’ of what the model should be doing, like translating a language, classifying a review etc.
- **Contextual prompting** provides specific details or background information relevant to the current conversation or task. It helps the model to understand the nuances of what’s being asked and tailor the response accordingly.
- **Role prompting** assigns a specific character or identity for the language model to adopt. This helps the model generate responses that are consistent with the assigned role and its associated knowledge and behavior.

There can be considerable overlap between system, contextual, and role prompting. E.g. a prompt that assigns a role to the system, can also have a context. However, each type of prompt serves a slightly different primary purpose:
- System prompt: Defines the model’s fundamental capabilities and overarching purpose.
- Contextual prompt: Provides immediate, task-specific information to guide the response. It’s highly specific to the current task or input, which is dynamic.
- Role prompt: Frames the model’s output style and voice. It adds a layer of specificity and personality.

Note: Claude's "role system" is more implicit and less structured, handing over more control to the content itself rather than the type labels of the messages. It encourages the use of natural language to control behavior instead of relying on fixed role labels. Gemini uses _role: contextual_ to pass documents, while Claude incorporates the **contextual** function into the content structure and achieves the same effect through content blocks; Claude's **role** is strictly limited to the two specific values ("user", "assistant"), and there are also constraints on the order, i.e. the conversation must strictly alternate (user → assistant → user); Claude has an independent **system** parameter. Claude's approach is more flexible, but it requires developers to do semantic organization in the system prompt and content themselves. Gemini's approach is more friendly to the framework and engineering, with clear responsibility boundaries.

#### System Prompt

System prompts can be useful for generating output that meets specific requirements. The name ‘system prompt’ actually stands for ‘providing an additional task to the system’. For example, you could use a system prompt to generate a code snippet that is compatible with a specific programming language, or you could use a system prompt to return a certain structure. 

![Table 3. An example of system prompting](https://github.com/ChloeWu822/LLM/blob/b2184f4c68af11a1a7ec9e2efd6ffd8ab3769133/Table%203.%20An%20example%20of%20system%20prompting.png)


The following table returns the output in JSON format. By prompting for a JSON format it forces the model to create a structure and limit hallucinations.

![Table 4. An example of system prompting with JSON format](https://github.com/ChloeWu822/LLM/blob/b2184f4c68af11a1a7ec9e2efd6ffd8ab3769133/Table%204.%20An%20example%20of%20system%20prompting%20with%20JSON%20format.png)

Note: System prompts can also be really useful for safety and toxicity. To control the output, simply add an additional line to your prompt like: ‘You should be respectful in your answer.’.


#### Role Prompt

Defining a role perspective for an AI model gives it a blueprint of the tone, style, and focused expertise you’re looking for to improve the quality, relevance, and effectiveness of the output. 

![Table 5. An example of role prompting](https://github.com/ChloeWu822/LLM/blob/fa89dade01614cae0d3cd9459c1fc285e6fa4c5b/Table%205.%20An%20example%20of%20role%20prompting.png)

Here are some styles can choose: Confrontational, Descriptive, Direct, Formal, Humorous, Influential, Informal, Inspirational, Persuasive

![Table 6. An example of role prompting with a humorous tone and style](https://github.com/ChloeWu822/LLM/blob/fa89dade01614cae0d3cd9459c1fc285e6fa4c5b/Table%206.%20An%20example%20of%20role%20prompting%20with%20a%20humorous%20tone%20and%20style.png)

#### Contextual Prompt
Contextual prompts can help ensure that AI interactions are as seamless and efficient as possible. The model will be able to more quickly understand the
request and be able to generate more accurate and relevant responses.

![Table 7. An example of contextual prompting](https://github.com/ChloeWu822/LLM/blob/fa89dade01614cae0d3cd9459c1fc285e6fa4c5b/Table%207.%20An%20example%20of%20contextual%20prompting.png)



