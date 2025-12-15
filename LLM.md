# LLM

## Table of content

- [LLM](#llm)
  - [Table of content](#table-of-content)
  - [Training](#training)
  - [LLM Design patterns](#llm-design-patterns)
    - [Reflection pattern](#reflection-pattern)
    - [Tool Use](#tool-use)
    - [Reason and Act](#reason-and-act)
    - [Planning Agent](#planning-agent)
    - [Task agent](#task-agent)

## Training

Having an enormous source of data is an expensive solution. Only big sompanies can afford it. ==> A lot of OpenSource models are trained via distillation. You have a big model that generates answers, you train a smaller model to try have the same answers. You train by imitation.

New idea to train multi-modal models: instead of making captions of images for training, have people speak for 60 seconds about the image.

Instead of drawing bounding boxes, we just "point" at objects. Makes it easier for the model to point to items & much faster to create a dataset.

## LLM Design patterns

### Reflection pattern

You ask a question and generate an answer. You then proceed to criticize and improve the answer in N rounds of reflection until finally the LLM is fully satisfied with the answer.

### Tool Use

A first LLM chooses which tool to use and which query to do with it. A second LLM takes the result from the tool to generate an answer. Example: RAG.

### Reason and Act

A first LLM can reason and act subsequent questions. It is a mix of the previous patterns.

### Planning Agent

We ask for a plan, which step to do in which order and which ones need to be finished before going to the next one. Careful with two things: planning might be useless for simple tasks. If failure, need to replan.

### Task agent

We have an LLM planner that either coordinates a list of agents or a mix of agents and deterministic tools. Dangerous to have too many agents => We take the risk of diluting responsabilities / not clear. Having a limited number of agents simplifies communication.
