---
layout: post
title: Paper Review - SpeedySpeech
subtitle: Bài 5 TTS
cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [Speech Synthesis, TTS]
---

# Tổng quan

Bài tiếp theo trong chuỗi bài tìm hiểu TTS models, hôm nay sẽ là SpeedySpeech.
Bài này đánh vào hướng phát triển về fast training và fast inference, low resource requirements, trong khi vẫn giữ được synthesis quality tốt. 
Ở đây, bài báo đề xuất việc quay lại sử dụng student-teacher network, như FastSpeech, nhưng đơn giản hơn. 
Đặc biệt tác giả nhận định self-attention thì không nên lạm dụng nhiều, trong khi sử dụng toàn simple convolutional blocks với residual connections(non-sequential approach) ở cả teacher và student và chỉ một layer attention duy nhất ở teacher, cùng với sự giúp sức của Vocoder là MelGAN, bài này đủ sức ăn Tacotron 2 !?

- Teacher network là một autoregressive convolutional network, được train để predict next spectrogram frame given input phonemes (including punctuation) và past frames; it uses attention to keep track of the phoneme it is generating. The attention values are then used to align phonemes with spectrogram frames and extract phoneme durations
- The student network là một non-autoregressive - fully convolutional network. Cũng encodes input phonemes, nhờ vào additional alignment supervision từ teacher, student sẽ học và predicts the duration cho mỗi phoneme, sau đó decodes a mel-scale spectrogram dựa trên phoneme encodings and durations
