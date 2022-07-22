---
layout: post
title: Paper Review - Portaspeech
subtitle: Bài 8 TTS
cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [Speech Synthesis, TTS]
---

# Tổng quan

Có một sự thật là những model TTS hiện đại cần phải đáp ứng được những tiêu chí khắt khe hơn, có thể kể ra như: việc đáp ứng được real-time applications (inference time phải super fast), hay cân nhắc việc minimize model size để có thể deploy sản phẩm lên các edge devices hoặc điện thoại chẳng hạn, và đương nhiên không thể thiếu, là việc phải tạo ra speech thật chính xác, tự nhiên, đa dạng với powerful prosody support! PortaSpeech là một model rất mới, ra đời đầu năm 2022, và chắc chắn, nó phải cố gắng để đạt được những tiêu chí trên. 

Well, đây là bài thứ 8 trong serie tìm hiểu TTS models cũng như cách tiếp cận lấy alignments trong TTS, cũng là bài em sẽ trình bày cho buổi seminar sắp tới - PortaSpeech. Như đã nói thì nó là một model mới, rất mới… Hãy cùng tìm hiểu model này có gì thú vị nhé mọi người :)

# Background

Bây giờ hãy xem có những gì cần lưu ý trước khi ta đi sâu vào model này. Trước hết, chắc hẳn mọi người không quên VAE đúng không, nó là một generative model với cấu trúc khá quen thuộc rồi. Gần đây, người ta đã cố gắng để apply các tiếp cận VAE cho mô hình TTS và thật thú vị khi hầu hết các model này sẽ có thể capture được dynamism and variability of ground-truth prosody kể cả khi model size nhỏ, trong khi mel gen ra được có chất lượng khá thấp (mờ, hay kiểu over-smoothing).

Tiếp theo, nếu mọi người đã tìm hiểu qua về model GlowTTS, thì cũng đã biết nó không còn phụ thuộc vào một kẻ ngoại bang nào cả, khi nói đến aligner LOL. Nó sử dụng cách tiếp cận flow-based, cụ thể là directly searches for the most probable monotonic alignment between text and the latent representation of speech using normalizing flows and dynamic programming. Chính xác là MAS là thứ gánh cho model này! Tuy nhiên, hãy để ý là những model flow-based NAR-TTS này có thể fast, diverse và controllable speech synthesis, không sợ mel mờ như bên trên, nhưng cũng kéo theo một nhược điểm, nó đòi hỏi sử dụng huge model capacity, và thực sự model size rất nhạy cảm với performance, theo hướng tiêu cực ạ!

Như vậy tại sao lại cần đề cập hai mục trên? Ở đây PortaSpeech sẽ tận dụng sức mạnh của hai ông này, VAE and normalizing flow thực sự đề xuất một model nhẹ mà vẫn generate mel-spectrograms with rich details and expressive prosody. Ngoài ra, không thể không nhắc đến việc đề xuất sử dụng mixture alignment in the linguistic encoder, which improves the prosody and reduces the dependence on fine-grained (phoneme-level) hard alignment, và introducing the grouped parameter sharing mechanism to the flow-based post-net.



# Main points

Như có thể quan sát ở hình bên trên, PortaSpeech is composed of a linguistic encoder with mixture alignment, a variational generator with enhanced prior and a flow-based post-net with the grouped parameter sharing mechanism. First, the text sequence with word-level boundary is fed into the linguistic encoder to extract the linguistic features in both phoneme and word level. Secondly, to model the expressiveness and variability of speech with lightweight architecture, we train the VAE-based variational generator to maximize the ELBO over the ground-truth mel-spectrograms conditioned on the linguistic features, whose prior distribution is modeled by a small volume-preserving normalizing flow. Finally, to refine and enhance the natural speech details in the generated mel-spectrograms, we train the post-net by maximizing the likelihood of ground-truth mel-spectrograms conditioned on both the linguistic features and the outputs of the variational generator. During inference, the text is transformed to mel-spectrograms by successively passing through the linguistic encoder, the decoder of the variational generator and the reversed flow-based post-net.

## Linguistic Encoder with Mixture Alignment

Trước đây, một số Non-autoregressive sẽ sử dụng một duration predictor để predict ra số frames ứng với một phoneme. Ở đây, nhãn - ground-truth phoneme duration (hard alignment) của các duration predictor sẽ có thể là lấy từ một autoregressive model (như ở FastSpeech) hoặc một tool hỗ trợ nào đó (ví dụ như MFA khi dùng FastSpeech2), hay thậm chí là jointly monotonic alignment training (giống như trong GlowTTS). Việc dùng phoneme-level hard alignment như vậy ẩn chứa sự nguy hiểm tiềm tàng. Since some of the boundaries between two phonemes are naturally uncertain, kiểu như It could be difficult to determine the exact boundary between two phonemes in millisecond level even for manually labeling. it is challenging for the alignment model to obtain very accurate phoneme-level boundaries, which inevitably introduces errors and noises. Further, these alignment errors and noises can affect the training of duration predictor, which hurts the prosody of the generated speech in inference.

⇒ Đề xuất sử dụng mixture alignment to the linguistic encoder. Linguistic Encoder consists of a phoneme encoder, a word encoder, a duration predictor and a word-to-phoneme attention module. Ở đây, block này sử dụng soft alignment in phoneme level and keeps hard alignment in word level. 

Một diều cần lưu ý là in training, the ground-truth word-level duration can be obtained by external forced alignment tools or autoregressive TTS models. Trong khi nói đến predict the word-level duration, mình sẽ use the duration predictor which lấy đầu ra của phoneme encoder làm input and then sums the predicted duration of the phonemes in each word as the word-level duration.

Một điều đặc biệt nữa là bài báo nhấn mạnh việc sử dụng cả word-to-phoneme attention module để add fine-grained linguistic information. Ngoài ra, bài báo đề xuất sử dụng thêm a word-level relative positional encoding embedding cho cả hai bên (như trên ảnh) để encourage the attention to be close to the diagonal.

Tóm lại lợi ích của block này thật tuyệt : ))

- Mixture alignment can improve the prosody, which may benefit from more accurate duration extraction and prediction; 
- Mixture alignment can also improve the generated voice quality since the soft alignment helps the end-to-end model optimization.


## Variational Generator with Enhanced Prior

Tiếp theo, bài báo dùng variational Generator dựa trên VAE để tạo ra Mel. Như mọi người biết thì VAE cổ điển thì simple distribution (Gaussian) as the prior, which results in strong constraints on the posterior: optimizing with Gaussian prior pushes the posterior distribution towards the mean, limiting diversity and hurting the generative power

⇒ To enhance the prior distribution, bài báo introduce a small volume-preserving normalizing flow - VP-flow, which transforms simple distributions (e.g., Gaussian distribution) to complex distributions through a series of K invertible mappings (a stack of WaveNet residual blocks with dilation 
1). Cái complex distributions sẽ là input prior of the VAE

In training, the posterior distribution N(µq, σq) is encoded by the encoder of the variational generator. Then zq is sampled from the posterior distribution using reparameterization and is passed to the decoder of the variational generator (the right dotted line). In the meanwhile, the posterior distribution is fed into the VP-Flow to convert it to a standard normal distribution (the middle dotted line). In inference, VP-Flow converts a sample in the standard normal distribution into a sample zp in the prior distribution of the variational generator and we pass the zp to the decoder of the variational generator.

## Flow-based Post-Net

Đến bước này, chúng ta sẽ mong muốn tạo ra high-quality mel-spectrograms bằng việc sử dụng flow-based postnet với strong condition inputs để refine lai output của Variational Generator. The architecture of the post-net adopts Glow and is conditioned on the outputs of the variational generator and the linguistic encoder. In training, the post-net transforms the mel spectrogram samples into latent prior distribution (isotropic multivariate Gaussian) and calculates the exact log-likelihood of the data using the change of variables. In inference, we sample the latent variables from the latent prior distribution and pass them into the post-net reversely to generate the high-quality mel-spectrogram.

Ngoài ra module này còn sử dụng một kỹ thuật khá hắc ám. Chính vì conditional inputs của postnet đã chứa thông tin về text và cả prosody từ các module trước, bây giờ post-net của chúng ta sẽ chỉ tập trung vào việc modeling the details in mel-spec, việc này cũng đã giúp giảm thiểu model capacity, tuy nhiên thế là chưa đủ. Bài báo này còn đề xuất grouped parameter sharing mechanism to the affine coupling layer, which shares some model parameters among different flow steps. 


## Experiments

Hình dưới đây cho thấy kết quả thí nghiệm của portaspeech. Có thể thấy về các độ đo audio performance (MOS-Q và MOS-P) thì bản normal của Portaspeech đã vượt qua TTS models khác như FastSPeech2, Tacotron 2  trong khi bản small cũng có kết quả competitive. Về RTF thì portaspeech cũng đã có kết quả tương đối tốt so với những model NAR. Điều đặc biệt nhất ở đây là portaspeech có khá ít Mem footprint và tham số so với các model TTS khác. 

In training, the ground-truth word-level duration can be obtained by external forced alignment tools or autoregressive TTS models. Loss terms: 
- Duration prediction loss Ldur: MSE between the predicted and the ground-truth word-level duration in log scale
- Reconstruction loss of variational generator: MAE between the ground-truth mel-spectogram and that generated by the variational generator
- The KL-divergence of variational generator


In inference:
- The linguistic encoder encodes text sequence, predicts the word-level duration, obtain the linguistic hidden states HL.
- Sample z from the enhanced prior, the decoder of the VG generates the mel-spectrograms conditioned on the linguistic hidden states HL.
- The post-net converts randomly sampled latent into fine-grained mel-spectrograms, conditioned on the linguistic hidden states HL and output from VG.

