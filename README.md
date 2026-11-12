Auditable LLM-Supported Workflow for Qualitative Thematic Analysis

This repository contains the Jupyter notebooks developed for the project Designing an Auditable LLM-Supported Workflow for Qualitative Thematic Analysis by post doc Nadia Jul Jeldtoft, University of Southern Denmark (2026).

The project presents a proof-of-concept computational operationalisation of inductive, interpretative thematic analysis (TA) for qualitative interview data. It combines locally deployed large language model (LLM) inference with deterministic Python-based data processing, validation, and traceability.

The workflow was developed and evaluated on Danish-language semi-structured interviews concerning experiences of changes in Danish society and family life from the 1960s to the 1980s.

Workflow overview

The workflow has two main phases:

Interview excerpts to codes through a Draft–Feedback–Refine procedure.

Codes to themes through iterative theme construction and code-to-theme assignment.

The LLM performs the interpretative tasks, while Python handles data structuring, identifiers, validation, error capture, and the links between interview excerpts, codes, justifications, and themes.

Phase 1: Interview excerpts to codes

Phase 1 implements a three-stage self-refinement procedure:

Draft: Generates one to three codes and an analytical justification for each interview excerpt. Excerpts containing only demographic or factual background information can be marked as not relevant.

Feedback: Evaluates each code in relation to the excerpt and research objective, then evaluates whether the associated justification is analytically adequate, vague, or paraphrastic.

Refine: Uses a deterministic decision tree to retain satisfactory outputs, revise the justification, revise both code and justification, or retain a not-relevant classification.

Each code is assigned a persistent code_uid, allowing the analytical output to be traced back to its interview and source excerpt.

Phase 2: Codes to themes

Phase 2 constructs an initial thematic map from a random batch of codes, justifications, and source excerpts. The remaining codes are then processed sequentially and either:

assigned to an existing theme with an explicit theme-assignment justification; or

placed in a no_fit reassessment pool.

When the reassessment pool exceeds the configured threshold, the workflow searches for additional candidate themes and reassesses the unassigned codes. Codes that repeatedly fail to fit a theme can be retained as Singular observation rather than being forced into an unsuitable theme.

Repository contents

Notebook

Purpose

Main input

Main output

Phase1_Draft_stage.ipynb

Generates initial codes and justifications

Segmented interview data

Draft coding results

Phase1_Feedback_stage.ipynb

Evaluates codes and then their justifications

Draft coding results

Code- and justification-level feedback

Phase1_Refine_stage.ipynb

Retains or revises outputs using the feedback

Feedback results

Final codes and justifications

Phase2_Codes_to_themes.ipynb

Develops themes and assigns codes to them

Final Phase 1 output

Theme-centric, code-centric, and no_fit results

The notebooks must be run in the order shown above.

Methodological design principles

The implementation is guided by five design principles:

Contextual interpretation: The input is segmented into analytical units containing two consecutive question–answer pairs.

Traceability: Persistent identifiers link the original interview excerpts to codes, justifications, themes, and theme-assignment justifications.

Explicit analytical reasoning: The workflow represents interpretative reasoning through code and theme justifications rather than returning labels alone.

Constrained inference: Pydantic schemas and Outlines are used to constrain, parse, and validate LLM outputs. Deterministic operations are handled in Python.

Privacy by design: The model is deployed locally in the compute environment, so sensitive interview material does not need to be sent to an external model API.

Requirements

The notebooks use:

Python 3.10 or later

JupyterLab or Jupyter Notebook

PyTorch

Hugging Face Transformers

Accelerate

Outlines

Pydantic 2

pandas

PyArrow

The implementation loads mistralai/Mistral-7B-Instruct-v0.3 in bfloat16 with automatic device mapping. It was designed for execution on a CUDA-enabled GPU environment such as an NVIDIA A100 or H100. Access to the model through Hugging Face may require authentication.

One possible environment setup is:

git clone https://github.com/njeldtoft/TA.git
cd TA

python -m venv .venv
source .venv/bin/activate
pip install jupyter torch transformers accelerate outlines "pydantic>=2" pandas pyarrow

On Windows, activate the environment with:

.venv\Scripts\activate

The repository does not currently include a pinned environment or lock file. Results may therefore depend on the installed package versions, model revision, hardware, and inference environment.

Data preparation

The source interview data are not included because they contain sensitive research material. To run the workflow, provide a pre-segmented Parquet file with at least the following columns:

Column

Description

int_id

Persistent interview identifier

unit_id

Persistent identifier for the analytical unit

interview_excerpt

Text passed to the model, formatted with Q: and A: markers

In the thesis implementation, each interview_excerpt contains two consecutive question–answer pairs. Interviewer questions provide local context, while coding is based primarily on the informant's answers.

The segmentation and preparation of the original interview transcripts are outside the scope of this repository.

Configuration

Before running the notebooks, define the directory variables used by the code. The current notebooks assume that these pathlib.Path variables already exist in the execution environment:

from pathlib import Path

OUTPUT_DIR = Path("/path/to/prepared_data")
DRAFT_DIR = Path("/path/to/results/phase1_draft")
FEEDBACK_DIR = Path("/path/to/results/phase1_feedback")
FINAL_DIR = Path("/path/to/results/phase1_final")

# Used in Phase 2
data_path = FINAL_DIR
pipeline1_path = Path("/path/to/results/phase2")

for directory in [DRAFT_DIR, FEEDBACK_DIR, FINAL_DIR, pipeline1_path]:
    directory.mkdir(parents=True, exist_ok=True)

The Draft notebook currently reads:

OUTPUT_DIR/test/test_interviews.parquet

Adjust this path or filename to match your prepared dataset.

Running the workflow

Launch Jupyter:

jupyter lab

Then run all cells in the notebooks in this order:

Phase1_Draft_stage.ipynb

Phase1_Feedback_stage.ipynb

Phase1_Refine_stage.ipynb

Phase2_Codes_to_themes.ipynb

Each Phase 1 notebook writes a timestamped Parquet file and a current... file used as input by the next stage. Phase 2 produces both Parquet and semicolon-separated CSV files.

Main outputs

Phase 1 ends with:

currentTEST.parquet: the final code-centric dataset, including code_uid, final codes, final justifications, feedback labels, refinement actions, and source excerpts.

Phase 2 writes:

current_themes.parquet and current_themes.csv: theme labels, theme descriptions, and associated code identifiers.

current_code_results.parquet and current_code_results.csv: code-to-theme assignments and theme-assignment justifications.

current_no_fit.parquet and current_no_fit.csv: codes that were not assigned to an existing theme.

Timestamped copies are also retained to support inspection of individual runs.

Reproducibility and interpretation

This repository contains a research prototype rather than a general-purpose software package. The prompts are domain-specific and reflect the research objective of the thesis. Applying the workflow to another dataset requires adaptation of the research objective, prompt content, segmentation strategy, and analytical criteria.

The Phase 2 procedure includes random code sampling and sequential theme assignment. Set a random seed if exact sampling reproducibility is required. More generally, local deployment and structured decoding improve governance and output control but do not remove the interpretative or model-dependent character of the analysis.

The repository contains the workflow notebooks only. The sensitive source data, transcript-preparation code, thesis evaluation data, and evaluation analyses are not included.

Citation

If you use or build on this workflow, please cite:

Jeldtoft, N. J. (2026). Designing an Auditable LLM-Supported Workflow for Qualitative Thematic Analysis [MSc thesis, University of Southern Denmark]. https://github.com/njeldtoft/TA

@mastersthesis{jeldtoft2026auditable,
  author = {Jeldtoft, Nadia Jul},
  title  = {Designing an Auditable LLM-Supported Workflow for Qualitative Thematic Analysis},
  school = {University of Southern Denmark},
  year   = {2026},
  url    = {https://github.com/njeldtoft/TA}
}

Author

Nadia Jul JeldtoftData Science, University of Southern Denmark

Licence

No licence is currently specified for this repository. Unless a licence is added, the code remains subject to standard copyright restrictions.
