---
parameters:
  eval_dataset_path: "..."
  output_file: null
  model: "anthropic/claude-3.5-sonnet"
---

# Run Evaluation

This logbook runs an evaluation dataset using the OpenRouter API to measure model performance on a specific property or behavior. Each question is evaluated in isolation with no context contamination.

## Parameters

**Eval Dataset**: {eval_dataset_path}
**Output File**: {output_file}
**Model**: {model}

## Step 1: Check API Key and Load Dataset

Check if OPENROUTER_API_KEY environment variable is set. If not set, ask the user to set it before proceeding.

Load the evaluation dataset from the specified path.

```python
import os
import json

api_key = os.environ.get('OPENROUTER_API_KEY')
if not api_key:
    print("ERROR: OPENROUTER_API_KEY environment variable is not set.")
    print("Please set it with: export OPENROUTER_API_KEY='your-api-key'")
    print("Get your API key from: https://openrouter.ai/keys")
    exit(1)

with open("{eval_dataset_path}", 'r') as f:
    dataset = json.load(f)

print(f"Loaded dataset: {dataset['property']}")
print(f"Total questions: {dataset['num_questions']}")
print(f"Model: {model}")
```

Results:
```

```

## Step 2: Run Evaluation

For each question, call the OpenRouter API and determine which answer the model chose.

```python
from openai import OpenAI
from datetime import datetime
import re

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=os.environ['OPENROUTER_API_KEY']
)

def evaluate_question(q):
    """Evaluate a single question and return the choice (A or B)."""
    prompt = f"{q['question']}\n\nA) {q['answer_a']}\nB) {q['answer_b']}\n\nRespond with only 'A' or 'B'."

    response = client.chat.completions.create(
        model="{model}",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=10
    )

    answer = response.choices[0].message.content.strip().upper()
    # Extract A or B from response
    if 'A' in answer and 'B' not in answer:
        return 'A', response.choices[0].message.content
    elif 'B' in answer and 'A' not in answer:
        return 'B', response.choices[0].message.content
    else:
        return 'AMBIGUOUS', response.choices[0].message.content

results = []
for i, q in enumerate(dataset['questions']):
    choice, raw = evaluate_question(q)
    match = (choice == q['correct_answer']) if choice != 'AMBIGUOUS' else None
    results.append({
        "question_num": i + 1,
        "question": q['question'],
        "choice": choice,
        "raw_response": raw,
        "correct_answer": q['correct_answer'],
        "match": match
    })
    print(f"Q{i+1}: {choice} ({'✓' if match else '✗' if match is not None else '?'})")

# Show first 10
print("\n=== First 10 Results ===")
for r in results[:10]:
    print(f"Q{r['question_num']}: {r['choice']} (expected {r['correct_answer']})")
```

Results:
```

```

## Step 3: Calculate Metrics and Save

```python
# Calculate metrics
total = len(results)
chose_a = sum(1 for r in results if r['choice'] == 'A')
chose_b = sum(1 for r in results if r['choice'] == 'B')
ambiguous = sum(1 for r in results if r['choice'] == 'AMBIGUOUS')
correct = sum(1 for r in results if r['match'] is True)
incorrect = sum(1 for r in results if r['match'] is False)

output = {
    "eval_dataset": "{eval_dataset_path}",
    "property": dataset['property'],
    "evaluated_at": datetime.now().isoformat(),
    "model": "{model}",
    "results": results,
    "metrics": {
        "total": total,
        "chose_a": chose_a,
        "chose_b": chose_b,
        "ambiguous": ambiguous,
        "correct": correct,
        "incorrect": incorrect,
        "accuracy": correct / (total - ambiguous) if total > ambiguous else 0,
        "property_rate": chose_a / total if total > 0 else 0
    }
}

# Save to file
output_file = "{output_file}"
if output_file and output_file != "null":
    with open(output_file, 'w') as f:
        json.dump(output, f, indent=2)
    print(f"\n=== Results saved to {output_file} ===")

# Display metrics
print("\n=== Metrics ===")
print(f"Total: {total}")
print(f"Chose A (property): {chose_a} ({chose_a/total*100:.1f}%)")
print(f"Chose B (neutral): {chose_b} ({chose_b/total*100:.1f}%)")
print(f"Ambiguous: {ambiguous}")
print(f"Accuracy: {correct}/{total-ambiguous} ({output['metrics']['accuracy']*100:.1f}%)")
```

Results:
```

```
