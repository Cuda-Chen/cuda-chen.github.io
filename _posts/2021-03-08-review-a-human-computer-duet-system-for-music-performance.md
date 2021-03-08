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
![Fig 1](/assets/images/2021/03/08/figure_1.png)

1. Proposed Virtual Musician System
2. Real-time Music Tracking
3. Automatic Animation
4. Discussion

## 1. Proposed Virtual Musician System
![Fig 2](/assets/images/2021/03/08/figure_2.png)

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
![Fig 3](/assets/images/2021/03/08/figure_3.png)

- A re-implementation of the *'Any time' music tracker* by by Arzt and Widmer [2, 3].
- The music detector starts tracking when performance begins.
- The rough position estimator returns a collection of possible positions of the live
piano signal which complies to the rehearsed piano.
- The decision maker chooses an ODTW thread as the most trusted one then output the
tracking result.

## 3. Automatic Animation
- Pose estimation is fulfilled by using [4] to extract the 3-D
position of the violinistsâĂŹ 15 body joints.
- Body movement generation, not surprisingly, needs a dataset which contains audio
as well as pose contents of violin performance for modeling the music-to-body-movement correspondence.
    - Talking to generating the movement, they extend the framework of 
    audio-to-body (A2B) dynamics [5] into 3-dimension as shown below.
![Fig 4](/assets/images/2021/03/08/figure_4.png)

## 4. Discussion
- Thought audiences responsed the performance was insightful, there are no any
methods to make the objective evaluation of the system. As a result, they dipict
their quality assessment approach.
    - The ways to improvement the system will be also discussed.

### Real-time Music Tracking Evaluation
- The intuition is to measure the deviation between online music tracking and
*offline alignment with the live recording* with the following procedures:
    1. Use a multi-channel recoding device to record both live piano and violin
    autio.
    2. Compare the time mapping of the recorded live piano with the referenced
    piano MIDI via offline synchronization as shown below.
![Fig 8](/assets/images/2021/03/08/figure_8.png)
    3. Compare the time mapping of the live violin ad the estimated violin
    via offline synchronization, too.
- They use the following four criteria for evaluation:
    - normal speed (115-145 bpm)
    - slow speed (90-120 bpm)
    - fast speed (135-175 bpm) 
    - accelerando speed starting with around 80bpm and
    ending with 160 bpm
- For all the four cases, the system takes around four measures for synchronizing
with the live piano. Once it synchronizes with the human pianist, the value of
deviation is ±0.25 beats.
- Also, we can observe that a sudden alteration of speed doesn't always result
in an alteration of latency.
    - As a matter of fact, the structure of music relates to latency.

### Body Movement Generation Evaluation
- The model is trained and evaluated on a dataset which contains 140 violin solo 
videos with total length of 11 hours.
    - For training, they select the videos from one of the violinists,
    and conduct a 14-fold cross validation to train the 14 models.
    - Each model is evaluated on the leftout piece (for that fold partition).
    - The results is averaged by number of the violinists.
- L1 distance and bowing attack F1-score are used for metrics.
    - L1 distance is defined by the distance between the generated and the
    ground-truth joints.
    - In this paper, the bowing attack means s the time instance when the bowing
    direction changes.
    - After they identify g all the true positive, false positive, and false negative
    of the bowing attack, they calculate the F1-score.
- We can observe the F1-scores are much better than the baseline, meaning the model
is effective.
- There are still two improvements: the deviation of the right hand position and
the timing of the bowing attack.

## Reference
[1] https://arxiv.org/abs/2009.07816

[2] Andreas Arzt and Gerhard Widmer. Simple tempo models for real-time music
tracking. In Proceedings of the Sound and Music Computing Conference (SMC),
2010

[3] Andreas Arzt and Gerhard Widmer. Towards effective ‘any-time’ music tracking.
In Proceedings of the Fifth Starting AI Researchers’ Symposium, page 24, 2010.

[4] Dario Pavllo, Christoph Feichtenhofer, David Grangier, and Michael Auli. 3d
human pose estimation in video with temporal convolutions and semi-supervised
training. In Proceedings of the IEEE Conference on Computer Vision and Pattern
Recognition (CVPR), pages 7753–7762, 2019.

[5] Eli Shlizerman, Lucio Dery, Hayden Schoen, and Ira Kemelmacher-Shlizerman.
Audio to body dynamics. In Proceedings of the IEEE Conference on Computer
Vision and Pattern Recognition (CVPR), pages 7574–7583, 2018.
