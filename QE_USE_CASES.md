# 🧪 QE Use Cases — shinu-llm-eval-toolkit

> Practical Quality Engineering applications using `outlines` for structured LLM evaluation.

---

## Table of Contents

1. [AI-Powered Test Result Classification](#1-ai-powered-test-result-classification)
2. [Structured Bug Report Extraction](#2-structured-bug-report-extraction)
3. [LLM-Based Test Case Generation](#3-llm-based-test-case-generation)
4. [Automated Severity & Priority Triage](#4-automated-severity--priority-triage)
5. [Log Analysis with Regex Constraints](#5-log-analysis-with-regex-constraints)
6. [RAG Response Evaluation](#6-rag-response-evaluation)
7. [Test Coverage Gap Analysis](#7-test-coverage-gap-analysis)

---

## 1. AI-Powered Test Result Classification

Classify test outcomes from free-text CI/CD logs or test runner output.

```python
import outlines
from typing import Literal

model = outlines.from_transformers(...)  # or .from_openai()

test_log = """
Test: UserLoginTest :: test_valid_login
Result: All assertions passed. Response time 234ms.
No errors. Session created successfully.
"""

outcome = model(
    f"Classify this test result:\n{test_log}",
    Literal["PASS", "FAIL", "FLAKY", "SKIP", "ERROR"]
)
# → "PASS"

# Batch classify multiple tests
logs = [
    "Test: API Tests :: test_create_user — AssertionError: status 500",
    "Test: UI Tests :: test_login — TimeoutError: element not found",
]

classifications = [model(log, Literal["PASS","FAIL","FLAKY","SKIP","ERROR"]) for log in logs]
```

---

## 2. Structured Bug Report Extraction

Extract structured bug reports from unstructured Jira tickets, Slack messages, or emails.

```python
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import date
import outlines

class BugReport(BaseModel):
    title: str = Field(description="Short bug title")
    severity: Literal["Critical", "Major", "Minor", "Trivial"]
    affected_component: str
    steps_to_reproduce: List[str]
    expected_vs_actual: str
    environment: Optional[str] = None
    suggested_priority: Literal["P0", "P1", "P2", "P3"]
    likely_root_cause: Optional[str] = None

slack_message = """
"The login page is broken!! When I click submit nothing happens.
This is blocking all our UAT testing. Happening on Chrome v120,
Windows 11. Users are stuck at login screen. @devs please fix ASAP!"
"""

bug = model(slack_message, BugReport)
# bug.title → "Login page submit button unresponsive"
# bug.severity → "Critical"
# bug.suggested_priority → "P0"
# bug.steps_to_reproduce → ["Navigate to login page", "Enter credentials", "Click Submit"]
print(f"[{bug.severity}] {bug.title} — Priority: {bug.suggested_priority}")
```

---

## 3. LLM-Based Test Case Generation

Generate structured test cases from plain-text requirements.

```python
from pydantic import BaseModel, Field
from typing import List

class TestCase(BaseModel):
    id: str
    title: str
    preconditions: List[str]
    test_steps: List[str]
    expected_results: List[str]
    test_data: Optional[str] = None
    automation_ready: bool

class TestSuite(BaseModel):
    feature: str
    test_cases: List[TestCase]

requirement = """
As a user, I want to reset my password via email.
I enter my registered email, receive a reset link,
click it, and set a new password. The link expires in 24 hours.
"""

suite = model(requirement, TestSuite)
for tc in suite.test_cases:
    print(f"\n### {tc.id}: {tc.title}")
    print(f"Steps: {len(tc.test_steps)} | Auto-ready: {tc.automation_ready}")
```

---

## 4. Automated Severity & Priority Triage

Use outlines to standardize bug triage across teams — eliminating human inconsistency.

```python
from pydantic import BaseModel
from typing import Optional

class TriageResult(BaseModel):
    severity: Literal["S1-Critical", "S2-Major", "S3-Minor", "S4-Trivial"]
    priority: Literal["P0-Immediate", "P1-High", "P2-Medium", "P3-Low"]
    impact_score: int  # 1-10
    urgency_score: int  # 1-10
    recommended_team: Optional[str] = None
    rationale: str

bug_desc = """
Production issue: All customer invoices fail to generate after last night's deployment.
Affects 100% of billing users. No workaround. Revenue impact ~$50K/hour.
"""

triage = model(bug_desc, TriageResult)
# triage.severity → "S1-Critical"
# triage.priority → "P0-Immediate"
# triage.impact_score → 10

# Compare with a minor issue:
minor = """
UI glitch: Button color is slightly off on the settings page.
Only affects dark mode users. No functional impact.
"""
triage2 = model(minor, TriageResult)
# triage2.severity → "S4-Trivial"
# triage2.priority → "P3-Low"
```

---

## 5. Log Analysis with Regex Constraints

Parse application logs with guaranteed format compliance using regex constraints.

```python
import outlines

# Define a regex pattern for log entries
log_pattern = r"\[(ERROR|WARN|INFO|DEBUG)\] \d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}Z - (.{5,200})"

log_line = "[ERROR] 2026-07-20T14:23:11Z - Database connection timeout after 30s"

parsed = model(
    f"Extract log level and message matching: {log_pattern}\nLog: {log_line}",
    log_pattern
)
# → "ERROR 2026-07-20T14:23:11Z - Database connection timeout after 30s"

# Count error types in log batch
error_types = Literal[
    "DATABASE_TIMEOUT", "AUTH_FAILURE", "API_RATE_LIMIT",
    "NULL_POINTER", "MEMORY_OVERFLOW", "UNKNOWN"
]

errors = [
    "FATAL: OutOfMemoryError: Java heap space",
    "ERROR: 429 Too Many Requests — rate limit exceeded",
]
for err in errors:
    etype = model(f"Classify this error: {err}", error_types)
    print(f"{etype}: {err[:50]}...")
```

---

## 6. RAG Response Evaluation

Evaluate RAG system outputs using structured scoring — the foundation of LLM evaluation.

```python
from pydantic import BaseModel, Field
from typing import List

class RagEvaluation(BaseModel):
    faithfulness_score: int = Field(ge=1, le=5, description="How faithful to source context (1-5)")
    answer_relevancy: int = Field(ge=1, le=5, description="How relevant to the question (1-5)")
    hallucination_detected: bool
    hallucination_excerpts: List[str]
    completeness: int = Field(ge=1, le=5)
    improvement_suggestions: Optional[str] = None

question = "What is the refund policy for annual subscriptions?"
context = "Annual subscriptions can be refunded within 30 days of purchase. Monthly subscriptions have a 7-day refund window."
response = "Customers can get a full refund on annual plans within 30 days, and monthly plans within a week."

eval_result = model(
    f"Question: {question}\nContext: {context}\nResponse: {response}\nEvaluate faithfulness and quality.",
    RagEvaluation
)
# eval_result.faithfulness_score → 5 (no hallucination)
# eval_result.hallucination_detected → False

# Now test a hallucinated response:
bad_response = "Annual subscriptions can be refunded at any time with no questions asked."
eval_result2 = model(
    f"Question: {question}\nContext: {context}\nResponse: {bad_response}\nEvaluate faithfulness and quality.",
    RagEvaluation
)
# eval_result2.faithfulness_score → 1
# eval_result2.hallucination_detected → True
```

---

## 7. Test Coverage Gap Analysis

Identify which functions/classes lack test coverage from code analysis.

```python
from pydantic import BaseModel
from typing import List

class CoverageGap(BaseModel):
    function_name: str
    file_path: str
    has_tests: bool
    risk_level: Literal["HIGH", "MEDIUM", "LOW"]
    suggested_test_scenarios: List[str]

class CoverageReport(BaseModel):
    total_functions: int
    covered: int
    uncovered: int
    gaps: List[CoverageGap]
    overall_risk: str

code_snippet = """
# src/payment/processor.py
def process_refund(user_id, amount, currency):
    # validates and processes refund
    pass

def calculate_tax(subtotal, tax_rate):
    return subtotal * tax_rate

# tests/test_payment.py
def test_calculate_tax():
    assert calculate_tax(100, 0.1) == 10.0
"""

report = model(code_snippet, CoverageReport)
for gap in report.gaps:
    if not gap.has_tests:
        print(f"⚠️ {gap.function_name} ({gap.risk_level} risk)")
        print(f"   Suggested: {gap.suggested_test_scenarios[0]}")
```

---

## Integration with Shinu AI Labs Toolchain

| Tool | Integration Point |
|:-----|:------------------|
| **QA Copilot** | Use outlines for structured RAG evaluation and answer classification |
| **Bug Triage Agent** | Replace free-text severity assessment with outlines-validated Pydantic models |
| **LLM-Eval-AI-Engineering** | Supplement DeepEval metrics with outlines-based faithfulness scoring |
| **Browser Assistant** | Structured extraction of page content into test assertions |

---

## References

- [Outlines Documentation](https://dottxt-ai.github.io/outlines/)
- [Outlines Paper — Efficient Guided Generation](https://arxiv.org/abs/2307.09702)
- [Shinu AI Labs Blog](https://www.shinuailabs.com/blog)
