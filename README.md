## Model Details
### Model Architecture
BERT (bert-base-cased): The BERT model is pre-trained for token classification tasks, making it well-suited for named entity recognition (NER). BERT processes token embeddings and their contextual relationships to classify each token into one of the pre-defined fields. Fine-tuning BERT for this entity extraction task involves token-level classification where each token is mapped to a specific entity.
Training Parameters
Learning Rate: 3e-5
Batch Size: 16
Epochs: 10
Optimizer: AdamW
Loss Function: Categorical Cross-Entropy
## Preprocessing
Token Grouping: Tokens are grouped based on proximity using coordinate-based clustering to ensure multi-token entities (e.g., employer addresses) are predicted as a single entity.
Coordinate Normalization: Token coordinates are normalized to ensure better input consistency across documents of varying sizes.
Text Cleaning: OCR errors are corrected (e.g., character misrecognitions like 0 vs O, S vs 5).

## EDA and Its Impact on Preprocessing and Model Selection
## EDA Insights
The Exploratory Data Analysis (EDA) provided key insights that directly impacted the preprocessing steps and the choice of the BERT model for token classification. Below are the main findings from the EDA and how they influenced the overall process:

### Token Distribution:

Other Entity have maximum number of tokens followed by employerAddressStreet_name and employerName .

Tokens for certain entities tend to appear in specific regions of the document (e.g., employer-related entities are generally in the top half, employee-related data in the lower half).

### Impact on Preprocessing:

Token Grouping: A preprocessing step was added to group tokens that should belong to a single entity (e.g., employerAddressStreet_name). This was necessary to avoid fragmented predictions for multi-token entities, ensuring proper entity boundaries.
Entity Contextualization: We utilized spatial and positional information to further group tokens based on their likely location in the document, which helped the model better learn entity relationships.
### OCR Quality:

OCR extraction was generally accurate for numeric entities (e.g., ssnOfEmployee, einEmployerIdentificationNumber), but certain alphanumeric fields (e.g., employerName, employerAddressStreet_name) had misrecognitions due to document noise and variation in formatting.

### Impact on Preprocessing:

Text Normalization: Tokens were preprocessed to correct common OCR errors, especially for frequently confused characters (e.g., 0 vs O, S vs 5). This step helped reduce errors during entity extraction by ensuring the input to the model was clean.
Noise Filtering: Spurious tokens and OCR artifacts (e.g., punctuation marks or stray symbols) were removed to prevent them from being treated as valid tokens.
Spatial Distribution:

Tokens in closely packed regions (e.g., address lines) were often misclassified as adjacent entities during the preliminary model runs.
Impact on Preprocessing:

Coordinate Normalization: The token coordinates were normalized to ensure consistency in documents of varying sizes. This allowed for more accurate grouping of spatially proximate tokens, especially for multi-line entities like addresses.
Influence on Model Choice
Given the insights from EDA, the BERT model was selected for the following reasons:

### Contextual Token Representation:

BERT’s transformer-based architecture excels at capturing context both within sentences and across tokens. Given the structured nature of the data, where entities are often adjacent or nearby, BERT was well-suited to model the contextual relationships between tokens.
Handling Multi-Token Entities:

The need to accurately classify multi-token entities (e.g., employerAddressStreet_name, box1WagesTipsAndOtherCompensations) influenced the decision to use a model like BERT, which can effectively manage token dependencies and provide accurate sequence labeling.
Pre-Trained Knowledge:

BERT's pre-trained knowledge on large corpora provided a strong starting point for fine-tuning on the token classification task. The rich feature representations from BERT helped the model generalize well to various entities, even when there were limited labeled examples for certain fields.
Error Analysis and Its Impact on Model Performance
Error Patterns
During the error analysis, the following key issues were identified:

### Misclassification of Closely Placed Tokens:

Some tokens for closely spaced entities, such as different components of an address, were misclassified due to their proximity. For example, employerAddressStreet_name was sometimes predicted as part of employerAddressCity.
Reasoning:

This issue was primarily caused by BERT misinterpreting spatially adjacent tokens as a part of the same entity, despite them representing different fields.
Improvement:

Introducing more precise token grouping during preprocessing helped alleviate this issue by ensuring tokens from distinct entities were correctly classified.
OCR Errors Leading to Mislabeling:

### Misrecognized characters
Misrecognized characters from OCR (e.g., 5 vs S or O vs 0) resulted in incorrect classifications for numeric entities like ssnOfEmployee and einEmployerIdentificationNumber.
Reasoning:

These errors occurred due to the inherent noise in the OCR process, which confused the model as it tried to classify tokens based on incorrect transcripts.
Improvement:

Post-processing steps that included validation rules (e.g., expected formats for SSN and EIN) helped catch these errors and allowed corrections based on pattern matching.
Truncated or Incomplete Tokens:
### Additional
In certain cases, tokens at the edges of the document or in areas with lower-quality OCR results were truncated, leading to incomplete predictions.
Reasoning:

The BERT model struggled with partial tokens that lacked sufficient context. This was especially true for entities like employerAddressStreet_name where only part of the token was present.
Improvement:

Token-level padding and careful handling of incomplete tokens were introduced to mitigate this issue. Additionally, rules for handling multi-line entities improved the prediction of truncated tokens.
## Boosting Model Performance
### Preprocessing Enhancements:

Cleaning the data, especially by correcting OCR mistakes and grouping multi-token entities, boosted the model's ability to generalize to different entities and reduced misclassifications.
Handling Fixed-Format Fields:

Using post-processing rules for fields like ssnOfEmployee, einEmployerIdentificationNumber, and box17StateIncomeTax improved the model’s precision by validating predictions against expected formats, catching and correcting errors where the model struggled.
### Fine-Tuning with Field-Specific Data:

Fine-tuning BERT with specific attention to high-variance fields (like addresses and wages) enhanced the model’s performance on complex, multi-token entities. The model was able to distinguish between fields with similar formats but different semantics.
## Hampering Factors
### OCR Noise:

OCR inconsistencies remained a significant issue, particularly for alphanumeric fields like employer names and addresses. While normalization and correction steps reduced the impact, this factor continued to hamper model performance in certain cases.
### Document Layout Variability:

The varying layouts of documents (i.e., different placements of the same entity) introduced some difficulties in token classification. Despite the BERT model’s strength in capturing context, highly irregular layouts still caused performance drops for certain fields.
## Conclusion
The combination of EDA-driven preprocessing steps and error analysis-based improvements helped optimize the performance of the BERT model for entity extraction. By understanding the token distribution, addressing OCR errors, and refining token-level features, the model’s precision, recall, and F1 scores were significantly boosted, particularly for structured and multi-token entities.
