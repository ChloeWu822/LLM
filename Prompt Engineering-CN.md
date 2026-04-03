## Prompt engineering

在NLP和LLM的背景下，prompt是指向模型提供的输入，用于生成响应或预测结果。当你编写提示时，你是在试图让LLM预测出正确的词序列。Prompt engineering是设计高质量提示的过程，旨在引导LLM生成准确的输出。这一过程包括不断调整以找到最佳prompt，优化prompt的长度，并评估prompt的写作风格和结构与任务的相关性。

无论使用Gemini、GPT、Claude，还是像Gemma或LLaMA这样的开源模型，**prompt内容都可能需要针对您的特定模型进行优化**。

## 输出配置
### 输出长度
一个重要的配置设置是响应中生成的token(词元)数量。生成更多的tokens需要LLM进行更多的计算，从而导致更高的能耗以及可能更慢的响应时间，进而导致更高的成本。
例如：Claude: max_tokens，在停止生成之前可生成的最大tokens。此参数仅指定了可生成的字数的绝对上限，因此Claude可能会在达到这个上限之前停止生成。此外，这是一个严格的停止条件，这意味着它可能会导致Claude在单词或句子中间停止生成。

### 抽样控制
LLM会预测下一个token可能是什么的概率，而不是正式预测单个的token，LLM vocabulary中的每个token都有相应的概率。然后从这些token概率中进行抽样，以确定接下来生成的token是什么。Temperature、top-K和top-P是最常见的配置设置，它们决定了预测的token概率如何被处理，从而选出一个最终的输出token。

#### Temperature参数
参数temperature控制token选择的随机性程度。较低的temperature倾向于生成更确定的响应，而较高的temperature可能导致更多样化或意想不到的结果(例如想要尝试创造性的输出)。

Temperature为0的输出是确定的，因为它总是选择概率最高的标记。但请注意：如果两个tokens具有相同的最高预测概率，那么根据平局时的处理方式，当温度为0时，可能并不总是得到相同的输出。

接近最大值的temperature往往会产生更多的随机输出，并且随着temperature越来越高，所有tokens都有可能成为下一个预测token。

#### Top-K & top-P参数
参数top-K和top-P限制预测的下一个token来自于预测概率最高的那些tokens。与temperature一样，这些采样设置控制生成文本的随机性和多样性。

Top-K抽样是从模型的预测分布中选取最有可能出现的前K个tokens。值越高，模型的输出就越具有创新性和多样性；值越低，模型的输出则越趋于稳定和事实性。当Top-K = 1时，相当于采用贪婪解码方式。

Top-P抽样会选择累积概率之和不超过P值的前几个tokens。P的取值范围从 0（贪婪式解码）到 1（使用vocabulary的所有词）。

#### 交互机制
如果temperature、top-K、和top-P一起使用，那么同时满足top-K和top-P标准的tokens将会是下一个预测标记的候选选项，然后会应用temperature来从候选tokens中进行抽样。如果temperature不可用，则会从满足top-K/top-P标准的tokens中随机选取一个来生成下一个预测token。


- 如果将temperature设为0，那么top-K和top-P将不再起作用，最有可能的token将直接成为下一个预测的token。如果将temperature设置得极高，那么temperature将不再起作用，而是随机从满足top-K/top-P条件的token中进行抽取。
- 如果将top-K设为1，temperature和top-P将不再起作用，因为只有一个token符合top-K条件。如果将top-K设置得极高（例如，与vocabulary大小相当），那么将只会过滤掉概率为0的tokens。
- 如果将top-P设为0，temperature和top-K将不再起作用，只考虑最有可能的token。如果将top-P设为1，任何具有非零概率的tokens都将成为接下来要预测的tokens的候选。

一般情况：temperature = .2, top-P = .95, top-K = 30。</br>
创新性结果：temperature = .9, top-P = .99, top-K = 40。</br>
非创新性结果：temperature = .1, top-P = .9, top-K = 20。</br>
唯一结果：temperature = 0。</br>


警告：不恰当的temperature、top-k、top-p设置可能会导致“重复循环错误”（回复以大量填充词结尾），这是LLMs中常见的问题，即模型陷入循环，反复生成相同的（填充）词、短语或句子结构。这种情况在low temperature和high temperature下都可能发生。在low temperature的情况下，模型变得过于确定，死板地遵循概率最高的路径，如果该路径再次访问之前生成的文本，就可能导致循环。相反，在high temperature下，模型的输出变得过于随机，随机选择的单词/短语有更大的概率回到先前的状态，从而由于大量可用选项而形成循环。

注意：在拥有更多自由度（更高的temperature、top-k、top-p）的情况下，LLMs的文本可能会变得不相关或者无意义。
