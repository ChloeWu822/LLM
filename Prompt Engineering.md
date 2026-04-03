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
