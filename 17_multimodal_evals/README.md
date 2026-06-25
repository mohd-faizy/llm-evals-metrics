# 17. Multimodal Evals

> **Purpose** — Evaluate systems that work with more than text: images, audio, video, and documents. Multimodal LLMs introduce unique evaluation challenges — visual grounding, cross-modal reasoning, OCR accuracy, and modality-specific failure modes. This section covers metrics, benchmarks, and strategies for each modality.

---

## Table of Contents

- [What Are Multimodal Systems?](#what-are-multimodal-systems)
- [Why Multimodal Evals Are Different](#why-multimodal-evals-are-different)
- [Modalities & Capabilities](#modalities--capabilities)
- [Vision Evaluation](#vision-evaluation)
- [Audio Evaluation](#audio-evaluation)
- [Video Evaluation](#video-evaluation)
- [Document Understanding](#document-understanding)
- [Cross-Modal Evaluation](#cross-modal-evaluation)
- [Multimodal Benchmarks](#multimodal-benchmarks)
- [Building Multimodal Eval Datasets](#building-multimodal-eval-datasets)
- [Common Failure Modes](#common-failure-modes)
- [Common Mistakes](#common-mistakes)
- [Key Takeaways](#key-takeaways)

---

## What Are Multimodal Systems?

Multimodal systems process and/or generate content across multiple modalities:

```
               ┌──────────────────────────────────────┐
               │         MULTIMODAL LLM                │
               │                                      │
  Image   ────>│                                      │──> Text
  Text    ────>│   Understand & Reason                │──> Image
  Audio   ────>│   Across Modalities                  │──> Audio
  Video   ────>│                                      │──> Code
  Document ───>│                                      │──> Structured Data
               └──────────────────────────────────────┘
```

---

## Why Multimodal Evals Are Different

| Challenge | Description |
|---|---|
| **Multiple input types** | Same model handles text, images, audio — each needs different eval |
| **Cross-modal reasoning** | Model must connect information across modalities |
| **Grounding** | Model must correctly reference specific parts of non-text input |
| **Hallucination is visual** | Model may "see" things that aren't in the image |
| **Subjectivity** | Image descriptions can be subjective — many valid answers |
| **Richer failure modes** | OCR errors, spatial reasoning failures, temporal misalignment |
| **Evaluation difficulty** | Harder to create ground truth for non-text content |

---

## Modalities & Capabilities

| Modality | Input → Output | Key Capabilities | Example Tasks |
|---|---|---|---|
| **Vision** | Image → Text | Description, VQA, classification, grounding | "What's in this image?" "Count the red cars" |
| **Vision** | Text → Image | Image generation | "Generate a sunset over mountains" |
| **Audio** | Audio → Text | Transcription, understanding | Speech-to-text, audio QA |
| **Audio** | Text → Audio | Speech synthesis | Text-to-speech, voice cloning |
| **Video** | Video → Text | Temporal understanding, summarization | "Summarize this video" "What happens at 2:30?" |
| **Document** | Document → Text | Layout understanding, extraction, QA | Invoice extraction, form filling |
| **Cross-modal** | Mixed → Text | Multi-source reasoning | "Does the caption match the image?" |

---

## Vision Evaluation

### Image Understanding Tasks

| Task | Description | Metrics | Example Benchmark |
|---|---|---|---|
| **Visual QA (VQA)** | Answer questions about images | Accuracy, VQA score | VQAv2, GQA |
| **Image captioning** | Generate descriptions of images | CIDEr, METEOR, human rating | COCO Captions |
| **Object detection** | Identify and locate objects | mAP, IoU | COCO, LVIS |
| **Spatial reasoning** | Understand object relationships | Accuracy | VSR, BLINK |
| **OCR / Text recognition** | Read text in images | Character accuracy, CER | TextVQA, DocVQA |
| **Visual grounding** | Connect text references to image regions | Accuracy@IoU | RefCOCO |
| **Image classification** | Categorize images | Accuracy, F1 | ImageNet |

### Vision-Specific Metrics

| Metric | What It Measures | Best For |
|---|---|---|
| **VQA Accuracy** | Exact answer match (with soft matching) | Visual question answering |
| **CIDEr** | Consensus-based image description quality | Captioning (correlates well with humans) |
| **CLIPScore** | Image-text alignment using CLIP embeddings | Generation, captioning |
| **Spatial Accuracy** | Correctness of spatial relationships | "Left of", "above", "between" reasoning |
| **OCR CER** | Character error rate in text recognition | Document understanding |
| **Hallucination Rate** | Objects/details described that aren't in the image | All vision tasks |

### Vision Failure Modes

| Failure | Description | Example |
|---|---|---|
| **Object hallucination** | Describes objects not present | "There's a dog in the image" (no dog) |
| **Counting errors** | Incorrect number of objects | "There are 3 people" (actually 5) |
| **Spatial errors** | Wrong spatial relationships | "The cat is on the table" (cat is under) |
| **OCR errors** | Misreads text in images | "STOP" read as "STOB" |
| **Cultural misinterpretation** | Misidentifies culturally specific content | Wrong identification of cultural symbols |
| **Fine-grained confusion** | Confuses similar objects/species/brands | Dog breed misidentification |

---

## Audio Evaluation

### Audio Tasks & Metrics

| Task | Metrics | Tools/Benchmarks |
|---|---|---|
| **Speech-to-text (ASR)** | WER (Word Error Rate), CER | LibriSpeech, Common Voice |
| **Audio QA** | Accuracy | Audio Question Answering benchmarks |
| **Speaker identification** | Accuracy, EER | VoxCeleb |
| **Emotion recognition** | F1, accuracy | IEMOCAP |
| **Sound classification** | Accuracy, mAP | AudioSet |
| **Music understanding** | Accuracy, human rating | MusicCaps |

### ASR-Specific Metrics

| Metric | Definition | Target |
|---|---|---|
| **WER (Word Error Rate)** | (Substitutions + Insertions + Deletions) / Total Words | < 5% for clear speech |
| **CER (Character Error Rate)** | Character-level error rate | < 3% |
| **RTF (Real-Time Factor)** | Processing time / audio duration | < 1.0 for real-time |
| **Speaker diarization accuracy** | Correct speaker assignment | > 90% DER |

---

## Video Evaluation

### Video Understanding Tasks

| Task | Description | Metrics |
|---|---|---|
| **Video QA** | Answer questions about video content | Accuracy |
| **Video summarization** | Generate text summary of video | ROUGE, human rating |
| **Temporal grounding** | Locate when something happens in a video | IoU, R@1 |
| **Action recognition** | Identify actions/activities | Accuracy, mAP |
| **Video captioning** | Describe video content frame by frame | CIDEr, METEOR |

### Video-Specific Challenges

| Challenge | Description | Evaluation Approach |
|---|---|---|
| **Temporal reasoning** | Understanding sequence of events | Multi-step temporal QA |
| **Long video understanding** | Processing hours of content | Sparse sampling + key event detection |
| **Cross-frame consistency** | Maintaining consistent understanding across frames | Consistency checks across timestamps |
| **Real-time processing** | Processing video as it streams | Latency + accuracy under real-time constraint |

---

## Document Understanding

### Document Tasks & Metrics

| Task | Description | Metrics | Benchmark |
|---|---|---|---|
| **Document QA** | Answer questions about document content | Accuracy, ANLS | DocVQA |
| **Key-value extraction** | Extract structured data from documents | F1, precision, recall | FUNSD, CORD |
| **Table extraction** | Parse tables from documents | TEDS (Tree-Edit Distance Similarity) | PubTabNet |
| **Layout understanding** | Understand document structure | Accuracy | DocLayNet |
| **Form filling** | Complete forms based on source documents | Field accuracy | Custom |
| **Invoice processing** | Extract fields from invoices | Field-level accuracy | Custom |

### Document-Specific Metrics

| Metric | What It Measures | Range |
|---|---|---|
| **ANLS** | Average Normalized Levenshtein Similarity | 0–1 (higher = better) |
| **TEDS** | Tree-Edit Distance Similarity (for tables) | 0–1 |
| **Field Accuracy** | % of fields extracted correctly | 0–1 |
| **Layout IoU** | Overlap between predicted and actual layout regions | 0–1 |
| **OCR Accuracy** | Character-level text extraction correctness | 0–1 |

---

## Cross-Modal Evaluation

Cross-modal evaluation tests the model's ability to reason across modalities simultaneously.

### Cross-Modal Tasks

| Task | Input | Output | What It Tests |
|---|---|---|---|
| **Image + text → reasoning** | Image + question about the image | Correct answer | Visual reasoning with text context |
| **Text + image → verification** | Claim + image | True/False | Cross-modal fact checking |
| **Document + question → answer** | Scanned document + question | Extracted answer | OCR + comprehension |
| **Image + image → comparison** | Two images | Comparison text | Multi-image reasoning |
| **Audio + text → understanding** | Audio clip + transcript | Correction/analysis | Multi-source verification |

### Cross-Modal Metrics

| Metric | What It Measures |
|---|---|
| **Cross-modal consistency** | Do text and visual descriptions agree? |
| **Grounding accuracy** | Does text correctly reference visual elements? |
| **Multi-source accuracy** | When information comes from multiple modalities, is it correctly integrated? |
| **Modality bias** | Does the model over-rely on one modality? |

---

## Multimodal Benchmarks

| Benchmark | Modality | Focus | Scale | Key Feature |
|---|---|---|---|---|
| **[MMMU](https://mmmu-benchmark.github.io/)** | Vision + Text | College-level multimodal QA | 11.5K questions | 30 subjects, expert-level |
| **[MathVista](https://mathvista.github.io/)** | Vision + Text | Mathematical reasoning with visuals | 6K problems | Charts, geometry, scientific figures |
| **[VQAv2](https://visualqa.org/)** | Vision + Text | Visual question answering | 1.1M questions | Large-scale, balanced |
| **[TextVQA](https://textvqa.org/)** | Vision + Text | Reading text in images | 45K questions | OCR-dependent |
| **[DocVQA](https://www.docvqa.org/)** | Document | Document question answering | 50K questions | Industry documents |
| **[BLINK](https://zeyofu.github.io/blink/)** | Vision | Core visual perception | 3.8K questions | Tests skills multimodal LLMs struggle with |
| **[MMBench](https://mmbench.opencompass.org.cn/)** | Vision + Text | Systematic capability evaluation | 3K questions | 20 ability dimensions |
| **[Video-MME](https://video-mme.github.io/)** | Video + Text | Video understanding | 900 videos | Short to long videos |
| **[LibriSpeech](https://www.openslr.org/12)** | Audio | Speech recognition | 1000h audio | De facto ASR benchmark |

---

## Building Multimodal Eval Datasets

### Dataset Requirements by Modality

| Modality | Data Format | Annotation Requirements | Storage Considerations |
|---|---|---|---|
| **Image** | PNG/JPEG | Bounding boxes, captions, QA pairs | ~1–10 MB per image |
| **Audio** | WAV/MP3 | Transcripts, timestamps, speaker IDs | ~1 MB per minute |
| **Video** | MP4 | Temporal annotations, event descriptions | ~10–100 MB per minute |
| **Document** | PDF/images | Layout annotations, field labels | ~1–5 MB per document |

### Dataset Construction Tips

| Tip | Why |
|---|---|
| **Include diverse visual content** | Models fail on uncommon visual scenarios |
| **Test with various image qualities** | Blurry, low-light, cropped images test robustness |
| **Include multi-language documents** | OCR accuracy varies by language |
| **Test with real-world artifacts** | Screenshots, photos of screens, handwritten text |
| **Include culturally diverse content** | Avoid Western-centric visual bias |

---

## Common Failure Modes

| Failure Mode | Modality | Description | Detection |
|---|---|---|---|
| **Object hallucination** | Vision | Describes objects not in the image | Compare with ground truth objects |
| **Counting errors** | Vision | Wrong count of objects | Numerical comparison |
| **Spatial confusion** | Vision | Wrong spatial relationships | Spatial reasoning QA |
| **OCR errors** | Document | Misreads text | Character-level comparison |
| **Temporal confusion** | Video | Events described in wrong order | Temporal ordering tests |
| **Modality ignoring** | Cross-modal | Ignores one modality, answers from the other only | Probe with modality-specific questions |
| **Resolution sensitivity** | Vision | Fails on low-resolution or small objects | Test across resolutions |
| **Language bias in OCR** | Document | Better at English OCR than other languages | Multi-language testing |

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| **Text-only evaluation of multimodal systems** | Misses visual/audio failure modes | Include modality-specific tests |
| **Only testing with clean, high-quality inputs** | Fails on real-world messy inputs | Test with noisy, blurry, partial inputs |
| **Ignoring cross-modal consistency** | Model may describe what's not there | Check visual grounding explicitly |
| **No hallucination testing** | Object/fact hallucination goes undetected | Include hallucination-specific probes |
| **Single benchmark reliance** | Benchmarks cover limited scenarios | Combine benchmarks with custom eval sets |
| **Not testing edge cases per modality** | Misses modality-specific failures | Build edge case sets for each modality |

---

## Key Takeaways

| Principle | Details |
|---|---|
| **Each modality has unique failure modes** | Vision hallucination ≠ text hallucination — test each specifically |
| **Cross-modal reasoning is the hardest part** | Connecting information across modalities requires dedicated testing |
| **Visual grounding is critical** | Models must correctly reference what they "see", not hallucinate |
| **Test with real-world quality** | Clean benchmark images don't represent production inputs |
| **Use modality-specific metrics** | CIDEr for captioning, WER for ASR, ANLS for documents |
| **Benchmark coverage is incomplete** | No single benchmark tests everything — build custom eval sets |

---

Move to [18. Coding Agent Evals →](../18_coding_agent_evals/README.md) to learn how to evaluate code generation, code editing, and repository-level coding agents.
