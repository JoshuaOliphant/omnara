# Test Execution

Execute the test suite and report results in structured JSON format.

## Instructions

- Run the complete test suite using Omnara's Makefile
- Capture all test output (stdout and stderr)
- Parse test results to determine pass/fail status
- Return results in the specified JSON format

## Test Commands

### Run All Tests
```bash
make test
```

### Run Unit Tests Only (Skip Docker-dependent integration tests)
```bash
make test-unit
```

### Run Integration Tests (Requires Docker)
```bash
make test-integration
```

### Run Specific Test
```bash
make test-k ARGS="test_name_pattern"
```

## Test Execution Strategy

1. **Start with unit tests**: `make test-unit`
   - These are fast and don't require Docker
   - Should always pass before proceeding

2. **Run integration tests**: `make test-integration`
   - Requires Docker to be running
   - Tests database interactions, API endpoints, etc.
   - May be skipped if Docker is unavailable

3. **Capture full output**: Save both stdout and stderr

4. **Parse results**: Extract:
   - Total tests run
   - Tests passed
   - Tests failed
   - Test errors
   - Coverage percentage (if available)

## Output Format

Return a JSON object in a markdown code block:

```json
{
  "test_summary": "<brief summary of test results>",
  "success": true/false,
  "tests_run": <number>,
  "tests_passed": <number>,
  "tests_failed": <number>,
  "test_errors": <number>,
  "coverage_percent": <number or null>,
  "failing_tests": [
    {
      "test_name": "<test_module::test_function>",
      "error_message": "<error message>",
      "test_file": "<path to test file>"
    }
  ],
  "test_output": "<full test output, truncated if >1000 chars>"
}
```

## Example

After running tests, return:

```json
{
  "test_summary": "Unit tests passed (45/45), 2 integration tests failed",
  "success": false,
  "tests_run": 47,
  "tests_passed": 45,
  "tests_failed": 2,
  "test_errors": 0,
  "coverage_percent": 87.5,
  "failing_tests": [
    {
      "test_name": "src/backend/tests/test_auth.py::test_jwt_validation",
      "error_message": "AssertionError: JWT token validation failed",
      "test_file": "src/backend/tests/test_auth.py"
    },
    {
      "test_name": "src/servers/tests/test_messages.py::test_message_queue",
      "error_message": "TimeoutError: Message queue did not respond",
      "test_file": "src/servers/tests/test_messages.py"
    }
  ],
  "test_output": "===== test session starts =====\nplatform darwin -- Python 3.11.5...\n..."
}
```

## Error Handling

If tests cannot be run due to environment issues:

```json
{
  "test_summary": "Test execution failed: <reason>",
  "success": false,
  "tests_run": 0,
  "tests_passed": 0,
  "tests_failed": 0,
  "test_errors": 0,
  "coverage_percent": null,
  "failing_tests": [],
  "test_output": "<error message>"
}
```

## Notes

- **Docker requirement**: Integration tests need Docker running (`./dev-start.sh`)
- **Test isolation**: Each test should be independent
- **Coverage**: Coverage reports are in `htmlcov/` after running with `--cov`
- **Markers**: Use `-m "not integration"` to skip integration tests
- **Verbose**: Add `-v` or `-vv` for more detailed output

## Validation

After fixing any failing tests, re-run to ensure all pass:

```bash
make test
```
