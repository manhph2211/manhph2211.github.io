---
layout: post
title: Paper Review - Portaspeech
subtitle: Bài 8 TTS
cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
tags: [Speech Synthesis, TTS]
---

# Tổng quan

Có một sự thật là những model TTS hiện đại cần phải đáp ứng được những tiêu chí khắt khe hơn, có thể kể ra như: việc đáp ứng được real-time applications (inference time phải super fast), hay cân nhắc việc minimize model size để có thể deploy sản phẩm lên các edge devices hoặc điện thoại chẳng hạn, và đương nhiên không thể thiếu, là việc phải tạo ra speech thật chính xác, tự nhiên, đa dạng với powerful prosody support! PortaSpeech là một model rất mới, ra đời đầu năm 2022, và chắc chắn, nó phải cố gắng để đạt được những tiêu chí trên. 

Well, đây là bài thứ 8 trong serie tìm hiểu TTS models cũng như cách tiếp cận lấy alignments trong TTS. Như đã nói thì nó là một model mới, rất mới… Hãy cùng tìm hiểu model này có gì thú vị nhé mọi người :)

# Background

Bây giờ hãy xem có những gì cần lưu ý trước khi ta đi sâu vào model này. Trước hết, chắc hẳn mọi người không quên VAE đúng không, nó là một generative model với cấu trúc khá quen thuộc rồi. Gần đây, người ta đã cố gắng để apply các tiếp cận VAE cho mô hình TTS và thật thú vị khi hầu hết các model này sẽ có thể capture được dynamism and variability of ground-truth prosody kể cả khi model size nhỏ, trong khi mel gen ra được có chất lượng khá thấp (mờ, hay kiểu over-smoothing).

Tiếp theo, nếu mọi người đã tìm hiểu qua về model GlowTTS, thì cũng đã biết nó không còn phụ thuộc vào một kẻ ngoại bang nào cả, khi nói đến aligner LOL. Nó sử dụng cách tiếp cận flow-based, cụ thể là directly searches for the most probable monotonic alignment between text and the latent representation of speech using normalizing flows and dynamic programming. Chính xác là MAS là thứ gánh cho model này! Tuy nhiên, hãy để ý là những model flow-based NAR-TTS này có thể fast, diverse và controllable speech synthesis, không sợ mel mờ như bên trên, nhưng cũng kéo theo một nhược điểm, nó đòi hỏi sử dụng huge model capacity, và thực sự model size rất nhạy cảm với performance, theo hướng tiêu cực ạ!

Như vậy tại sao lại cần đề cập hai mục trên? Ở đây PortaSpeech sẽ tận dụng sức mạnh của hai ông này, VAE and normalizing flow thực sự đề xuất một model nhẹ mà vẫn generate mel-spectrograms with rich details and expressive prosody. Ngoài ra, không thể không nhắc đến việc đề xuất sử dụng mixture alignment in the linguistic encoder, which improves the prosody and reduces the dependence on fine-grained (phoneme-level) hard alignment, và introducing the grouped parameter sharing mechanism to the flow-based post-net...
