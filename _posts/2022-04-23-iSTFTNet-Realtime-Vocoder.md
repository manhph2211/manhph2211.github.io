---
layout: post
title: Paper Review - iSTFTNet
subtitle: Bài 9 TTS
cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [Speech Synthesis, TTS]
---

# Tổng quan

Hihi cũng đã lâu mình không viết về một model nào đó :D Lần này là một kiến trúc vocoder chứ không phải là một acoustic model nữa. Model này mới được ra mắt vào 3/2022 với mục tiêu recovery of the original-scale magnitude spectrogram, phase reconstruction, and frequency-to-time conversion. Đương nhiên, những yêu cầu này cũng không phải quá khó khăn đối với một số models được xuất bản gần đây. Những model này thường tiếp cận theo cách giải quyết ba vấn đề train một cách jointly và implicitly using a convolutional neural network, including temporal upsampling layers, when directly calculating a raw waveform. Rõ ràng hướng này sẽ giúp giảm thiểu những task dư thừa ví dụ như the direct reconstruction of high dimensional original-scale spectrograms, tuy nhiên, nhược điểm kéo theo là chúng ta không biết ba vấn đề trên thực sự được giải quyết thế nào! Nó như một black box vậy! Chúng ta thực sự quên đi employ the time frequency structures existing in the mel-spectrogram, chính xác là hướng đi giúp giảm số lượng tham số!

⇒ iSTFTNet, which employs time and frequency structures explicitly using iSTFT after sufficiently reducing the frequency dimension using some upsampling layer (Hình dưới)

⇒ Utilize the characteristics of STFT, that is, the trade-off between the time and frequency resolution

When the iSTFT required after s× upsampling is represented as iSTFT(fs, hs, ws), where fs, hs, and ws indicate the FFT size, hop length, and window length, respectively; iSTFT(fs, hs, ws) can be calculated using the parameters of iSTFT required for the original-scale spectrogram, iSTFT(f1, h1, w1)

⇒ Có thể giảm the frequency dimensions bằng cách tăng s. Khoan đã :) không thể cứ tăng s mãi đúng không :)) Người ta đã thí nghiệm để kiểm tra xem s ở mức nào là ngon nhất, hình dưới cho thấy cái trường hợp c, ứng với việc dùng two upsamples là ngon nhất!

Như vậy, thực chất đây không hẳn là một vocoder mới tinh LOL, chúng ta sẽ có thể cân nhắc việc áp dụng kỹ thuật này cho các vocoder hiện đại (HifiGan). Mọi người thử quan sát kiến trúc trong bài HifiGan:

⇒ Hoàn toàn có thể tạo ra một vocoder nhanh và nhẹ hơn nhiều! Còn speech quality thì hem bít :)

Chưa xong đâu ạ :D, bài báo này nhắc là cần xem xét thêm three minor but essential modifications:

(i) The output channels of the final convolutional layer should be changed from 1 to (fs/2 + 1) × 2 to generate magnitude and phase spectrograms instead of a raw waveform. 
(ii) Exponential and sine activation functions should be applied to the output of (i) when calculating the magnitude and phase spectrograms, respectively. 
Vì sao lại vậy: the required magnitude spectrogram uses a linear scale, whereas the input mel-spectrogram uses a log scale, and applies a sine activation function to represent the periodic characteristics of the phase spectrogram.
(iii) A raw waveform should be generated from the magnitude and phase spectrograms using iSTFT (Eq. (1)). 
