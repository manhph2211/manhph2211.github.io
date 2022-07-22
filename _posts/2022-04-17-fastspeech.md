---
layout: post
title: Paper Review - FastSpeech
subtitle: 
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [Speech Synthesis, TTS]
---

# Tổng quan

Hôm trước mình đã viết về Tacotron 2, đầu tiên sẽ sử dụng một mạng với cách tiếp cận autoregressive và tận dụng sức mạnh của Location Sensitive Attention tạo ra mel-spectrograms với alignments hiệu quả từ text, rồi từ đó tổng hợp âm thanh nhờ vào modified wavenet. Tuy nhiên, thực tế đã cho thấy, ngay cả những phương pháp với concatenative and statistical parametric approaches thì cũng có tốc độ infer nhanh hơn. Ngoài ra Tacotron 2 cũng được cho là không thực sự mạnh mẽ LOL, cụ thể là một số words có thể bị skipped hoặc repeated, hay là không thực sự có controllability về voice speed, và prosody. Từ đó, FastSpeech ra đời với nhiệm vụ giải quyết các vấn đề trên. FastSpeech cũng lấy đầu vào là phoneme sequence và gen mel-spectrograms một cách none autoregressive, đặc biệt cải thiện tốc độ inference, cũng như các vấn đề về robustness hay controllable requirements! Sau đây sẽ là các phần chi tiết hơn về FastSpeech.

# FastSpeech

Như đã trình bày ở trên, FastSeech sử dụng cách tiếp cận non autoregressive, thay vì sử dụng encoder-attention-decoder based architecture, thì dùng feedforward structure để generate a target mel spectrogram sequence in parallel...
