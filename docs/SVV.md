# Software Verification and Validation Plan: Mosaic
Generated: 2026-11-08

## 1. Introduction

This document is the baseline and framework for testing, verifying, and validating Mosaic. Mosaic is a Python package that lets users write code as prompts: they mark a stub, describe it in a docstring, and Mosaic uses the Claude Code SDK to write the code and splice it back into the file. Developers and testers use this document to know which tests exist, at which level they run, where they run, and what result each one expects. Every functional requirement in the SRS is covered by at least one test case.

## 2. Testing levels

| Level | What it covers | How | Where it runs | Data |
| --- | --- | --- | --- | --- |
| Unit | Each module's functions and classes: logic, boundary values, fault handling | pytest | GitHub Actions CI and local | Synthetic |
| Integration | Modules and components working together and with external software and systems: data flow, interfacing, inputs, outputs | pytest with a mocked `claude-agent-sdk`, mocked subprocess for package managers, and temporary git repos | GitHub Actions CI and local | Synthetic |
| System | The complete software against the SRS functional requirements | pytest with click's CliRunner running the full CLI, with a mocked `claude-agent-sdk` | GitHub Actions CI and local | Synthetic |
| End to end | Manual verification of the functional requirements via pre-built scenarios | Manual, with a real Claude account and a real Claude Code install | Local | Real-world data and execution |

## 3. Test records

Records from unit, integration, and system tests run in the CI pipeline are kept as GitHub Actions artifacts. End-to-end test records are kept as Markdown files in the repository, with the test results.

## 4. Test cases

| Requirement | Test | Level | Input and procedure | Expected |
| --- | --- | --- | --- | --- |
| REQ-CLI-ARGS-001 | TC-CLI-ARGS-001 | Unit | Run `compose` through CliRunner with a file, a folder, `a.py::Dog.get_breed`, `a.py::module_1`, `--model`, `--dry-run` and `--no-source`. | The config holds the file, the folder and both block IDs as targets, the model from `--model`, dry run on and source off. |
| REQ-CLI-ARGS-002 | TC-CLI-ARGS-002 | Unit | Run `restore` through CliRunner with one file and two block IDs. | RUN receives the file and both block IDs as restore targets. |
| REQ-CLI-ARGS-003 | TC-CLI-ARGS-003 | Unit | Run with (a) a missing path, (b) `a.py::`, (c) `a.py::nope`, (d) an unknown option. | An error, a non-zero exit and no file changes in each case. |
| REQ-CLI-ARGS-004 | TC-CLI-ARGS-004 | Integration | Run with and without `--model X`, with a mocked SDK. | Model X is set in the first run. No model is set in the second. |
| REQ-CLI-ARGS-005 | TC-CLI-ARGS-005 | System | Run `compose --dry-run` on three dependent blocks. | The order and the prompts are printed. No SDK call is made. Files and `.mosaic/` are unchanged. |
| REQ-CLI-ARGS-006 | TC-CLI-ARGS-006 | Unit | Build a prompt with the default setting, then with `--no-source`. | The default includes the YAML and the source. `--no-source` includes the YAML only. |
| REQ-CLI-RUN-001 | TC-CLI-RUN-001 | System | Run on a folder with code, class, method, module-level and test blocks. The mock returns valid code. | All blocks are composed and the exit code is 0. |
| REQ-CLI-RUN-001 | TC-CLI-RUN-002 | End to end | Use a real account. Compose the `vcf_ops.py` `average_qual` block, a `Dog` class block with a `get_breed` method block, and a test block with `#Tests: vcf_ops.py::average_qual`. Run `main` on a sample VCF, call `Dog().get_breed()`, then run pytest. | The correct average QUAL is printed. `get_breed` returns "shepherd". The test passes. |
| REQ-CLI-RUN-002 | TC-CLI-RUN-003 | System | Run on three independent blocks; the mock always returns invalid code for the middle one. | The first and third blocks are composed. The middle one is unchanged. |
| REQ-CLI-RUN-003 | TC-CLI-RUN-004 | System | Run on files with one marker error, one missing directive target and one cycle. | All three are reported. No SDK call is made. Exit code is non-zero. |
| REQ-CLI-RUN-004 | TC-CLI-RUN-005 | Integration | Run twice. In run (a) the mock raises "CLI not found" on the first block. In run (b) it raises "not authenticated" on the first block. | In each run, the run stops with a clear error and no further block is attempted. |
| REQ-CLI-RUN-005 | TC-CLI-RUN-006 | System | (a) B has `#Calls: A` and A always fails. (b) A class block always fails. | (a) B is skipped with a reason naming A. (b) Its method blocks are skipped with a reason naming the class. |
| REQ-CLI-RUN-006 | TC-CLI-RUN-007 | System | Run where one block fails, one is skipped and one succeeds. | The report lists the failed and skipped blocks with reasons. Exit code is non-zero. |
| REQ-PARSE-SCAN-001 | TC-PARSE-SCAN-001 | Unit | Search a folder tree with nested folders, a gitignored file, and files in `.venv/`, `.mosaic/` and `.git/`. | Only `.py` files that are not ignored are returned, including nested ones. |
| REQ-PARSE-SCAN-002 | TC-PARSE-SCAN-002 | Unit | Scan a file with no blocks, a code block, a test block, and a class block followed by an indented method block. | The count, kind and lines are correct for each block. |
| REQ-PARSE-SCAN-003 | TC-PARSE-SCAN-003 | Unit | Scan (a) a start marker with no end, (b) `#MosaicCode:start` closed by `#MosaicTest:end`, (c) a method block inside a class block. | An error with the line number in each case. |
| REQ-PARSE-SCAN-004 | TC-PARSE-SCAN-004 | Unit | Scan blocks that are (a) empty, (b) two functions, (c) a function with no docstring, plus valid function, class, method and module-level string blocks. | (a), (b) and (c) give errors. The valid blocks are accepted. |
| REQ-PARSE-SCAN-005 | TC-PARSE-SCAN-005 | Unit | Scan test blocks with (a) no `Behavior tested:`, (b) an empty `Behavior tested:`, (c) a filled one. | (a) and (b) give errors. (c) is accepted. |
| REQ-PARSE-DIR-001 | TC-PARSE-DIR-001 | Unit | Read a block with `#Calls: a, b`, a repeated `#Calls: c`, `#DependsOn: x.py::K.m` and `#CalledBy: main`. | Calls is [a, b, c]. DependsOn is [`x.py::K.m`], found in `x.py`. CalledBy is [main]. |
| REQ-PARSE-DIR-002 | TC-PARSE-DIR-002 | Unit | Read (a) `#Requires: x`, (b) `#Calls x` (no colon), (c) `#Tests: f` in a code block, (d) `#Calls: b.py::missing`. | A structure error with the line number in each case. No SDK call is made. |
| REQ-PARSE-AST-001 | TC-PARSE-AST-001 | Unit | Parse `async def f(self, path: str, n=3, *, k: int = 1)` inside `class Outer:` / `class Inner:`, plus one composed block. | Kind is `async def`. Parameters are in order with annotations and defaults as written, and `n` has no type. The composed block is marked composed. |
| REQ-PARSE-AST-002 | TC-PARSE-AST-002 | Unit | Parse `a.py`, which has a nested-class method block and two module-level blocks. | The IDs are `a.py::Outer.Inner.f`, `a.py::module_1` and `a.py::module_2`. |
| REQ-PARSE-AST-003 | TC-PARSE-AST-003 | Unit | Parse a file with a syntax error outside any block. | An error with the line number. |
| REQ-CTX-REF-001 | TC-CTX-REF-001 | Unit | In a temporary repo: `b.py` calls the block with `from a import f`, `c.py` calls it with `a.f()`, the block has `#CalledBy: d.py::g` in the file, and `e.py` is unrelated. | Callers are b, c and `d.py::g`. e is not included. |
| REQ-CTX-REF-001 | TC-CTX-REF-002 | Unit | A composed block whose record holds `#CalledBy: d.py::g`, with no directive in the file. | `d.py::g` is found as a caller. |
| REQ-CTX-REF-001 | TC-CTX-REF-003 | Unit | A caller that uses `getattr(a, "f")` and has no directive. | It is not found, and no error is raised (a known limit). |
| REQ-CTX-REF-002 | TC-CTX-REF-004 | Unit | A block with `#DependsOn: helper` whose composed code also calls the repo's `util()` and `numpy.mean`. | `helper` and `util` are included. numpy is excluded. |
| REQ-CTX-PACK-001 | TC-CTX-PACK-001 | Integration | Run PARSE then CTX for a code block and for a test block with `#Tests`. | Valid YAML that holds only the callers, the dependencies and the `#Tests` targets. |
| REQ-CTX-ORDER-001 | TC-CTX-ORDER-001 | Unit | X has `#Calls: Y`, Y has `#DependsOn: Z`, and W has `#CalledBy: X`. | Z comes before Y, and Y before X. W's position is not tied to X. |
| REQ-CTX-ORDER-002 | TC-CTX-ORDER-002 | Unit | Order a `Dog` class block with method blocks `Dog.get_breed` and `Dog.bark`, listed methods first. | `Dog` is ordered before both methods. |
| REQ-CTX-ORDER-003 | TC-CTX-ORDER-003 | Unit | (a) A calls B and B calls A. (b) f calls f. | (a) gives a cycle error naming A and B. (b) gives no error. |
| REQ-CTX-ORDER-004 | TC-CTX-ORDER-004 | System | Run `compose a.py::f`, where f has `#Calls: g` and g is uncomposed and not selected. | g is composed before f, and the report says so. |
| REQ-GEN-PROMPT-001 | TC-GEN-PROMPT-001 | Unit | Build prompts for a function block and a module-level block. | Each prompt contains the instruction, the signature (function only), the YAML, the source, and the no-change and no-additions rules. |
| REQ-GEN-PROMPT-002 | TC-GEN-PROMPT-002 | Unit | Build class prompts for `Dog` (a) without and (b) with a `Dog.__init__` method block. | (a) allows attributes and `__init__`. (b) allows attributes only. Neither allows other methods. |
| REQ-GEN-PROMPT-003 | TC-GEN-PROMPT-003 | Unit | Build a test prompt for the `test_get_quality_metrics` example. | The prompt contains the `Behavior tested:` text, the signature, the YAML with `get_quality_metrics`, and the one-unit rule. |
| REQ-GEN-PROMPT-004 | TC-GEN-PROMPT-004 | Unit | Build one prompt of each kind. | Every prompt has the AskUserQuestion rule and the package-report rule. |
| REQ-GEN-CLIENT-001 | TC-GEN-CLIENT-001 | Integration | Generate two blocks with a mocked SDK. | One new session is opened for each block attempt. |
| REQ-GEN-CLIENT-001 | TC-GEN-CLIENT-002 | End to end | Use a real account and one simple block. | Code is returned. No credential prompt is shown. |
| REQ-GEN-CLIENT-002 | TC-GEN-CLIENT-003 | Integration | Set the timeout to 1 s in code. The mock (a) stalls, (b) asks a question that is never answered. | The call ends in both cases, and the attempt counts as failed. |
| REQ-GEN-CLIENT-003 | TC-GEN-CLIENT-004 | Integration | Generate a block with no `ast` callers and no `#CalledBy`. | The only allowed tool is AskUserQuestion. |
| REQ-GEN-ASK-001 | TC-GEN-ASK-001 | Integration | The mock sends an AskUserQuestion, and CliRunner supplies an answer. | The question is printed, and the answer goes back to the same session. |
| REQ-GEN-ASK-001 | TC-GEN-ASK-002 | End to end | Use a real account and a deliberately vague instruction. | Claude asks a question in the terminal. The answer shapes the code. |
| REQ-GEN-SANDBOX-001 | TC-GEN-SANDBOX-001 | Integration | Generate a block called through `ast`, and a block called only through a cached `#CalledBy`. | Both run in a temporary copy that holds the candidate code, with the sandbox on and network allowed. |
| REQ-GEN-SANDBOX-002 | TC-GEN-SANDBOX-002 | Integration | Generate a called block in a uv, a poetry, a pixi and a plain project. | The allowed tools are exactly AskUserQuestion, Read, Grep, Glob, and Bash for `<runner> python -c "import <module>"`, with the correct runner each time. |
| REQ-GEN-SANDBOX-002 | TC-GEN-SANDBOX-003 | End to end | Use a real account, a called block, and an instruction that pushes Claude to write files, run tests or run `pip install`. | Those actions are denied. Only the import check runs. |
| REQ-GEN-SANDBOX-003 | TC-GEN-SANDBOX-004 | Integration | End a session after a mocked run in the copy. | The temporary folder is gone. A checksum of the original repo is unchanged. |
| REQ-GEN-VALID-001 | TC-GEN-VALID-001 | Unit | Validate (a) code with a syntax error, (b) two functions returned for one stub. | Both are rejected. |
| REQ-GEN-VALID-002 | TC-GEN-VALID-002 | Unit | Check stub `def f(a: int, b=2):` against (a) `async def`, (b) a renamed parameter, (c) swapped parameter order, (d) a dropped annotation, (e) a changed default, (f) an added parameter, (g) an exact match. | (a) to (f) are rejected. (g) is accepted. |
| REQ-GEN-VALID-003 | TC-GEN-VALID-003 | Unit | Check stub `class Dog(Animal):` against (a) a renamed class, (b) changed bases, (c) an added `bark` method, (d) `__init__` plus attributes, (e) `__init__` when a `Dog.__init__` block exists. | (a), (b), (c) and (e) are rejected. (d) is accepted. |
| REQ-GEN-VALID-004 | TC-GEN-VALID-004 | Integration | The mock returns invalid code, then invalid code, then valid code. | 3 sessions are opened, and the valid code is accepted. |
| REQ-GEN-VALID-004 | TC-GEN-VALID-005 | Integration | The mock returns a signature mismatch, then a timeout, then a network error. | After 3 attempts the block is unchanged, and the run continues. |
| REQ-COMP-SPLICE-001 | TC-COMP-SPLICE-001 | Unit | Splice code into a top-level block with directives and into an indented method block. | The directives and docstring are gone. The markers are kept with their indentation. The code is indented correctly. |
| REQ-COMP-SPLICE-002 | TC-COMP-SPLICE-002 | Unit | Splice one block into a file with comments and unusual formatting. | The bytes outside the block are identical to the input. |
| REQ-COMP-SPLICE-002 | TC-COMP-SPLICE-003 | System | Run a full `compose` (mocked SDK) on the same file. | The bytes outside the block in the written file are identical to the input, except the imports added at the top of the file. |
| REQ-COMP-IMPORT-001 | TC-COMP-IMPORT-001 | Unit | Add code that needs `statistics` to a file that (a) lacks the import, (b) already has it. | (a) One import is added at the top. (b) No duplicate is added. |
| REQ-COMP-DEPS-001 | TC-COMP-DEPS-001 | Integration | In uv, poetry and pixi projects, the model reports `pysam`, with subprocess mocked. Repeat with `pysam` already declared. | The add command is called once, with no version pin. It is not called when `pysam` is already declared. |
| REQ-COMP-DEPS-002 | TC-COMP-DEPS-002 | Integration | In each project type, the model reports `hypothesis` for a test block. | The dev-group form of the add command is called. |
| REQ-COMP-DEPS-003 | TC-COMP-DEPS-003 | Unit | Run in a plain project with no uv, poetry or pixi files. | No add command is run and no warning is shown. The import is still added. |
| REQ-COMP-DEPS-004 | TC-COMP-DEPS-004 | Integration | In a uv project, the model reports `pysam` and the mocked subprocess makes `uv add` fail. | A warning names `pysam` and tells the user to add it to the project configuration by hand. The written code is kept. The block is reported as a success. |
| REQ-COMP-WRITE-001 | TC-COMP-WRITE-001 | Integration | Write a valid result, then force an invalid result. | The valid result overwrites the file. The invalid result leaves the file unchanged and reports an error. |
| REQ-CACHE-MATCH-001 | TC-CACHE-MATCH-001 | Unit | Records exist for `a.py::f` and `a.py::Dog.get_breed`; the file holds both blocks in swapped order. | Each block is matched to its own record. |
| REQ-CACHE-MATCH-002 | TC-CACHE-MATCH-002 | Unit | With two module-level blocks: (a) insert a new uncomposed block above them, (b) swap the composed blocks, (c) hand-edit one composed block and move it. | (a) and (b) are matched by hash and renumbered in file order. (c) is matched by position, with a warning. |
| REQ-CACHE-STORE-001 | TC-CACHE-STORE-001 | Unit | Save a record for a block that has directives and one stored answer. | The `.mosaic/` record holds the ID, stub text, instruction and its hash, directives, key, code and its hash, and the answer. |
| REQ-CACHE-STORE-002 | TC-CACHE-STORE-002 | Unit | Change (a) the instruction, (b) the signature, (c) a directive, (d) a caller's source, (e) a stored answer, (f) unrelated code in the same file. | (a) to (e) change the key. (f) does not. |
| REQ-CACHE-STORE-003 | TC-CACHE-STORE-003 | Integration | Change the source of a composed block's callee, then run compose. | The prompt is built from the cached instruction and directives, and includes the stored answers. |
| REQ-CACHE-STORE-004 | TC-CACHE-STORE-004 | Integration | Run twice with no changes, then hand-edit the block and run again. | No SDK call is made on runs 2 and 3. Run 3 keeps the edit and shows the warning. |
| REQ-CACHE-STORE-005 | TC-CACHE-STORE-005 | Integration | Hand-edit a composed block, change a callee so its key changes, run and answer no. Repeat and answer yes. | Answering no leaves the block unchanged. Answering yes regenerates the block and overwrites it. |
| REQ-CACHE-STORE-006 | TC-CACHE-STORE-006 | Integration | Delete the `.mosaic/` record of a composed block, then run compose. | An error is reported, the block is skipped, the message suggests deleting the code and writing a new stub, and other blocks continue. |
| REQ-CACHE-RESTORE-001 | TC-CACHE-RESTORE-001 | System | Compose a class block, a method block and a function block, then run `mosaic restore a.py::Dog.get_breed`. | Only that block shows its original stub and directives inside its markers. Everything else is unchanged. |
| REQ-CACHE-RESTORE-001 | TC-CACHE-RESTORE-002 | End to end | Use a real account. Compose, restore, edit the instruction and one directive, then compose again. | The new code follows the edited instruction. The new record holds the edited directive. |
| REQ-CACHE-RESTORE-001 | TC-CACHE-RESTORE-004 | System | Compose all blocks in a.py (two composed blocks and one uncomposed block) and in b.py (one composed block), then run `mosaic restore a.py`. | Both composed blocks in a.py show their original stubs and directives inside their markers. The uncomposed block in a.py and all of b.py are unchanged. |
| REQ-CACHE-RESTORE-002 | TC-CACHE-RESTORE-003 | Unit | Restore a block with no matching record. | An error is reported and no file changes. |
