# OTUS NLP Final Project: Detocification of Russian Texts (based on RUSSE-2022 Detoxification)
Repository for the final project for the OTUS NLP course

## Data
A parallel detoxification dataset was provided as a part of the [Detoxification shared task](https://russe.nlpub.org/2022/tox/) at Dialogue-2022. The source sentences are Russian toxic messages from Odnoklassniki, Pikabu and Twitter platforms. The target part of the dataset are the same messages which were manually rewritten by crowd workers to eliminate toxicity. Some toxic sentences contain multiple (up to 3) variants of detoxification.

The dataset is divided into train, development, and test sets. Dataset statistics:

    - train: 3,539 toxic sentences with 1-3 detoxified versions;
    - development: 800 toxic sentences with 1-3 detoxified versions;
    - test: 1,474 toxic sentences with 1-3 detoxified versions.

## Automatic Evaluation Metrics
- **Style transfer accuracy (STA)** — the average confidence of the [pre-trained BERT-based toxicity classifier](https://huggingface.co/SkolkovoInstitute/russian_toxicity_classifier) for the output sentences.
- **Meaning preservation (SIM)** — the distance of embeddings of the input and output sentences. The embeddings are generated with the [LaBSE model](https://huggingface.co/cointegrated/LaBSE-en-ru).
- **Fluency score (FL)** — the average confidence of the [BERT-based fluency classifier](https://huggingface.co/SkolkovoInstitute/rubert-base-corruption-detector) trained to discriminate between real and corrupted sentences.
- **Joint score (J)** — the sentence-level multiplication of the STA, SIM, and FL scores.
- **chrF** — the [chrF metric](https://www.nltk.org/_modules/nltk/translate/chrf_score.html) computed with respect to reference texts.

## Solutions
All solutions were based on fine-tuning either a [ruT5-base model](https://huggingface.co/ai-forever/ruT5-base) or a [ruGPT3-small model](https://huggingface.co/ai-forever/rugpt3small_based_on_gpt2).

### Baseline solutions
- **[ruT5-base](https://huggingface.co/ai-forever/ruT5-base)**: fine-tuned on training data with learning rate = 5e-5 and number of epochs = 30;
- **[rugpt3-small_based_on_gpt2](https://huggingface.co/ai-forever/rugpt3small_based_on_gpt2)**: fine-tuned on training data with learning rate = 5e-5 and number of epochs = 5. Training data was converted into the following format: `<toxic>TOXIC_COMMENT</toxic> >>>>> <neutral>NEUTRAL_COMMENT</neutral>` (later - prompt v1);
- **[rugpt3-small_based_on_gpt2](https://huggingface.co/ai-forever/rugpt3small_based_on_gpt2)**: fine-tuned on training data with learning rate = 5e-5 and number of epochs = 5. Training data was converted into the following format: `<toxic>TOXIC_COMMENT</toxic> >>>>> <neutral>NEUTRAL_COMMENT</neutral>` (later - prompt v2);

### Solutions with preprocessing
Each of the aforementioned models was then trained on data preprocessed in two ways. Learning rate and number of epochs remained unchanged.
- **preprocessing v1**: each toxic-neutral pair was automatically evaluated on STA, SIM and FL metrics. The training dataset was filtered so that only examples with style transfer accuracy, similarity and fluency over 0.5 remained;
- **preprocessing v2**: training data was cleaned from emojis, then filtered as described previously.

## Results on development-set
We provide the results of automatic evaluation of each model on development dataset. The results are sorted by the descending Joint score, because this was the target metric for the RUSSE Detoxification competition. Predictions made by the best model were submitted for the [competition leaderboad on CodaLab (Development phase)](https://codalab.lisn.upsaclay.fr/competitions/642#results) under the name *MariaNg*.
We also provide metrics for the competition baselines (described [here](https://www.dialog-21.ru/media/5755/dementievadplusetal105.pdf)) on the development dataset (as seen [here](https://codalab.lisn.upsaclay.fr/competitions/642#results)) for comparison. Competition baseline models are *italicised* in the table below.
|Model Name|Prompt|Preprocessing|STA|SIM|FL|J|ChrF1|
|----------|------|-------------|--:|--:|-:|-|----:|
|ruT5|n/a|v1|0.83|0.82|0.88|0.59|0.57|
|ruT5|n/a|v2|0.85|0.82|0.85|0.58|0.58|
|*ruT5 (competition baseline)*|n/a|none|0.75|0.81|0.82|0.50|0.58|
|ruT5|n/a|none|0.78|0.78|0.80|0.50|0.57|
|*ruPrompts (competition baseline)*|n/a|none|0.76|0.77|0.80|0.48|0.55|
|ruGPT3|v2|v2|0.76|0.70|0.74|0.40|0.47|
|ruGPT3|v1|v1|0.66|0.72|0.76|0.37|0.49|
|*Delete (competition baseline)*|n/a|none|0.53|0.87|0.82|0.36|0.52|
|ruGPT3|v1|v2|0.65|0.73|0.76|0.36|0.50|
|ruGPT3|v2|v1|0.74|0.68|0.73|0.36|0.46|
|ruGPT3|v2|none|0.71|0.64|0.75|0.35|0.45|
|ruGPT3|v1|none|0.78|0.50|0.65|0.27|0.33|

## Results on test-set
Predictions made by the best model were submitted for the [competition leaderboad on CodaLab (Post-evaluation phase)](https://codalab.lisn.upsaclay.fr/competitions/642#results) under the name *MariaNg*. In the table below we provide the results of the automatic evaluation of our predictions compared to the competition baselines.
|Model Name|Prompt|Preprocessing|STA|SIM|FL|J|ChrF1|
|----------|------|-------------|--:|--:|-:|-|----:|
|ruT5|n/a|v1|0.87|0.84|0.90|0.65|0.57|
|ruT5 (competition baseline)|n/a|none|0.80|0.83|0.84|0.56|0.57|
|ruPrompts (competition baseline)|n/a|none|0.81|0.79|0.80|0.53|0.55|
|Dalete (competition baseline)|n/a|none|0.56|0.89|0.85|0.41|0.53|