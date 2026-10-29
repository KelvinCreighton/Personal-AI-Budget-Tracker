# Personal AI Budget Tracker

A local-first spending monitor that uses the installed Gemini CLI to read account
statements, records transactions in a private SQLite database, and writes monthly
Markdown reports with category trends, cash-flow savings, coverage status, and
possible reimbursement matches.

## Quick start

1. Install and sign in to [Gemini CLI](https://github.com/google-gemini/gemini-cli).
   Check that `gemini --version` works in the same terminal where you run this app.
2. Put statement files in `Uploads/`. Supported formats: PDF, CSV, TXT, Markdown,
   DOCX, HTML, and JSON.
3. Run:

   ```sh
   python3 budget.py run
   ```

4. Read the generated reports in `reports/`. Processed originals move to
   `Archived/`. Both directories are private and ignored by Git.

Other commands:

```sh
python3 budget.py status  # pending files and local record counts
python3 budget.py report  # regenerate reports without calling Gemini
python3 budget.py run --keep  # process files but leave originals in Uploads/
python3 budget.py transactions --month 2026-10  # inspect records and their IDs
python3 budget.py edit 42 --category Groceries  # correct a classification
python3 budget.py edit 42 --reimbursable  # exclude a volunteer purchase from personal averages
```

Run `python3 budget.py run` each month after placing new statements in `Uploads/`.
The database remembers previously processed statements and skips duplicate
transactions, including overlapping statement periods. Re-running reports never
calls Gemini.

## What the reports mean

- **Earned** counts positive income deposits and excludes transfers and refunds.
  **Spent (gross)** counts
  negative non-transfer transactions. Credit-card payments and movements between
  your own accounts are excluded as transfers to avoid double-counting.
- **Saved** is earned income plus refunds received minus gross spending for the
  listed transactions. It can be negative. A reimbursement does not erase the
  original cash outflow; reports separately show marked purchases and possible
  later refunds, including refunds found in subsequent uploads.
- If statements use more than one currency, reports keep totals and category
  comparisons separate by currency; they do not apply exchange rates.
- Categories include housing, utilities, groceries, dining, transportation,
  vehicle, health, personal care, shopping, entertainment, travel, education,
  pets, gifts and donations, fees and interest, income, transfers, reimbursements,
  and other.
- A month is marked complete only when the calendar month has ended and known
  statement periods cover every day of that month. Otherwise it is marked **so
  far** with the reason. A statement period is not proof that every transaction
  was captured; review reports against your source statements.
- Category averages use prior complete months only. Explicitly marked
  reimbursable expenses are excluded from this personal-spending baseline.
  Trends and suggestions become more useful as complete months accumulate.
  Use `transactions` to inspect AI results and `edit` to correct categories,
  transaction kinds, descriptions, or reimbursement flags. Corrections are kept
  in the local database and reports refresh immediately.

## Privacy and data handling

`Uploads/` and `Archived/` are tracked only as empty directories. Git ignores all
files below them, along with `.data/` (the SQLite database) and `reports/`. Keep
these paths ignored if you customize `.gitignore`; do not force-add statement or
report files. The app does not upload anything to GitHub.

When processing, the statement is provided to the Gemini CLI, which sends the
prompt and file contents to Google's Gemini service under the account and plan
configured in your CLI. Check Google's current privacy and retention terms before
using statements containing sensitive information. The app itself stores the
resulting transaction details and archived originals locally in this project.
Protect the machine and backups that contain this folder.

## Accuracy and review

Gemini can misread dates, signs, merchants, categories, transfers, and
reimbursements. The database has transaction fingerprints to prevent overlapping
statements from double-counting the same date, description, amount, currency, and
account. Different accounts with the same label may need review. Check unusual
items and reimbursement matches against the original statement. If Gemini fails
to process a file, it remains in `Uploads/`; supported files that process
successfully are moved to `Archived/`.

The starter version writes private Markdown reports and keeps a local SQLite
history. It does not connect to banks, move money, or provide tax/accounting
advice. The reports are decision support: use verified transactions and your own
judgment before acting on a suggestion.
