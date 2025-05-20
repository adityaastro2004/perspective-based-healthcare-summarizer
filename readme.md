{\rtf1\ansi\ansicpg1252\cocoartf2822
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\pard\tx566\tx1133\tx1700\tx2267\tx2834\tx3401\tx3968\tx4535\tx5102\tx5669\tx6236\tx6803\pardirnatural\partightenfactor0

\f0\fs24 \cf0  # Explanation and How to Run\
\
 1.  **Setup:**\
     *   Make sure your Kaggle notebook has the dataset linked (e.g., `/kaggle/input/nlp-project-dataset/puma_dataset`). Adjust `BASE_PATH` if necessary.\
     *   Ensure GPU is enabled.\
     *   The first code cell installs all necessary libraries.\
\
 2.  **Configuration:**\
     *   The `if __name__ == "__main__":` block at the end controls the execution flow.\
     *   `TRAIN_CLASSIFIER`, `TRAIN_SUMMARIZER`, `RUN_INFERENCE`, `RUN_EVALUATION` flags allow you to run specific parts of the pipeline. Set them to `True` or `False`.\
     *   `CLASSIFIER_EPOCHS` and `SUMMARIZER_EPOCHS` are set to 1 for a quick demonstration. Increase these for proper training (e.g., 3-5 epochs).\
     *   `USE_ENERGY_LOSS_TRAINING`: Set to `True` to include the custom loss during summarizer training. Set to `False` to train with only the standard Cross-Entropy loss (useful for ablation or if energy models fail).\
\
 3.  **Classifier Training:**\
     *   If `TRAIN_CLASSIFIER` is `True`, it fine-tunes a `roberta-base` model on the task of classifying text spans into one of the five perspectives.\
     *   The best model based on validation loss is saved to `/kaggle/working/checkpoints/classifier/`. This model is later used for the `Ep` energy calculation.\
\
 4.  **Summarizer Training:**\
     *   If `TRAIN_SUMMARIZER` is `True`:\
         *   It loads the `facebook/bart-large` base model.\
         *   It applies LoRA configuration to the attention layers (`q_proj`, `v_proj`).\
         *   If `USE_ENERGY_LOSS_TRAINING` is `True`, it initializes the energy models (BERT for `Et`, the trained RoBERTa for `Ep`).\
         *   It trains the model using the `SummarizationCustomDataset` (which creates the perspective-specific prompts) and AdamW optimizer.\
         *   The loss is a combination of standard Cross-Entropy loss and the custom energy-based perspective loss (weighted by `lambda_perspective`). Gradient accumulation is used to handle potentially large memory requirements.\
         *   The best PEFT adapter weights (LoRA layers) are saved based on validation Cross-Entropy loss to `/kaggle/working/checkpoints/summarizer/`.\
         *   A plot of the training loss per epoch is saved.\
\
 5.  **Inference:**\
     *   If `RUN_INFERENCE` is `True`:\
         *   It loads the base `facebook/bart-large` model.\
         *   It loads the saved LoRA adapter weights onto the base model using `PeftModel.from_pretrained`.\
         *   It generates summaries for the test set using beam search.\
         *   The generated summaries, along with the target perspective, actual summary, and source text, are saved to `/kaggle/working/generated/bart_lora_generated_results.csv`.\
\
 6.  **Evaluation:**\
     *   If `RUN_EVALUATION` is `True`:\
         *   It loads the predictions and references from the inference CSV.\
         *   It calculates ROUGE, METEOR, BLEU, and BERTScore using the `EvaluationMetrics` class.\
         *   The scores are printed and saved to `/kaggle/working/generated/bart_lora_evaluation_scores.json`.\
\
 7.  **Memory Management:**\
     *   Includes `cleanup_memory()` calls (`gc.collect()` and `torch.cuda.empty_cache()`) at various stages to try and mitigate OOM errors, especially when loading multiple large models.\
     *   Uses gradient accumulation during summarizer training.\
     *   Includes basic OOM error handling within the training loop.\
\
\
 --- END OF FILE modified_plasma_bart_lora.ipynb ---}