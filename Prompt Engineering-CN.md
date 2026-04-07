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

## Prompt Techniques

### 零样本提示词
Zero-shot prompt是最基本的一种提示形式，它仅对任务进行描述，并为LLM提供一些初始文本，以便其开始工作。

![Table 1. An example of zero-shot prompting](https://github.com/ChloeWu822/LLM/blob/52ecc595e5e7f6f8a73a91f0a61d8aa11188834c/Table%201.%20An%20example%20of%20zero-shot%20prompting.png)

### 单样本 & 少样本
在为AI模型生成提示时可以提供examples，从而帮助模型理解所要求的内容，尤其想要引导模型生成特定输出结构或模式时，examples尤为重要。
**One-shot** prompt：提供一个example，其理念是模型能够依据该示例进行模仿，从而最有效地完成任务。
**Few-shot** prompt：向模型提供多个examples，从而增加模型遵循该模式的可能性。

Few-shot的examples数量取决于几个因素：任务的复杂程度、examples的质量以及生成式AI模型的能力。一般来说，few-shot引导应至少使用三个到五个examples。对于更复杂的任务，可能需要使用更多的examples；而由于模型的输入长度限制，也可能需要使用较少的examples。

![Table 2. An example of few-shot prompting](https://github.com/ChloeWu822/LLM/blob/main/Table%202.%20An%20example%20of%20few-shot%20prompting.png)

### System, contextual and role prompting
System, contextual and role prompting are all techniques used to guide how LLMs generate text, but they focus on different aspects:

System、contextual、role prompting均是用于指导LLM生成文本的几种技术，但它们关注的方面各不相同：
- **System prompting**设定了LLM的整体背景和目的。它明确了模型应执行的操作“大方向”，例如进行语言翻译、对评论进行分类等。
- **Contextual prompting** 提供了与当前对话或任务相关的具体细节或背景信息。它有助于模型理解所提出问题的细微差别，并据此调整回应。
- **Role prompting** 为LLM指定特定的角色或身份。这有助于模型生成符合所分配角色及其相关知识和行为的回应。

System、contextual、role prompting之间可能存在相当程度的重叠。例如，一个为系统分配role的提示，也可能包含contextual信息。然而，每种类型的prompt所起的主要作用略有不同：
- System prompt: 定义模型的基本功能和总体目标。
- Contextual prompt: 提供即时的、针对特定任务的信息，以指导回应。它高度针对当前任务或输入，且是动态变化的。
- Role prompt: 确定模型的输出风格和语气。它增加了特性和个性的层面。

注意：Claude的roles体系更隐式，不那么结构化，把更多控制权交给了prompt的内容本身，而不是消息的类型标签，鼓励用自然语言描述来控制行为，而非依赖固定的role标签。Gemini用 _role: contextual_ 传文档，而Claude把contextual的功能内化到content结构里，用content blocks实现同样效果；Claude的role字段是严格枚举的，只允许特定值user和assistant，并且顺序也有约束，对话必须严格交替(user → assistant → user 交替)；Claude有独立的system参数。Claude的方式更灵活，但需要开发者自己在system prompt和content里做好语义组织，而Gemini的方式对框架和工程化更友好，职责边界清晰。

#### System Prompt
System prompts对于生成符合特定要求的输出非常有用。“System prompts”这个名称实际上指的是“为系统提供额外的任务”。例如，可以使用system prompts来生成与特定编程语言兼容的代码片段，或者可以使用system prompts来返回某种结构。

![Table 3. An example of system prompting](https://github.com/ChloeWu822/LLM/blob/b2184f4c68af11a1a7ec9e2efd6ffd8ab3769133/Table%203.%20An%20example%20of%20system%20prompting.png)


以下表格以JSON格式返回输出结果。采用JSON格式可以迫使模型构建结构并减少幻觉。
![Table 4. An example of system prompting with JSON format](https://github.com/ChloeWu822/LLM/blob/b2184f4c68af11a1a7ec9e2efd6ffd8ab3769133/Table%204.%20An%20example%20of%20system%20prompting%20with%20JSON%20format.png)

注意：System prompts对于安全性和毒性方面的信息同样非常有用。要控制输出内容，只需在提示中添加一行额外内容，例如：“You should be respectful in your answer.”

#### Role Prompt
为AI模型设定一个角色视角，能为其提供一份关于语气、风格以及所需专业知识的概览，从而有助于提升输出内容的质量、相关性和有效性。

![Table 5. An example of role prompting](https://github.com/ChloeWu822/LLM/blob/fa89dade01614cae0d3cd9459c1fc285e6fa4c5b/Table%205.%20An%20example%20of%20role%20prompting.png)

以下是一些可供选择的风格：Confrontational(对抗式)、Descriptive(描述式)、Direct(直接式)、Formal(正式)、Humorous(幽默式)、Influential(有影响力)、Informal(非正式)、Inspirational(励志式)、Persuasive(说服式)。

![Table 6. An example of role prompting with a humorous tone and style](https://github.com/ChloeWu822/LLM/blob/fa89dade01614cae0d3cd9459c1fc285e6fa4c5b/Table%206.%20An%20example%20of%20role%20prompting%20with%20a%20humorous%20tone%20and%20style.png)

#### Contextual Prompt
Contextual prompts有助于确保AI的交互尽可能地流畅和高效，使得模型将能够更快地理解请求，并能够生成更准确且更相关的回复。

![Table 7. An example of contextual prompting](https://github.com/ChloeWu822/LLM/blob/fa89dade01614cae0d3cd9459c1fc285e6fa4c5b/Table%207.%20An%20example%20of%20contextual%20prompting.png)

### Step-back prompting
Step-back prompting是一种提升模型性能的技术，其原理是先促使LLM先考虑与当前具体任务相关的通用问题，然后将该通用问题的答案作为后续针对具体任务的提示输入，从而让LLM在尝试解决具体问题之前能够激活相关的背景知识和推理过程。这有助于减轻LLM回复中的偏见，因为它侧重于一般原则而非具体细节。

![Table 8. A traditional prompt before we compare it with a step back prompt](https://github.com/ChloeWu822/LLM/blob/9679eca35b4cb89bb06896c28117dcd0ba2295ec/Table%208.%20A%20traditional%20prompt%20before%20we%20compare%20it%20with%20a%20step%20back%20prompt.png)
![Table 9. An example of prompting for self consistency](https://github.com/ChloeWu822/LLM/blob/9679eca35b4cb89bb06896c28117dcd0ba2295ec/Table%209.%20An%20example%20of%20prompting%20for%20self%20consistency.png)
![Table 10. An example of prompting for self consistency](https://github.com/ChloeWu822/LLM/blob/9679eca35b4cb89bb06896c28117dcd0ba2295ec/Table%2010.%20An%20example%20of%20prompting%20for%20self%20consistency.png)


### Chain of Thought (CoT)
Chain of Thought (CoT) prompting是一种通过**生成中间推理步骤**来提升推理能力的技术。

![Table 12. An example of Chain of Thought prompting](https://github.com/ChloeWu822/LLM/blob/1c1f4c86cbe65db9626c247e5e3a2f0b3444b73b/Table%2012.%20An%20example%20of%20Chain%20of%20Thought%20prompting.png)

优点：
1. 操作简便，效果显著，并且能与现成的LLM完美配合（无需进行微调）。
2. 更好的可解释性，可以从LLM的回复中学习，并看到所遵循的推理步骤，便于识别bug。
3. CoT在不同版本的LM之间切换时似乎能提高其稳定性，所以使用CoT的prompt在不同LLM之间的性能差异会更小。

缺点：
需要输出更多tokens，意味着更高的成本和更长的回复时间。

将它与few-shot prompting结合起来，可以在更复杂的推理任务中获得更好的结果。

![Table 13. An example of chain of thought prompting with a single-shot](https://github.com/ChloeWu822/LLM/blob/1c1f4c86cbe65db9626c247e5e3a2f0b3444b73b/Table%2013.%20An%20example%20of%20chain%20of%20thought%20prompting%20with%20a%20single-shot.png)

### 一致性(Self-consistency)

LLM的推理能力被视为无法通过单纯增大模型规模来克服的局限性。正如在CoT prompting部分所了解到的，模型可以被提示生成类似于人类解决问题时的推理步骤。然而，CoT使用的是简单的“贪婪解码”策略，这限制了其有效性。

Self-consistency结合了采样和投票的机制，可以生成多样化的推理路径，并选择最一致的答案。它提高了LLM生成的答复的准确性和连贯性，可以给出一个answer is correct的伪概率，但显然成本很高。

其步骤如下：
1. 生成多样化的推理路径：给LLM多次重复提供相同的prompt，high temperature的设置会促使模型生成不同的推理路径和对问题的不同看法。
2. 从每个生成的回复中提取答案。
3. 选择出现次数最多的答案。

![Table 14. An example of prompting for self consistency-1.png](https://github.com/ChloeWu822/LLM/blob/baa0bb26e2c4fd726b482ff2dad538b8f1bc3949/Table%2014.%20An%20example%20of%20prompting%20for%20self%20consistency-1.png)
![Table 14. An example of prompting for self consistency-2.png](https://github.com/ChloeWu822/LLM/blob/263337c46d5e8f75ef296291e1bd6386c8149567/Table%2014.%20An%20example%20of%20prompting%20for%20self%20consistency-2.png)
![Table 14. An example of prompting for self consistency-3.png](https://github.com/ChloeWu822/LLM/blob/7147144e12ad6365223859ccaddbf965fb1a3f67/Table%2014.%20An%20example%20of%20prompting%20for%20self%20consistency-3.png)

### Tree of Thoughts (ToT)
Tree of Thoughts(ToT)将CoT的概念进行了扩展，因为它使LLM能够**同时探索多种不同的推理路径**，而非仅仅遵循单一的线性流程。

![Figure 1. A visualization of chain of thought prompting on the left versus. Tree of Thoughts prompting on the right.png](https://github.com/ChloeWu822/LLM/blob/0378887decea682761f872b2724ca9fba73471ee/Figure%201.%20A%20visualization%20of%20chain%20of%20thought%20prompting%20on%20the%20left%20versus.%20Tree%20of%20Thoughts%20prompting%20on%20the%20right.png)


### ReAct (reason & act)
Reason and act (ReAct) prompting是一种解决复杂任务的范式，它将自然语言推理与外部工具（搜索、代码解释器等）相结合，使大型语言模型能够执行某些action (例如与外部API进行交互以获取信息)，这是构建Agent的第一步。

ReAct类似于人类在现实世界中的运作方式，通过语言进行推理，并采取行动来获取信息。

LLMs首先对问题进行分析并制定出action plan。接着，它会执行该plan中的各项行动，并观察结果。然后，LLMs会利用这些观察结果来更新其推理过程，并生成新的action plan。持续这个过程，一直到该模型找到问题的解决方案。

在代码片段1中，使用了Python的langchain框架，同时结合了VertexAI（google-cloud-aiplatform）和 google-search-results 这两个pip包。
![]()

代码片段2展示了结果。请注意，ReAct进行了五次搜索链式操作。LLM抓取谷歌搜索结果中获取信息，以确定金属乐队有四名成员。然后它会搜索每个乐队成员，请求他们的子女总数，并将所有总数相加。最后，返回子女的总数作为最终答案。
!()[]
