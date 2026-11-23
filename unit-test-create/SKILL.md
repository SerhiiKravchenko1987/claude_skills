---
name: unit-test-create
description: Unit test creation. Use it if you need to create unit tests or increase unit test coverage for global, local classes in ABAP.
argument-hint: class name(s)
allowed-tools: Read, Edit, Write, mcp__sap-adt__CreateSession, mcp__sap-adt__DestroySession, mcp__sap-adt__GetClass, mcp__sap-adt__GetClassInclude, mcp__sap-adt__GetInterface, mcp__sap-adt__GetStructure, mcp__sap-adt__GetStructureFields, mcp__sap-adt__GetTable, mcp__sap-adt__GetTableFields, mcp__sap-adt__SaveClassTestInclude, mcp__sap-adt__ActivateObject, mcp__sap-adt__RunAbapUnit, mcp__sap-adt__GetCoverageResult, mcp__sap-adt__GetStatementCoverage, mcp__sap-adt__CheckSyntax, mcp__sap-adt__GetObjectStatus
---

How to create unit tests.

Write unit tests for the following class(es): $ARGUMENTS
If several classes are listed, process each one separately.
The branch coverage should be 100%.
If needed create a common base class for repeated methods GIVEN_ WHEN_ THEN_.
For each public method of the class under the test a standalone test class should be created.
Don't execute private and protected methods of the class under the test, only public methods.
Every test method name must start with verb.
At first declare GIVEN_ methods, then WHEN_ methods, then THEN_ methods.
Separate logical sections within a test method (GIVEN / WHEN / THEN blocks) with a blank line for readability.
All literals should be wrapped in class constants.
If constant is going to be used in several classes, then put it in ltc_base class.
Don't leave empty CATCH statements.
Selects from DB tables and CDS views should be mocked by framework CL_OSQL_TEST_ENVIRONMENT.
If needed you can use framework CL_CDS_TEST_ENVIRONMENT for CDS views instead of CL_OSQL_TEST_ENVIRONMENT.
FMs should be mocked by framework CL_FUNCTION_TEST_ENVIRONMENT.
If it is not possible to ensure 100% coverage, the maximum possible coverage should be provided.

## ABAP-specific rules

**Method name limit**: ABAP method names are limited to 30 characters. Count characters before finalizing any method name.

**OSQL test environment lifecycle**:
- Create in `CLASS_SETUP`, destroy in `CLASS_TEARDOWN` with `destroy()`, reset between tests with `clear_doubles()` in `SETUP`.
- Unlike FM env, the OSQL env can be shared in `ltc_base` via `CLASS-DATA` — do this when multiple test classes mock the same DB tables.
- `i_dependency_list` must include every table and CDS view accessed by the method under test; any table not listed will be queried against the real DB, breaking test isolation.
- GIVEN_ helpers must be called inside each test method body (after SETUP has already called `clear_doubles()`), not in SETUP itself.

**FM test environment lifecycle** (when FMs are called by the class under test):
- Create FM test environments in `CLASS_SETUP` using `CLASS-DATA`, not in `SETUP`/`TEARDOWN` — `CL_FUNCTION_TEST_ENVIRONMENT=>CREATE` registers doubles session-globally; calling it twice for the same FM raises `CX_FTD_EXCEPTION`.
- Destroy in `CLASS_TEARDOWN`.
- In `SETUP` (per-test reset), only call `clear_doubles()` — do not call `create()` again.
- Each test class creates only the FM environments it needs; do not create a shared FM environment in `ltc_base`.

**FM double method chain line-break gotcha**:
Splitting a chain ending with `->then_raise_classic_exception(...)` onto a new line causes "Unable to interpret" syntax error. Break at an earlier step using an intermediate variable:
```
DATA(lo_config) = lo_double->configure_call( )->ignore_all_parameters( ).
lo_config->then_raise_classic_exception( 'EXCEPTION_NAME' ).
```

## Workflow (mandatory)

1. Write or update the test class(es) directly, without asking for confirmation.
2. After writing, ALWAYS save and activate, then run the unit tests for each class.
3. If any test fails or does not compile, fix it and run again. Repeat until all tests pass.
4. Check the branch coverage. If it is below 100%, add tests and rerun.
5. Finish with a short report: class, number of tests, pass/fail, coverage.
Never report the task as done without a successful test run.