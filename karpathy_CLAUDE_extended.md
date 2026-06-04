# CLAUDE.md - Behavioral Guidelines for LLM Coding

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

# EXAMPLES.md - Real-World Examples

Real-world code examples demonstrating the four principles. Each example shows what LLMs commonly do wrong and how to fix it.

---

## 1. Think Before Coding

### Example 1: Hidden Assumptions

**User Request:** "Add a feature to export user data"

**❌ What LLMs Do (Wrong Assumptions)**

```python
def export_users(format='json'):
    users = User.query.all()
    if format == 'json':
        with open('users.json', 'w') as f:
            json.dump([u.to_dict() for u in users], f)
    return f"Exported {len(users)} users"
```

Problems: assumed ALL users, assumed file location, assumed fields.

**✅ What Should Happen (Surface Assumptions)**

```
Before implementing, I need to clarify:
1. Scope: Export all users or filtered subset? (privacy implications)
2. Format: Download file, API endpoint, or background job?
3. Fields: Which user fields? (some might be sensitive)
4. Volume: How many users typically? (affects approach)
```

### Example 2: Multiple Interpretations

**User Request:** "Make the search faster"

**✅ What Should Happen**

```
"Make the search faster" could mean:
1. Faster response time (< 100ms) → indexes, caching — 2h
2. More concurrent searches (throughput) → async, pooling — 4h
3. Faster perceived speed (UX) → progressive loading — 3h

Which aspect matters most?
```

---

## 2. Simplicity First

### Example 1: Over-abstraction

**User Request:** "Add a function to calculate discount"

**❌ What LLMs Do (Overengineered)**

```python
class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float: pass

class PercentageDiscount(DiscountStrategy): ...
class FixedDiscount(DiscountStrategy): ...
class DiscountCalculator: ...
# 40+ lines for a simple calculation
```

**✅ What Should Happen (Simple)**

```python
def calculate_discount(amount: float, percent: float) -> float:
    return amount * (percent / 100)
```

### Example 2: Speculative Features

**User Request:** "Save user preferences to database"

**✅ What Should Happen (Just What Was Asked)**

```python
def save_preferences(db, user_id: int, preferences: dict):
    db.execute(
        "UPDATE users SET preferences = ? WHERE id = ?",
        (json.dumps(preferences), user_id)
    )
```

Add caching/validation/merging only when actually needed.

---

## 3. Surgical Changes

### Example 1: Drive-by Refactoring

**User Request:** "Fix the bug where empty emails crash the validator"

**❌ What LLMs Do:** Fix email bug + add username validation + add docstring + change comments.

**✅ What Should Happen:** Only fix the empty email lines. Nothing else.

### Example 2: Style Drift

**User Request:** "Add logging to the upload function"

**✅ What Should Happen:** Match existing style (single quotes, no type hints, existing patterns). Only add the logging lines.

---

## 4. Goal-Driven Execution

### Example 1: Vague vs. Verifiable

**User Request:** "Fix the authentication system"

**✅ What Should Happen**

```
Plan (e.g. "users stay logged in after password change"):
1. Write test: Change password → verify old session invalidated
   Verify: Test fails (reproduces bug)
2. Implement: Invalidate sessions on password change
   Verify: Test passes
3. Check edge cases → additional tests pass
4. Full test suite green
```

### Example 2: Test-First Verification

**User Request:** "The sorting breaks when there are duplicate scores"

**✅ What Should Happen**

```python
# 1. Write test that reproduces the issue
def test_sort_with_duplicate_scores():
    scores = [{'name': 'Alice', 'score': 100}, {'name': 'Bob', 'score': 100}]
    result = sort_scores(scores)
    assert result[0]['score'] == 100  # fails non-deterministically

# 2. Fix with stable sort
def sort_scores(scores):
    return sorted(scores, key=lambda x: (-x['score'], x['name']))
```

---

## Anti-Patterns Summary

| Principle | Anti-Pattern | Fix |
|-----------|-------------|-----|
| Think Before Coding | Silently assumes file format, fields, scope | List assumptions explicitly, ask |
| Simplicity First | Strategy pattern for single discount calculation | One function until complexity is needed |
| Surgical Changes | Reformats quotes, adds type hints while fixing bug | Only change lines that fix the issue |
| Goal-Driven | "I'll review and improve the code" | "Write test for bug X → make it pass → verify no regressions" |

**Good code is code that solves today's problem simply, not tomorrow's problem prematurely.**