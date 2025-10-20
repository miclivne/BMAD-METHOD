## <!-- Powered by BMAD™ Core -->

docOutputLocation: outputs/{slug(question)}_{date}.csv
template: '{root}/templates/scan-output-tmpl.yaml'

---

# Facilitate Social Media Scan Task

Scan selected social platforms for answers to a user question and export normalized results as CSV. Analysis is not part of this task.

## Process

### Step 1: Scan Setup

Ask 2 context questions one at a time:

1. Which social media would you like to scan?
   - A. Facebook
   - B. Reddit
   - C. TripAdvisor
   - D. All of them

2. Only if it is not clear from the question: Which country are you traveling to?

Defaults:
- Time range: last 24 months
- Language: English only
- Scope: user posts in travel groups plus replies and comments
- Exclude: page posts
- Depth: deep scan
- Minimum target: 30 relevant items per selected platform

### Step 2: Execute and Principles

Run the scan automatically using the defaults from Step 1.

- Execution
  - Scan sequentially: process one platform at a time
  - Target at least 30 relevant items per platform if available
  - Continue to the next platform on failure

- Relevance filter
  - Include only user posts in travel groups
  - Include replies and comments
  - Exclude page posts
  - Language: English only
  - Time window: last 24 months

- Deduplication
  - Key: platform + author identifier or name + ISO date + first 80 characters of reply
  - Keep the earliest by date on collision

- Error handling and logging
  - Log all errors to: outputs/{slug(question)}_{date}-errors.log
  - Do not print errors in chat

### Step 3: Document Output

Use the template at `{root}/templates/scan-output-tmpl.yaml`.

This template generates two files that share the same basename:
- CSV: `outputs/{slug(question)}_{date}.csv`
- Metadata YAML: `outputs/{slug(question)}_{date}.meta.yaml`

Metadata YAML fields:
- `question`: original user question
- `location`: detected or user provided, or "unspecified"
- `time_period.start`: date minus 24 months
- `time_period.end`: current date
- `category`: auto detected from keywords. One of Hotel, Activity, Transportation, Food, Comparison, Other

CSV columns:
- `platform`
- `name`
- `date`
- `reply`
- `likes`
- `permalink`

CSV rules:
- `date` must be ISO 8601 UTC
- `reply` must be single line with quotes escaped
- `likes` must be integer. Default 0 if missing
- `permalink` must point to the specific post or comment

## Key Principles

- Accuracy over quantity
  Collect only content that directly answers the question. Ignore promotional or unrelated content.

- Traceability
  Always capture a working permalink for every row.

- Consistency
  Apply the same filters across platforms. Language, post type, date range, and relevance.

- Deduplication
  Ensure each result is unique according to the defined key.

- Error isolation
  Never stop the whole scan because one platform fails. Write errors to the log file only.

- Non interactive execution
  After Step 1 the scan runs to completion without further questions.

- Compliance
  Respect platform terms and rate limits. Use approved access methods.

- Future proofing
  Preserve metadata in the YAML file so later analysis can run without rescanning.
