# Scalable VLM Probing

[[Paper]](https://aclanthology.org/2023.starsem-1.26.pdf)
[[ACL Anthology page]](https://aclanthology.org/2023.starsem-1.26/)
[[Poster]](https://lit.eecs.umich.edu/posters/probing_clip_poster_2023.pdf)

This work proposes a simple and effective method to **probe vision-language models** (VLMs). 

Our method is **scalable**, as it does not require data annotations since it leverages existing datasets. 
With our method, we analyzed the performance of [CLIP](https://openai.com/research/clip), a popular state-of-the-art
multi-modal model, on the [SVO-Probes](https://github.com/deepmind/svo_probes) benchmark. 

![A description of our probing method, showing 2 images being input to CLIP, then 3 scores being computed. Different
kind of features are used to compute their correlation with each of the scores.](task_overview.png)

We hope our work contributes to ongoing efforts to discover the limitations of multi-modal models and help build more
robust and reliable systems. Our framework can be easily used to analyze other benchmarks, features, and multi-modal
models.

For more information, read our [*SEM 2023](https://sites.google.com/view/starsem2023) paper:

[Scalable Performance Analysis for Vision-Language Models](https://aclanthology.org/2023.starsem-1.26.pdf)

By [Santiago Castro](https://santi.uy/)+, [Oana Ignat](https://oanaignat.github.io/)+, and
[Rada Mihalcea](https://web.eecs.umich.edu/~mihalcea/).

(+ equal contribution.)

This repository includes the obtained results along with the code to reproduce them.

## Obtained Results

Under [results/](results) you can find the detailed results obtained with our method for the 3 different scores tested.
They were generated by running the code in this repository. See below to reproduce it and read the paper (see the link
above) to find an explanation of the results.

## Reproducing the Results

1. With Python >= 3.8, run the following commands:

    ```bash
    pip install -r requirements.txt
    python -c "import nltk; nltk.download(['omw-1.4', 'wordnet'])"
    spacy download en_core_web_trf
    mkdir data
    ````

2. Compute the CLIP scores for each image-sentence pair and save it to a CSV file. For this step, we used [a Google
    Colab](https://colab.research.google.com/drive/1I10mjHD-_brEtaKdHqhvkHjRFjVzK1hl?usp=sharing). You can see the
    results in [this Google Sheet](https://docs.google.com/spreadsheets/d/1TPYLRk_f6zMm7pYy8xLPeeS6EsybSCoxjnONOrg40vA/edit?usp=sharing).
    [This file is available to
    download](https://huggingface.co/datasets/MichiganNLP/scalable_vlm_probing/blob/main/svo_probes_with_scores.csv).
    Place it at `data/svo_probes_with_scores.csv`.
3. Compute a CSV file that contains the negative sentences for each of the negative triplets. We lost the script for
    this step, but it's about taking the previous CSV file as input and taking the sentence for the same triplet in the
    `pos_triplet` column (you can use [the original SVO-Probes file](https://github.com/deepmind/svo_probes/blob/main/svo_probes.csv)
    if there are missing sentences). This file should have the columns `sentence` and `neg_sentence`, in the same order
    as the column `sentence` from the previous CSV file. We provide [this file already
    processed](https://huggingface.co/datasets/MichiganNLP/scalable_vlm_probing/blob/main/neg_d.csv). Place it at
    `data/neg_d.csv`.
4. Merge the information from these 2 files:

    ```bash
    ./merge_csvs_and_filter.py > data/merged.csv
    ```

    We provide [the output of this command ready to
    download](https://huggingface.co/datasets/MichiganNLP/scalable_vlm_probing/blob/main/merged.csv).

5. Compute word frequencies in a 10M-size subset from [LAION-400M](https://laion.ai/blog/laion-400-open-dataset/):

    ```bash
    ./compute_word_frequencies.py > data/words_counter_LAION.json
    ```

    We provide [the output of this command ready to
    download](https://huggingface.co/datasets/MichiganNLP/scalable_vlm_probing/blob/main/words_counter_LAION.json).

6. Obtain LIWC 2015. See [LIWC website](https://www.liwc.app/) for more information. Set the path or URL of the file
    `LIWC.2015.all.txt` in the environment variable `LIWC_URL_OR_PATH`:
    
    ```bash
    export LIWC_URL_OR_PATH=...
    ```

    You can also disable the LIWC features in the next command by using the flag `--remove-features` along with other
    features, such as the default removed ones: `--remove-features LIWC wup-similarity lch-similarity path-similarity`.

7. Run the following to obtain the resulting correlation scores and save them as files:

    ```bash
    ./main.py --dependent-variable-name pos_clip_score --no-neg-features > results/pos_scores.txt
    ./main.py --dependent-variable-name neg_clip_score > results/neg_scores.txt
    ./main.py --dependent-variable-name clip_score_diff > results/score_diff.txt
    ```

    We already provide these files under [results/](results). By default, this script takes our own `merged.csv` file as
    input, but you can provide your own by using `--input-path data/merged.csv`. The same happens for other files. Run
    `./main.py --help` to see all the available options. We also recommend you look at the code to see what it does.
    Note that this repository includes code for preliminary experiments that we didn't report in the paper (for clarity)
    and we include it here in case it's useful.

## Citation

```bibtex
@inproceedings{castro-etal-2023-scalable,
    title = "Scalable Performance Analysis for Vision-Language Models",
    author = "Castro, Santiago  and
      Ignat, Oana  and
      Mihalcea, Rada",
    booktitle = "Proceedings of the 12th Joint Conference on Lexical and Computational Semantics",
    month = jul,
    year = "2023",
    address = "Toronto, Canada",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2023.starsem-1.26",
    pages = "284--294",
    abstract = "Joint vision-language models have shown great performance over a diverse set of tasks. However, little is known about their limitations, as the high dimensional space learned by these models makes it difficult to identify semantic errors. Recent work has addressed this problem by designing highly controlled probing task benchmarks. Our paper introduces a more scalable solution that relies on already annotated benchmarks. Our method consists of extracting a large set of diverse features from a vision-language benchmark and measuring their correlation with the output of the target model. We confirm previous findings that CLIP behaves like a bag of words model and performs better with nouns and verbs; we also uncover novel insights such as CLIP getting confused by concrete words. Our framework is available at https://github.com/MichiganNLP/Scalable-VLM-Probing a and can be used with other multimodal models and benchmarks.",
}
```
