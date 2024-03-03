---
layout: post
title: brief review -- yolov9
category:
tags:
mathjax: true 
---
In this post, **yolov9** [^yolov9], by Academia Sinica,
is reviewed. In this paper:

- Propose **Programmable Gradient Information (PGI)**
to solve information bottleneck.
- Propose **Generalized ELAN (GELAN)**.
- The yolov9 outperforms SOTA models whlist trained
from scratch and just convension convolutionanl layers.

## Outline

- information bottleneck
- PGI
- GELAN
- ablation studies

## Information Bottleneck

- In deep networks, the phenomenon of input data losing
information during the feedforward process is called
**information bottleneck**.
- There are three mitigations to deal with this problem (with drawbacks listed):
  1. reversible architecture: uses repeated data in an explicitly way
    - requires additional layers hence increase inference cost
    - hard to model high-order semantic information during training
  2. masked modeling: uses reconstruction loss and adopts am implicit way to maximize the extracted features and retain the input information
    - reconsturction loss sometimes conflict with target loss
    - most this method produces incorrect associations with data
  3. introduction of the deep supervision concept: uses shallow features that have not lost too much vital information to pre-establish a mapping from features to target
    - prodcues error accumulation, and the problem become worse when training on small and difficult tasks

## PGI

- To address the information bottleneck issue, PGI is proposed.
- PGI is composed of three parts [figure here]:
  - main branch
  - auxiliary reversible branch
  - multi-level auxiliary information
- The main thought use auxiliary reversible branch to generate
reliable gradients, so that key characteristics of the deep
features can still be maintained.
- No extra cost is needed as the reversible architecture of PGI
is built on auxiliary branch.
- PGI also conquers two problems of:
  - deep supervision mechanism by allowing setting with various sizes.
  - mask modeling by permitting setting loss function freely.

### auxiliary reversible branch

- Adding reversible architecture to main branch leads to consuming a larget amount
of inference cost.
- The author realizes the reversible branch should not be accessed
in inference mode as our goal is to use reversible architecture to
obtain reliable gradients, and the author thinks the auxiliary reversible
branch should be the extend part of deep supervision branch.
- Thinking from this, auxiliary reversible branch does not
force the main branch to retain complete original information. Rather,
the main branch just needs to update itself by generating useful gradient.
  - Based on this design, the auxiliary reversible branch can be also
applied on shallower networks.
  - Moreover, there will be no impact on inference speed because 
auxiliary reversible branch can be removed in such scene.
  - Of course, it is possible to choose arbitrary reversible
architecture in PGI to play the role of auxiliary reversible branch.

### multi-level auxiliary information

- For higher-order semantic task such as object detection, it has
been well-proven that feature pyramid network is helpful for
detecting objects of different sizes.
  - Nevertheless, it causes the deep feature pyramids to lose a lot
of information for assisting predict the target object.
  - The author consider each feature pyramid needs to receive information
about all target objects, thus the subsequent main-branch can still
get the entire information for learning predictions for a variety of targets.
- The concept of multi-level auxiliary information is to insert an integration network between the feature pyramid network of auxiliary supervision and the main branch. Then, we use it to combine returned gradient from different prediction head.

## GELAN

## References

[^yolov9]: https://arxiv.org/abs/2402.13616 
