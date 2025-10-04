---
layout: default
title: "RAG-HAR"
---

# Ablation Study

## Experimental Setup

We use publicly available and widely adopted HAR datasets for our experiments.

#### HHAR

This dataset primarily contains locomotion-style activities such as walking, sitting, standing, and going up or down the stairs. The data was collected from smartphones and smartwatches placed on the wrist, consisting of 6 channels. We use 2-second windows with 50% overlap.

#### PAMAP2

This dataset includes 12 protocol-defined activities recorded from 8 subjects. We retain 36 IMU channels, excluding heart rate, temperature, and orientation. Data is sampled at 100 Hz, with 2-second windows and a 0.5-second step. The dataset covers locomotion as well as daily activities such as ironing and vacuum cleaning.

#### MHEALTH

This dataset contains 12 activities, including locomotion and exercises such as jogging and cycling. It consists of 21 channels sampled at 50 Hz. We use sliding windows of 4 seconds with a 1-second step.

#### GOTOV

This dataset includes 16 activities collected from older adults (aged 61+) across three body positions, with 9 sensor channels in total. It contains pace variations (e.g., walking slow, normal, fast) and equipment variations (e.g., sitting on chair, sofa, or couch). Windows are created using 24-sample segments with 50% overlap.

#### SKODA

This dataset records 10 assembly-line activities performed by car manufacturing workers using 60 sensors on the right-hand side of the body. We use 24-sample windows with 50% overlap.

#### USC-HAD

This dataset contains 12 activities collected from 14 subjects using accelerometer and gyroscope signals (6 channels). Data is downsampled to 33.3 Hz and segmented into 1-second windows with 50% overlap. Subjects 13 and 14 are reserved for testing.

<br>
## Temporal and Hyperparameter Analysis of RAG-HAR
<br>
### Temporal Partitioning Analysis

We investigated the effect of different temporal partitioning strategies used during hybrid search. The motivation for this analysis is that activity patterns may not be uniformly distributed across the temporal dimension of a signal window. Transitions often occur at the beginning or end of a segment, while steady-state motion is captured in the middle portion. By explicitly modeling different temporal partitions, we aim to assess whether combining localized representations yields better retrieval performance compared to a single global embedding. This analysis was conducted on the MHEALTH dataset.

We considered the following partitioning configurations:

- **Whole window only:** A single embedding computed over the entire temporal window.
- **Start + End partitions:** Separate embeddings computed from the initial and final segments of the window.
- **Start + Mid + End partitions:** Embeddings derived from three equal subdivisions, excluding the whole-window representation.
- **Whole + Start + Mid + End partitions:** A hybrid configuration that combines the global representation with localized embeddings from all three partitions.

The results are summarized in **Table 1**. The whole-window representation alone provides a strong baseline. Incorporating start and end partitions captures transitional dynamics, yielding modest gains. Adding mid-segment information further improves performance, particularly for activities with sustained motion. The best results are consistently obtained when both the global embedding and all local partitions are combined, demonstrating that global context and localized temporal cues are complementary.

##### Table 1. Impact of different temporal partitioning strategies on retrieval performance

| Partitioning Strategy     | Accuracy | F1-score |
| ------------------------- | -------- | -------- |
| Whole window only         | 92.00    | 92.13    |
| Start + End               | 91.00    | 91.86    |
| Start + Mid + End         | 90.00    | 90.79    |
| Whole + Start + Mid + End | 95.00    | 94.90    |

---

<br>

### Hyperparameter Sensitivity Analysis

To better understand the impact of different configurations, we conducted a systematic hyperparameter sensitivity analysis on the MHEALTH dataset. The goal was to isolate the contribution of each parameter while controlling for confounding effects. Following are the models and hyper-parameters used as our **default configuration** throughout the experiments:

- Classifier model: `gpt-5-mini`
- Embedding model: `text-embedding-3-small`
- Weighted Reranking: (0.40, 0.20, 0.20, 0.20)
- Retrieved context: 10

When exploring the effect of one parameter, all others were fixed at these default values. This controlled setup allows us to attribute performance differences directly to the varied hyperparameter. The analysis was conducted on a subset of 100 samples, selected to maintain a balanced distribution of activity classes while ensuring the feasibility of repeated sensitivity experiments within reasonable computational limits.

---

#### Hybrid Search Weights

We studied the effect of different weighting strategies in the hybrid search mechanism, which combines whole, start, mid, and end embeddings. The results are presented in **Table 2**. We find that the scheme `[0.4, 0.2, 0.2, 0.2]` achieves the highest accuracy. By assigning greater weight to the whole-window embedding, this configuration allows the hybrid search to prioritize global activity patterns while still incorporating complementary fine-grained cues from localized partitions.

##### Table 2. Impact of hybrid search weighting on retrieval accuracy and F1-score

| Weights [whole, start, mid, end] | Accuracy | F1-score |
| -------------------------------- | -------- | -------- |
| [0.25, 0.25, 0.25, 0.25]         | 93.00    | 93.90    |
| [0.4, 0.2, 0.2, 0.2]             | 95.00    | 94.90    |
| [0.1, 0.4, 0.1, 0.4]             | 92.50    | 92.40    |
| [0.2, 0.2, 0.4, 0.2]             | 85.00    | 85.30    |

---

#### Embedding Models

Next, we compared different embedding models for vectorization. The results are presented in **Table 3**. `text-embedding-3-small` achieved strong and comparable performance, while `text-embedding-3-large` underperformed. This suggests that compact embedding models can be more effective than higher-capacity variants for retrieval-augmented classification, as they often produce cleaner, more generalizable similarity representations, whereas larger models may introduce noise or overfit to less relevant details.

##### Table 3. Impact of different embedding models on retrieval accuracy and F1-score

| Text Embedding Model   | Accuracy | F1-Score |
| ---------------------- | -------- | -------- |
| text-embedding-3-small | 95.0     | 94.9     |
| text-embedding-3-large | 89.0     | 89.6     |
| Ada-002                | 85.0     | 82.2     |

---

#### Retrieved-Context Exemplars

We then varied the number of retrieved-context exemplars provided in the prompt. The results are presented in **Table 4**. We found that performance increases from 5 to 10 exemplars, with 10 examples achieving the best results. However, increasing the number beyond 10 led to a slight decrease in accuracy, suggesting that adding more context does not necessarily improve performance and may even dilute the most relevant signals. This indicates that relatively small exemplar settings are sufficient for strong performance.

##### Table 4. Impact of number of retrieved-context exemplars on classification performance

| Number of Exemplars | Accuracy | F1-Score |
| ------------------- | -------- | -------- |
| 5                   | 90.8     | 91.0     |
| 10                  | 95.0     | 95.0     |
| 15                  | 94.0     | 92.4     |
| 20                  | 95.0     | 94.45    |
| 25                  | 92.0     | 90.33    |

---

#### Classifier Model

Finally, we evaluated the impact of different large language models while keeping embeddings and search configurations fixed. The results are presented in **Table 5**. Among all models, `gpt-5-mini` achieved the best performance (95.0% accuracy, 95.1% F1), confirming the advantage of reasoning-oriented models in structured classification tasks. The open-source `gpt-oss` also delivered competitive results (92.7% accuracy, 92.3% F1), outperforming larger instruction-tuned models such as `gemma-27b-it` and `llama3.3`. In contrast, the general-purpose `gpt-4o` lagged behind (88.0% accuracy, 87.6% F1), suggesting that general reasoning strength does not directly translate to higher accuracy in retrieval-augmented classification.

##### Table 5. Impact of different LLM models on classification performance

| LLM Model         | Accuracy | F1-Score |
| ----------------- | -------- | -------- |
| gpt-5-mini        | 95.0     | 95.1     |
| gpt-4o            | 88.0     | 87.6     |
| llama3.3          | 89.0     | 88.3     |
| gemma-27b-it 90.1 | 90.0     | 89.6     |
| gpt-oss           | 92.7     | 92.3     |

---
