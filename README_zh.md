<p align="center">
    <!-- community badges -->
    <a href="https://discord.gg/G2rSbAf8uP"><img src="https://img.shields.io/badge/加入-Discord-blue.svg"/></a>
    <!-- license badge -->
    <a href="https://github.com/Eladlev/AutoPrompt/blob/main/LICENSE">
        <img alt="License" src="https://img.shields.io/badge/许可证-Apache_2.0-blue.svg"></a>
</p>

# 📝 AutoPrompt

**AutoPrompt 是一个提示词优化框架，旨在针对真实世界用例增强和完善您的提示词。**

该框架能自动生成高质量、细节丰富的提示词，以精准匹配用户意图。它采用一种精细化校准流程，通过迭代方式构建包含复杂边界案例的数据集，并据此优化提示词。此方法不仅减少了手动进行提示词工程的工作量，还有效解决了提示词[敏感性](https://arxiv.org/abs/2307.09009)和固有[模糊性](https://arxiv.org/abs/2311.04205)等常见问题。

**我们的使命：** 借助大语言模型的力量，赋能用户生成高质量、鲁棒的提示词。

# 为什么选择 AutoPrompt？
- **提示词工程的挑战：** LLMs 的质量在很大程度上取决于所使用的提示词。即使是[微小的改动](#提示词敏感性示例)也可能显著影响其性能。
- **基准测试的挑战：** 为生产级提示词创建基准测试通常劳动密集且耗时。
- **可靠的提示词：** AutoPrompt 能生成鲁棒的高质量提示词，使用最少的数据和标注步骤，提供可衡量的准确性和性能提升。
- **模块化与适应性：** 框架核心采用模块化设计，可与 LangChain、Wandb 和 Argilla 等流行的开源工具无缝集成，并能适应各种任务，包括数据合成和提示词迁移。

## 系统概述

![系统概述](./docs/AutoPrompt_Diagram.png)

该系统为诸如内容审核这类常受数据分布不平衡挑战的真实场景而设计。系统实现了[基于意图的提示词校准](https://arxiv.org/abs/2402.03099)方法。流程始于用户提供的初始提示词和任务描述，可选择性地包含用户示例。优化过程迭代地生成多样化的样本，通过用户或LLM进行标注，并评估提示词性能，随后由一个LLM建议改进后的提示词。

通过首先设计一个排序器提示词，然后利用这个学习到的排序器执行提示词优化，此优化过程可以扩展到内容生成任务。优化在达到预算或迭代限制时结束。

这种联合合成数据生成和提示词优化的方法优于传统方法，同时只需最少的数据和迭代次数。了解更多，请参阅我们的论文 [基于意图的提示词校准：利用合成边界案例增强提示词优化](https://arxiv.org/abs/2402.03099)，作者 E. Levi 等 (2024)。

**使用 GPT-4 Turbo 时，此优化通常只需几分钟即可完成，成本低于 1 美元。** 为了管理与 GPT-4 LLM 令牌使用相关的成本，该框架允许用户为优化设置预算上限（以美元或令牌数为单位），配置方法如[此处](docs/examples.md#steps-to-run-example)所示。

## 演示

![管道录制](./docs/autoprompt_recording.gif)

## 📖 文档
- [如何安装](docs/installation.md) （设置说明）
- [提示词优化示例](docs/examples.md) （用例：电影评论分类、生成和聊天审核）
- [工作原理](docs/how-it-works.md) （流程解释）
- [架构指南](docs/architecture.md) （主要组件概述）

## 特性
- 📝 以最少的数据和标注步骤提升提示词质量。
- 🛬 为生产用例设计，如内容审核、多标签分类和内容生成。
- ⚙️ 支持在不同模型版本或 LLM 提供商间无缝迁移提示词。
- 🎓 支持提示词压缩。将多条规则合并为单个高效的提示词。

## 快速开始
AutoPrompt 需要 `python <= 3.10`
<br />

> **步骤 1** - 下载项目

```bash
git clone git@github.com:Eladlev/AutoPrompt.git
cd AutoPrompt
```

<br />

> **步骤 2** - 安装依赖

根据您的偏好使用 Conda 或 pip。使用 Conda：
```bash
conda env create -f environment_dev.yml
conda activate AutoPrompt
```

使用 pip：
```bash
pip install -r requirements.txt
```

使用 pipenv：
```bash
pip install pipenv
pipenv sync
```

<br />

> **步骤 3** - 配置您的 LLM

通过更新配置文件 `config/llm_env.yml` 来设置您的 OpenAI API 密钥。
- 如需帮助查找您的 API 密钥，请访问此[链接](https://help.openai.com/en/articles/4936850-where-do-i-find-my-api-key)。
- 我们推荐使用 [OpenAI 的 GPT-4](https://platform.openai.com/docs/guides/gpt) 作为 LLM。我们的框架也支持其他提供商和开源模型，如[此处](docs/installation.md#configure-your-llm)所述。

<br />

> **步骤 4** - 配置您的标注器
- 为您的项目选择一种标注方法：
    - 我们推荐从“人在回路”方法开始，利用 [Argilla](https://docs.v1.argilla.io/en/v1.11.0)。请注意，AutoPrompt 兼容 **Argilla V1**，而非最新的 V2。请遵循 [Argilla 设置说明](https://docs.v1.argilla.io/en/v1.11.0/getting_started/quickstart_installation.html)，但需进行以下修改：
        - 如果使用本地 docker，请使用 `v1.29.0` 标签而非 `latest` 标签。
        - 如需通过 HF 快速设置，请复制此 [空间](https://huggingface.co/spaces/Eladlev/test4)。
    - 或者，您也可以通过以下[配置步骤](docs/installation.md#configure-llm-annotator)设置一个 LLM 作为您的标注器。
- 用于评估提示词性能的默认预测器 LLM（GPT-3.5）在 `config/config_default.yml` 文件的 `predictor` 部分配置。
- 在输入的配置 yaml 文件中使用 `max_usage` 参数定义您的预算。对于 OpenAI 模型，`max_usage` 设置最大花费（美元）。对于其他 LLMs，它限制最大令牌数。

<br />

> **步骤 5** - 运行流程

首先，通过编辑 `config/config_default.yml` 来配置您的标签：
```
dataset:
    label_schema: ["是", "否"]
```

对于**分类流程**，请在相应的工作目录下使用以下终端命令：
```bash
python run_pipeline.py
```
如果初始提示词和任务描述没有直接作为输入提供，系统将引导您输入这些详细信息。或者，您也可以将它们指定为命令行参数：
```bash
python run_pipeline.py \
    --prompt "这条电影评论是否包含剧透？回答 是 或 否" \
    --task_description "助手是一个专家分类器，负责对电影评论进行分类，并告知用户它是否包含所评论电影的剧透。" \
    --num_steps 30
```
您可以使用 [W&B](https://wandb.ai/site) 仪表板跟踪优化进度，设置说明可在此[找到](docs/installation.md#monitoring-weights-and-biases-setup)。

如果您使用 pipenv，请确保激活环境：
``` bash
pipenv shell
python run_pipeline.py  
```
或者，在命令前加上 `pipenv run`：
```bash
pipenv run python run_pipeline.py 
```

#### 生成流程
要运行生成流程，请使用以下示例命令：
```bash
python run_generation_pipeline.py \
    --prompt "撰写一篇关于特定电影的优质、全面的影评。" \
    --task_description "助手是一个被要求撰写电影评论的大语言模型。"
```
更多信息，请参考我们的[生成任务示例](docs/examples.md#generating-movie-reviews-generation-task)。

<br />

享受成果。完成这些步骤后，您将获得一个为您的任务量身定制的**精细化提示词**，以及一个包含挑战性样本的**基准测试集**，它们存储在默认的 `dump` 路径中。

## 提示与技巧

- 优化过程中提示词的准确性可能会有波动。为了找到最佳提示词，我们建议在初始生成基准测试集后进行持续优化。使用 `--num_steps` 设置优化迭代次数，并通过在 `dataset` 部分指定 `max_samples` 来控制样本生成。例如，设置 `max_samples: 50` 和 `--num_steps 30` 将基准测试集限制在 50 个样本，假设每次迭代生成 10 个样本，则允许进行 25 次额外的优化迭代。
- 框架支持检查点功能，便于从上次保存的状态轻松恢复优化。它会自动将最近的优化状态保存在 `dump` 路径中。使用 `--output_dump` 设置此路径，并使用 `--load_path` 从检查点恢复。
- 迭代过程包括多次调用 LLM 服务，涉及长提示词和请求 LLM 生成相对大量的令牌。这可能需要一些时间（尤其是在生成任务中），请耐心等待。
- 如果 Argilla 服务器连接/出现错误，请尝试重启空间。

## 提示词敏感性示例
假设您编写了一个用于识别电影剧透的提示词：
```
审阅所提供的内容，并指出其是否包含任何重要的剧情透露或关键点，这些内容可能揭示故事或其结局的重要元素。如果包含此类剧透或关键见解，则回应“是”；如果避免揭示关键故事元素，则回应“否”。
```
使用 GPT-4 LLM，该提示词在您的[基准测试](docs/examples.md#filtering-movie-reviews-with-spoilers-classification-task)上得分为 81。然后，您做了一个微小的修改：
```
审阅文本并确定其是否提供了关于故事的基本启示或关键细节，这些内容将构成剧透。如果存在剧透，则回应“是”；如果不存在，则回应“否”。
```
令人惊讶的是，第二个提示词得分仅为 72，准确率下降了 11%。这说明了谨慎进行提示词工程过程的必要性。

## 🚀 贡献

我们非常感激您的贡献！如果您渴望贡献力量，请参阅我们的[贡献指南](docs/contributing.md)以获取详细信息。

如果您想了解我们的未来计划，请访问我们的项目路线图。
如果您希望加入我们的旅程，我们邀请您通过我们的 [Discord 社区](https://discord.gg/G2rSbAf8uP)与我们联系。我们很高兴您的加入！

## 🛡 免责声明

AutoPrompt 项目按“原样”提供，不作任何明示或暗示的保证或担保。

我们对提示词优化和使用的看法：

1.  AutoPrompt 的核心目标是优化和完善提示词以获得高质量的结果。这是通过迭代校准过程实现的，有助于减少错误并提高 LLMs 的性能。然而，该框架不能保证在所有情况下都绝对正确或无偏见。
2.  AutoPrompt 旨在提高提示词的可靠性并缓解敏感性问题，但并不声称能完全消除此类问题。

请注意，使用由 AutoPrompt 支持的 LLMs（如 OpenAI 的 GPT-4）可能会因令牌使用而产生显著成本。使用 AutoPrompt 即表示您承认自己有责任监控和管理您的令牌使用及费用。我们建议定期检查您的 LLM 提供商的 API 使用情况，并设置限制或警报以防止意外收费。
为了管理与 GPT-4 LLM 令牌使用相关的成本，该框架允许用户为优化设置预算上限（以美元或令牌数为单位），配置方法如[此处](docs/examples.md#steps-to-run-example)所示。

## 引用

如果您在研究中使用了我们的代码，请引用我们的[论文](https://arxiv.org/abs/2402.03099)：

```
@misc{2402.03099,
Author = {Elad Levi and Eli Brosh and Matan Friedmann},
Title = {Intent-based Prompt Calibration: Enhancing prompt optimization with synthetic boundary cases},
Year = {2024},
Eprint = {arXiv:2402.03099},
}
```

## 许可证

此框架根据 [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0) 许可。

## ✉️ 支持 / 联系我们
- [社区 Discord](https://discord.gg/G2rSbAf8uP)
- 我们的邮箱：[autopromptai@gmail.com‬](mailto:autopromptai@gmail.com)

---