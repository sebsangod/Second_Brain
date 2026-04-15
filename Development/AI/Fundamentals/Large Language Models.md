---
aliases:
  - Large Language Models
tags:
  - learning
---
**Date**: 19/mar/2026
**Update**: 19/mar/2026
**Sources**: [Cloudflare](https://www.cloudflare.com/learning/ai/what-is-large-language-model/)

**Tags:** #AI #LLM #model

**Related:** [[Artificial Intelligence]], [[Dataset]], [[Machine Learning]], [[Transformer]], [[Deep Learning]], [[Neural Networks]], [[Fine-tunning]], [[Claude]], [[Claude Code]], [[OpenAI]], [[ChatGPT]], [[GitHub Copilot]], [[Qwen]], [[Alibaba]], [[Llama]], [[Meta]]

---

## Description

A large language model (LLM) is a type of `AI` program that can recognize and generate text, among other tasks.

In simpler terms, an LLM is **a computer program that has been fed enough examples to be able to recognize and interpret human language or other types of complex data**.

---

## Details

**LLMs are trained on huge `sets of data` — hence the name "large." LLMs are built on `machine learning`: specifically, a type of `neural network` called a `transformer` model.** Many LLMs are trained on data that has been gathered from the Internet — thousands or millions of gigabytes' worth of text.

**LLMs use a type of `machine learning` called `deep learning` in order to understand how characters, words, and sentences function together.** Deep learning involves the probabilistic analysis of unstructured data, which eventually enables the deep learning model to recognize distinctions between pieces of content without human intervention.

**LLMs are then further trained via tuning: they are fine-tuned or prompt-tuned to the particular task** that the programmer wants them to do, such as interpreting questions and generating responses, or translating text from one language to another.


### What are LLMs used for?

LLMs can be trained to do a number of tasks. **One of the most well-known uses is their application as generative AI**: when given a prompt or asked a question, they can produce text in reply. The publicly available LLM `ChatGPT`, for instance, can generate essays, poems, and other textual forms in response to user inputs.

**Any large, complex data set can be used to train LLMs, including programming languages.** Some LLMs can help programmers write code. They can write functions upon request — or, given some code as a starting point, they can finish writing a program. LLMs may also be used in:

- Sentiment analysis
- DNA research
- Customer service
- Chatbots
- Online search


### What are some advantages and limitations of LLMs?

A key characteristic of LLMs is **their ability to respond to unpredictable queries.** An LLM can respond to natural human language and use data analysis to answer an unstructured question or prompt in a way that makes sense. Whereas a typical computer program would not recognize a prompt like "What are the four greatest funk bands in history?", an LLM might reply with a list of four such bands, and a reasonably cogent defense of why they are the best.

In terms of the information they provide, however, LLMs can only be as reliable as the data they ingest. If fed false information, they will give false information in response to user queries. LLMs also sometimes "[hallucinate](https://www.cloudflare.com/learning/ai/what-are-ai-hallucinations/)": they create fake information when they are unable to produce an accurate answer. For example, in 2022 news outlet Fast Company [asked](https://www.fastcompany.com/90819887/how-to-trick-openai-chat-gpt) ChatGPT about the company Tesla's previous financial quarter; while ChatGPT provided a coherent news article in response, much of the information within was invented.

In terms of [security](https://www.cloudflare.com/the-net/ai-secure/), user-facing applications based on LLMs are as prone to bugs as any other application. LLMs can also be manipulated via malicious inputs to provide certain types of responses over others — including responses that are dangerous or unethical. Finally, one of the [security problems with LLMs](https://www.cloudflare.com/the-net/vulnerable-llm-ai/) is that users may upload secure, confidential data into them in order to increase their own productivity. But LLMs use the inputs they receive to further train their models, and they are not designed to be secure vaults; they may expose confidential data in response to queries from other users. Learn more about the [best ways to secure LLMs.](https://www.cloudflare.com/learning/ai/what-is-ai-security/)


### How developers can quickly start building their own LLMs

To build LLM applications, developers need easy access to multiple `data sets`, and they need places for those data sets to live. Both cloud storage and on-premises storage for these purposes may involve infrastructure investments outside the reach of developers' budgets. Additionally, training `data sets` are typically stored in multiple places, but [moving that data](https://www.cloudflare.com/the-net/cloud-egress-fees-challenge-future-ai/) to a central location may result in massive [egress fees](https://www.cloudflare.com/learning/cloud/what-are-data-egress-fees/).

---

## Examples

* `ChatGPT` (from `OpenAI`)
* `Claude` (from `Anthropic`)
* `Qwen` (from `Alibaba`)
* `Llama` (from `Meta`)
* `Copilot` (from `GitHub`) is another example, but for coding instead of natural human language.

---