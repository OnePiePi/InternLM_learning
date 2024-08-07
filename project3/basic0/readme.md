InternLM2技术报告总结

\1.   简介

InternLM2是国产开源的大型语言模型（LLM），由上海人工智能实验室开发，旨在在多个基准测试中超越其前身和其他开源模型。InternLM2通过创新的预训练和优化技术，使其在长上下文建模和开放式主观评估中表现出色。

 

\2.   预训练过程

InternLM2的预训练过程涵盖了多种数据类型，包括文本、代码和长上下文数据。模型首先在4k个tokens上训练，然后在预训练和微调阶段扩展到32k个tokens。在200k“Needle-in-a-Haystack”测试中，InternLM2表现出色，显示了其捕捉长期依赖关系的能力。“针在草堆”（Needle-in-a-Haystack）测试是一个用于评估长上下文处理能力的基准测试。在这个测试中，模型需要在非常长的文本中找到特定的信息或答案，这就像在一大堆草中找到一根针一样困难。

 

\3.   对齐技术

InternLM2采用了监督微调（SFT）和一种新的条件在线人类反馈强化学习（COOL RLHF）策略，以解决人类偏好的冲突和奖励欺骗问题。这一策略显著提高了模型在各种主观对话评估中的性能。

COOL RLHF（Conditional Online Reinforcement Learning from Human Feedback）是一种新颖的强化学习策略，用于优化大型语言模型（LLM）以更好地符合人类偏好。

传统的RLHF方法面临两个主要挑战：

偏好冲突：在开发对话系统时，模型需要既有用（提供有价值的信息）又无害（避免产生有害或不适当的内容），这两者往往难以同时满足。

奖励欺骗：模型可能通过捷径获得高分，而不是真正学习到预期的行为，导致模型在未预期的方式上最大化奖励，从而影响其有效性和可靠性。

因此COOL RLHF引入了条件奖励机制，通过不同的系统提示来处理不同领域的偏好冲突：

系统提示对不同领域的数据进行动态分配，使单一奖励模型能够优化多种偏好。数据涵盖对话、文章写作、诗歌、摘要、编程、数学等多个领域，使用240万对二值偏好对进行训练。为了减少数据集中易难样本的不平衡影响，引入了难度衰减系数到排序损失函数中，并加入了对奖励分布的对数障碍惩罚，从而提高模型的鲁棒性和一致性。

同时COOL RLHF采用了多轮在线RLHF策略，分为快速路径和慢速路径：

快速路径：快速识别并修正奖励欺骗行为，通过构建偏好对来提高奖励模型的可靠性。

慢速路径：覆盖最新的模型响应，进行一般性的奖励模型改进，通过人类标注提供更细致的反馈。

在RLHF对齐阶段，COOL RLHF采用标准的近端策略优化（PPO）算法，并进行了以下调整：

模型初始化：参考模型和actor模型从SFT模型权重初始化，评论者模型从奖励模型初始化，并进行预训练以稳定价值估计。

条件奖励：根据不同领域的查询，添加适当的条件系统提示来计算奖励分数。

预训练梯度：加入预训练损失以减轻PPO阶段的灾难性遗忘风险。

超参数：设置KL散度系数为0.01，actor模型和评论者模型的学习率分别为1e-6和5e-6，并采用保守的采样策略来平衡采样多样性和收敛速度。

通过三轮在线RLHF的优化，COOL RLHF显著提高了模型在主观对话评估中的性能，增强了模型的可靠性和对人类偏好的适应性。

\4.   基础设施

InternLM2使用InternEvo训练框架进行预训练、SFT和RLHF。该框架通过数据、张量、序列和流水线并行化，实现了在数千个GPU上的大规模模型训练，并通过混合精度训练和FlashAttention技术优化了GPU内存利用率。

 

\5.   性能评估

InternLM2在多个长上下文基准测试中表现出色。在L-Eval和LongBench评估中，InternLM2-Chat-20B-SFT模型在多个任务中取得了最佳表现。在编程任务和推理任务上，经过能力增强训练后的InternLM2模型也展示了显著的性能提升。

 

\6.   数据准备

报告详细描述了InternLM2的预训练数据准备过程，包括通用文本数据、领域特定数据和SFT数据。通过对预训练数据的精细处理，模型在编码、推理、问答和考试任务中展示了强大的能力。主要数据源是中文和英文网页数据，占总量的86.46%。虽然其他来源的数据量较小，但平均文档长度较长，内容质量较高。数据处理管道包括数据格式化、基于规则的过滤、数据去重、安全过滤和质量过滤。将数据标准化为指定格式，分类并存储为JSON Lines格式。设计启发式过滤规则，清理数据中的低质量内容，如解析错误、格式错误和非自然语言文本，使用局部敏感哈希（LSH）方法进行模糊去重，采用域名封锁、关键词封锁、色情内容分类器和毒性分类器进行过滤。采用广告分类器和流畅性分类器进一步过滤低质量内容。

 

\7.   结论

InternLM2在主观和客观评估中表现出色，适用于多种场景。报告开源了模型，还提供了详细的训练过程描述，包括预训练数据和对齐数据的准备方法。通过COOL RLHF策略，InternLM2在对齐人类偏好方面也取得了显著进展。

 

文档报告链接：InternLM2 技术报告https://arxiv.org/pdf/2403.17297.pdf