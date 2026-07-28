# Verifying the Accuracy of an AI Text Classifier - with Confusion Matrix

This repository contains the evaluation workflow used to measure — and improve — the reliability of an LLM-based text classifier before trusting it to label an entire dataset. It is the companion repository to **[ai-text-classification](../ai-text-classification)**, which performs the actual classification of EUDR public consultation feedback into `include_leather`, `exclude_leather`, and `other_topic`.

The logic here is generic: it can be used to check the accuracy of any AI classification task, not just this one.

## Methodology and credit

This evaluation workflow follows the methodology taught by **Jeremy Merrill**, data/AI reporter at the *Washington Post*, as part of the Lede Program (2026 cohort): *"vibes are not enough"* — a handful of eyeballed examples cannot tell you whether a classifier is good enough to publish on. Instead, the workflow requires a **manually hand-coded ground-truth sample**, compared statistically against the model's guesses.

## Workflow

1. **Draw a random sample** of the dataset (e.g., 50 rows) before running any large-scale, paid classification.
2. **Export the sample to CSV** with a blank `groundtruth` column.
3. **Hand-code the sample**: a human reviewer fills in the correct category for each row, working from the original text.
4. **Merge and score**: the hand-coded ground truth is compared to the AI's guesses (`ai_guess`) using `scikit-learn`:
   - **Accuracy** (`accuracy_score`) — overall share of correct predictions.
   - **Baseline comparison** — accuracy of the "dumbest possible" classifier that always guesses the most frequent category. If the model doesn't beat this baseline, it isn't adding value.
   - **Confusion matrix** (`ConfusionMatrixDisplay`) — shows exactly which categories get confused with which, to guide prompt fixes.
   - **Precision and recall** — for a single category of interest (e.g., "is this about leather at all?"), computed as a binary classification: precision penalizes false positives, recall penalizes false negatives. Which one matters more depends on the editorial/analytical use case (e.g., shrinking a haystack of documents to review favors high recall).
5. **Inspect the errors**: rows where `ai_guess != groundtruth` are printed for review, to spot systematic mistakes (e.g., a language the prompt handles poorly, or a category boundary that's ambiguous).
6. **Iterate on the prompt**: adjust the system prompt (add examples, clarify edge cases, add "cajoling" instructions), re-run the classifier on the same hand-coded sample, and re-check the accuracy/precision/recall — keeping the same sample makes runs directly comparable.
7. **Only once the metric is judged "good enough"** for the task at hand, run the classifier on the full dataset.

## Sample output

For a 50-row hand-coded sample, the workflow reports a summary like:

```
Accuracy score: 84.0%
Baseline (always guessing "other_topic"): 46.0%
```

along with a confusion matrix plotted across the three categories, and — if a single category is singled out (e.g., `exclude_leather` vs. everything else) — a precision/recall pair such as:

```
Precision: 91.7%
Recall: 78.6%
```

(Values above are illustrative of the kind of report the workflow produces; your own accuracy will depend on your prompt, model, and data.)

## Tech stack

- Python 3, `pandas`
- `scikit-learn` (`accuracy_score`, `precision_score`, `recall_score`, `ConfusionMatrixDisplay`)
- A hand-coded `handcoded.csv` file (one row per sampled item, with a `groundtruth` column filled in manually)

## Setup

1. Run the classification pipeline from [ai-text-classification](../ai-text-classification) on a small random sample first.
2. Export that sample and manually fill in the `groundtruth` column (in Excel, Google Sheets, or a CSV editor).
3. Save the file as `handcoded.csv` in this repository's working directory.
4. Run the scoring script/notebook to get accuracy, baseline, confusion matrix, precision, and recall.
5. Adjust the classification prompt (in the other repo) and repeat until the metric is acceptable for your use case.

## License

MIT 
