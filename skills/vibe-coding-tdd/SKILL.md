---
name: vibe-coding-tdd
description: |
  Test-driven development guide — enforces define-test → fail → implement → verify cycle during BUILD phase.
  Use when: (1) User says "write tests first", "TDD", "test-driven", "write test for [feature]",
  (2) vibe-coding-build activates this before implementing any function/component,
  (3) User says "add tests", "unit test this", "integration test", "test coverage",
  (4) A feature is marked must-have in IMPLEMENTATION_PLAN and has no existing tests.
  Outputs test file stubs → implementation prompt → verification checklist.
  Supports: Jest/Vitest (JS/TS), pytest (Python), RSpec (Ruby), Go testing.
---

# Vibe Coding — TDD Guide

Enforce test-first development. Cycle: **define test → run (fail) → implement → run (pass) → refactor**.

## Entry Router

```
Called directly by user (no feature context)?
→ READ: ## Direct Activation

Called by vibe-coding-build with feature context?
→ Detect framework → READ: ## [Framework] Template → output TDD steps
```

**Jump directly to the matching ## [Framework] Template section. Do not read all templates — only the one matching the detected framework.**

---

## Direct Activation

```
1. Read progress.txt — get framework and active feature
2. If missing or no CURRENT_PHASE listed:
   Ask: "What feature are you writing tests for? What framework?
   (e.g. Next.js + Vitest, FastAPI + pytest, Go)"
3. Proceed as if BUILD passed the context
```

---

## Framework Detection

```
Node.js/TypeScript:
  vitest in package.json → Vitest
  jest in package.json → Jest
  default → recommend Vitest

Python: pytest in requirements.txt → pytest (default: recommend pytest)
Ruby: default → READ: ## RSpec Template
Go: default → built-in testing package → READ: ## Go Template
Rust: default → built-in #[test] → READ: ## Rust Template
```

Then READ: ## [Vitest/Jest Template | pytest Template | RSpec Template | Go Template | Rust Template]

---

## Vitest/Jest Template

```typescript
// tests/[feature].test.ts
import { describe, it, expect, vi } from 'vitest'
import { [functionName] } from '../src/[module]'

describe('[Feature Name]', () => {
  it('should [expected behavior] when [input condition]', () => {
    const result = [functionName]([test data])
    expect(result).toEqual([expected output])
  })
  it('should throw [ErrorType] when [invalid input]', () => {
    expect(() => [functionName](invalidInput)).toThrow('[error message]')
  })
})
```

Component tests (React Testing Library):
```typescript
it('calls [handler] when [action]', () => {
  const mockHandler = vi.fn()
  render(<[Component] on[Action]={mockHandler} />)
  fireEvent.click(screen.getByRole('button', { name: '[label]' }))
  expect(mockHandler).toHaveBeenCalledOnce()
})
```

---

## pytest Template

```python
# tests/test_[feature].py
import pytest
from src.[module] import [function_name]

class Test[FeatureName]:
    def test_[behavior]_when_[condition](self):
        result = [function_name]([input])
        assert result == [expected]

    def test_raises_[error]_when_[invalid](self):
        with pytest.raises([ErrorType]):
            [function_name](invalid_input)
```

---

## RSpec Template

```ruby
# spec/[feature]_spec.rb
require 'rails_helper'  # or 'spec_helper' for non-Rails
require_relative '../lib/[module]'  # adjust path

RSpec.describe [ClassName], type: :model do
  describe '#[method_name]' do
    context 'when [valid condition]' do
      it 'returns [expected result]' do
        subject = described_class.new([params])
        expect(subject.[method]).to eq([expected])
      end
    end

    context 'when [invalid condition]' do
      it 'raises [ErrorClass]' do
        expect { described_class.new([bad params]).[method] }.to raise_error([ErrorClass])
      end
    end
  end
end
```

Run command: `bundle exec rspec spec/[feature]_spec.rb`

---

## Go Template

```go
// [feature]_test.go
func Test[Feature](t *testing.T) {
    tests := []struct{ name string; input [T]; expected [T]; wantErr bool }{
        {"happy path", [input], [expected], false},
        {"error case", [bad input], [zero], true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := [Func](tt.input)
            if (err != nil) != tt.wantErr { t.Errorf("err: %v", err) }
            if result != tt.expected { t.Errorf("got %v, want %v", result, tt.expected) }
        })
    }
}
```

---

## Rust Template

```rust
// In the same file (unit tests) or tests/[feature].rs (integration)
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_[behavior]_when_[condition]() {
        let result = [function_name]([input]);
        assert_eq!(result, [expected]);
    }

    #[test]
    #[should_panic(expected = "[error message]")]
    fn test_panics_when_[invalid]() {
        [function_name]([bad_input]);
    }
}
```

Run command: `cargo test`

---

## TDD Output

After generating test file, output:

```
TDD: [Feature Name]
===================
STEP 1 — TESTS WRITTEN: [N] test cases
File: tests/[feature].test.ts

STEP 2 — RUN TO FAIL:
Command: [npm test | pytest | go test ./...]
Expected: All [N] tests FAIL. If any pass without implementation — test is wrong.

STEP 3 — IMPLEMENT:
File to create: src/[module].ts
[Describe expected implementation — do NOT write it yet]

STEP 4 — RUN TO PASS:
Command: [same as step 2]
Expected: All [N] tests PASS. Coverage: [target]%

STEP 5 — VERIFY:
[ ] All tests pass
[ ] No tests skipped
[ ] Coverage meets threshold
[ ] Edge cases covered
```

---

## Coverage Thresholds

| Code type | Minimum |
|-----------|---------|
| Core business logic | 80% |
| API endpoints | 70% |
| UI components | 60% |
| Utilities/helpers | 90% |

Skip coverage for: UI layout/styling, config files, type definitions, generated code.

---

## Mock Strategy

Mock: external APIs, DB queries (unit tests), file system, time/date, random values.
Do NOT mock: the function under test, pure utilities it uses, business logic dependencies.

```typescript
vi.mock('../src/lib/httpClient', () => ({
  get: vi.fn().mockResolvedValue({ data: { id: 1 } }),
}))
```

---

## Progress Tracking

After tests written and passing, write to progress.txt:
```
TDD:
  feature: [name]
  tests_written: [N]
  tests_passing: [N]
  coverage: [N]%
```
