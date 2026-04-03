## Prompt engineering
In the context of natural language processing and LLMs, a prompt is an input provided to the model to generate a response or prediction. When you write a prompt, you are attempting to set up the LLM to predict the right sequence of tokens. Prompt engineering is the process of designing high-quality prompts that guide LLMs to produce accurate outputs. This process involves tinkering to find the best prompt, optimizing prompt length, and evaluating a prompt’s writing style and structure in relation to the task.
在NLP和LLM的背景下，prompt是指向模型提供的输入，用于生成响应或预测结果。当你编写提示时，你是在试图让LLM预测出正确的词序列。Prompt engineering是设计高质量提示的过程，旨在引导LLM生成准确的输出。这一过程包括不断调整以找到最佳prompt，优化prompt的长度，并评估prompt的写作风格和结构与任务的相关性。
Prompts might need to be optimized for your specific model, regardless of whether you use Gemini language models in Vertex AI, GPT, Claude, or an open source model like Gemma or LLaMA.
无论您使用的是 Vertex AI 中的 Gemini 语言模型、GPT、Claude，还是像 Gemma 或 LLaMA 这样的开源模型，**prompt内容都可能需要针对您的特定模型进行优化**。

## Output Configuration with Parameters
### Output length
An important configuration setting is the **number of tokens** to generate in a response. Generating more tokens requires more computation from the LLM, leading to higher energy consumption and potentially slower response times, which leads to higher costs.
一个重要的配置设置是响应中生成的token(词元)数量。生成更多的tokens需要model进行更多的计算，从而导致更高的能耗以及可能更慢的响应时间，进而导致更高的成本。

Example parameter:
Claude: max_tokens - the maximum number of tokens to generate before stopping. This parameter only specifies the absolute maximum number of tokens to generate, so Claude may stop before reaching this maximum. Besides, this is a hard stop, meaning that it may cause Claude to stop generating mid-word or mid-sentence.
在停止生成之前可生成的最大tokens。此参数仅指定了可生成的字数的绝对上限，因此Claude可能会在达到这个上限之前停止生成。此外，这是一个严格的停止条件，这意味着它可能会导致Claude在单词或句子中间停止生成。

### 
