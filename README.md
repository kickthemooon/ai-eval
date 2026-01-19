# ai-eval
A reusable GitHub Action that uses Claude AI to evaluate input content against a configurable prompt and returns a structured approval/rejection decision.

## Features

- 🤖 Uses Anthropic Claude AI with structured outputs for reliable JSON responses
- 📝 Configurable system prompt for any evaluation criteria
- 📄 Accepts input as direct text or from a file
- ✅ Returns consistent `approved` (boolean) and `reason` (string) outputs
- 🔒 Secure API key handling via input or environment variable
- ⚡ Automatic input truncation to prevent API limits

## Usage

### Basic Example (Direct Input)

```yaml
- name: AI Evaluate
  id: ai_eval
  uses: ./.github/actions/ai-evaluate
  with:
    prompt: |
      You are an expert code reviewer.
      APPROVE if the code follows best practices.
      REJECT if there are security issues or bugs.
    input: |
      function getUserData(id) {
        return db.query(`SELECT * FROM users WHERE id = ${id}`);
      }
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

- name: Check Result
  run: |
    echo "Approved: ${{ steps.ai_eval.outputs.approved }}"
    echo "Reason: ${{ steps.ai_eval.outputs.reason }}"
```

### File Input Example

```yaml
- name: Get PR Diff
  id: diff
  run: |
    gh pr diff ${{ github.event.pull_request.number }} > pr_diff.txt
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

- name: AI Review PR
  id: ai_review
  uses: ./.github/actions/ai-evaluate
  with:
    prompt: |
      You are a senior engineer reviewing pull requests.
      APPROVE if changes are safe and follow conventions.
      REJECT if there are issues that need addressing.
    input_file: pr_diff.txt
    model: claude-opus-4-5-20251101
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### With API Key as Input

```yaml
- name: AI Evaluate
  uses: ./.github/actions/ai-evaluate
  with:
    prompt: "Your evaluation criteria here..."
    input: "Content to evaluate"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `prompt` | System prompt defining the evaluation criteria | ✅ Yes | - |
| `input` | Direct text input to evaluate | ❌ No* | `''` |
| `input_file` | Path to a text file to evaluate | ❌ No* | `''` |
| `model` | Claude model to use | ❌ No | `claude-opus-4-5-20251101` |
| `max_input_chars` | Maximum input characters (truncated if exceeded) | ❌ No | `50000` |
| `anthropic_api_key` | Anthropic API key | ❌ No | Uses `ANTHROPIC_API_KEY` env var |

> **Note:** Either `input` or `input_file` must be provided, but not both.

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `approved` | Whether the input was approved | `true` or `false` |
| `reason` | Explanation for the decision | `"The code follows security best practices..."` |
| `response_json` | Full JSON response from AI | `{"approved": true, "reason": "..."}` |

## Response Format

The AI always returns a JSON object with this structure:

```json
{
  "approved": true,
  "reason": "Detailed explanation of the decision with supporting observations..."
}
```

## Writing Effective Prompts

For best results, structure your prompt to clearly define:

1. **Role**: What expert perspective should the AI take?
2. **Approval Criteria**: When should the input be approved?
3. **Rejection Criteria**: When should the input be rejected?
4. **Formatting Preferences**: How should the reasoning be formatted?

### Example Prompt Template

```
You are an expert [ROLE] evaluating [WHAT].

APPROVE the input if:
- [Condition 1]
- [Condition 2]
- [Condition 3]

REJECT the input if:
- [Condition 1]
- [Condition 2]
- [Condition 3]

FORMAT YOUR REASONING using markdown:
- Use **bold** for key findings
- Use bullet points for multiple observations
- Be concise but thorough
```

## Use Cases

### 1. Kubernetes Resource Review

```yaml
prompt: |
  You are a DevOps engineer reviewing Kubernetes resource changes.
  
  APPROVE if:
  - Changes are limited to resource requests/limits
  - Values are within reasonable ranges
  
  REJECT if:
  - Changes affect non-resource configurations
  - Values are unreasonably high or dangerously low
```

### 2. Security Audit

```yaml
prompt: |
  You are a security engineer auditing code changes.
  
  APPROVE if:
  - No security vulnerabilities detected
  - Follows secure coding practices
  
  REJECT if:
  - SQL injection risks
  - Hardcoded credentials
  - Insecure cryptographic practices
```

### 3. Documentation Review

```yaml
prompt: |
  You are a technical writer reviewing documentation.
  
  APPROVE if:
  - Clear and well-structured
  - Accurate technical information
  - Proper grammar and formatting
  
  REJECT if:
  - Missing critical information
  - Factual errors
  - Poor readability
```

### 4. Configuration Validation

```yaml
prompt: |
  You are a platform engineer validating Terraform configurations.
  
  APPROVE if:
  - Follows naming conventions
  - Uses appropriate instance sizes
  - Has proper tagging
  
  REJECT if:
  - Missing required tags
  - Oversized or undersized resources
  - Security group misconfigurations
```

## Error Handling

The action handles various error scenarios:

| Scenario | Behavior |
|----------|----------|
| No input provided | Action fails with error message |
| Both `input` and `input_file` provided | Action fails with error message |
| Input file not found | Action fails with error message |
| No API key provided | Action fails with error message |
| API returns non-200 status | Action fails, outputs contain error details |
| API returns empty response | Action fails, outputs contain error details |
| Input exceeds max chars | Input is truncated with warning |

## Requirements

- **Anthropic API Key**: Required for Claude API access
- **jq**: Used for JSON processing (pre-installed on GitHub runners)
- **curl**: Used for API calls (pre-installed on GitHub runners)

## License

MIT
