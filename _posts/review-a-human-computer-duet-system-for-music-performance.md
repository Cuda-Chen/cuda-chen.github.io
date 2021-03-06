---
layout: post
title: "Review -- A Human-Computer Duet System for Music Performance (Automatic Accompaniment)"
category: [paper review]
tags: [machine learning, 
       automatic accompaniment,
       body movement generation,
       animation,
       computer-human interaction,
       music information retrieval
      ]
---

In this post, **A Human-Computer Duet System for Music Performance** [1], by
Academia Sinica, is reviewed. In this paper:
- **A virtual violinist** is created who can perform chamber music with
a human pianist that **does not need any intervention**.
- The proposed system **has performed in a public concert**.
- Discussions of effective improvement of the system is presented.

This work is supported by the Automatic Music Concert Animation (AMCA) project
of the Institute.

## Outline
![Fig 1]()

1. Proposed Virtual Musician System
2. Real-time Music Tracking
3. Automatic Animation
4. Discussion

## 1. Proposed Virtual Musician System
![Fig 2]()

- The system has the capabilities to synchronize the reference, rehearsed, and live contents
altogether.
- In the beginning, the person who is going to perform with virtual violinist selects
a music piece.
    - The reference and live piano audio are aligned using the offline DTW algorithm.
- In live performance, the real-time music tracker **follows the live piano audio** with
the help of online DTW (ODTW) alignment.
    - The music tracker then creates the visual animation and sound synthesis in order to
    generate the body movement and violin music of virtual violinist.
- Two approaches for generating motion ared used by automatic animation: pose estimation
and music-to-motion generation techniques.
- At last, audio and motion of the virtual violinist are rendered with real-time sound
synthesizer and animator.

## 2. Real-time Music Tracking
![Fig 3]()

- A re-implementation of the *'Any time' music tracker* by by Arzt and Widmer [2, 3].
- The music detector starts tracking when performance begins.
- The rough position estimator returns a collection of possible positions of the live
piano signal which complies to the rehearsed piano.
- The decision maker chooses an ODTW thread as the most trusted one then output the
tracking result.

## 3. Automatic Animation

## 4. Discussion

## Reference
[1] https://arxiv.org/abs/2009.07816

[2] Andreas Arzt and Gerhard Widmer. Simple tempo models for real-time music
tracking. In Proceedings of the Sound and Music Computing Conference (SMC),
2010

[3] Andreas Arzt and Gerhard Widmer. Towards effective ‘any-time’ music tracking.
In Proceedings of the Fifth Starting AI Researchers’ Symposium, page 24, 2010.
