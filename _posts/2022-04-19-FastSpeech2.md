---
layout: post
title: Paper Review - FastSpeech2
subtitle: 
cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [Speech Synthesis, TTS]
---

# Tổng quan

Bây giờ mình sẽ viết tiếp đến FastSpeech 2. Đến bài này, tác giả chỉ ra FastSpeech đúng là đã có thể tổng hợp âm thanh nhanh hơn nhiều nhưng vẫn có độ chính xác cạnh tranh với các mô hình autoregressive lúc đó. Tuy nhiên, FastSpeech lại có thể gặp một số vấn đề:
- Sử dụng teacher-student distillation pipeline khá phức tạp và tốn thời gian
- Sử dụng duration extracted từ teacher model có thể chưa đủ chính xác, và target mel-spectrograms sai distilled từ teacher model có thể dẫn đến information loss

Nhắc lại về TTS là một bài toán giải quyết vấn đề one-to-many mapping, cụ thể là multiple possible speech sequences can correspond to a text sequence due to variations in speech, such as pitch, duration, sound volume, and prosody. 

Tiếp theo mình nhắc lại một chút về:
- Phoneme duration, which represents how long the speech voice sounds
- Pitch, which is a key feature to convey emotions and greatly affects the speech prosody
- Energy, which indicates the frame-level magnitude of Mel-spectrograms and directly affects the volume and prosody of speech.

Trong non-autoregressive TTS, thì nếu chỉ xem xét đầu vào liên quan tới text thì có vẻ là chưa đủ để fully predict the variance in speech. Khi đó, model có khả năng đã bị overfit với variations of the target speech in the training set, và dẫn đến poor generalization ability. FastSpeech alleviates the one-to-many mapping problem by knowledge distillation nhưng dẫn tới information loss. Như vậy, FastSpeech 2 ra đời và được chứng minh là đã nhanh hơn, chính xác hơn, training đơn giản hơn so với FastSpeech và các autoregressive models. Lần này với những đóng góp chính sau:
- Directly training the model with ground-truth target instead of the simplified output from teacher, and 
- Introducing more variation information of speech (e.g., pitch, energy, and more accurate duration) as conditional inputs

Sau đây mình sẽ trình bày kĩ hơn về FastSpeech 2 và cả FastSpeech 2s(fully end-to-end text-to-waveform synthesis - directly generate speech waveform from the text in parallel)

# FastSpeech 2

Bây giờ hãy xem thử các thành phần chính của model. Đầu tiên, FastSpeech 2 sử dụng a feed-forward Transformer block (như trong FastSpeech) - encoder converts the phoneme embedding sequence sâng the phoneme hidden sequence, sau đó sẽ có bộ variance adaptor xử lý các thông tin như duration, pitch, and energy into the hidden sequence. Cuối cùng thì decoder converts the adapted hidden sequence sang Mel-spectrogram sequence in parallel. 

Ở hình b trong bài báo, ta thấy đặc biệt có khối variance adaptor với sự xuất hiện của duration predictor, pitch predictor, và energy predictor. Cả ba đều có kiến trúc giống nhau, bao gồm 2-layer 1D-convolutional network với ReLU activation, theo sau bởi layer normalization, dropout layer, và extra linear layer. cả ba nhánh đều sử dụng MSE loss cho việc optim.

Với Duration Predictor, ta sử dụng nhãn ở đây sẽ lấy từ MFA, thay vì autoregressive teacher model. Output sẽ là the length of each phoneme in the logarithmic domain. Có thể thấy thông tin về duration sẽ được tận dụng, và có ý nghĩa đối với các predictor sau :)) Với pitch predictor, để predict the variations in pitch contour, bài báo sử dụng continuous wavelet transform (CWT) để decompose the continuous pitch series into pitch spectrogram và training với nhãn là pitch spectrogram (quantize pitch F0 of each frame to 256 possible values in log-scale and further convert it into pitch embedding vector). Với energy predictor thì cũng khá giống với pitch predictor, tính toán L2-norm của the amplitude STFT frame as the energy sau đó quantizing energy of each frame to 256 possible values uniformly, encoded it into energy embedding.
