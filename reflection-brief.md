# Reflection Brief — Harness Engineering Capstone

**Name:** Megha Shyam Avirneni
**Date:** 2026-09-26

## Environment

- Model(s): Claude/Anthropic model specified by the course; live API model execution was performed through the course-provided Vocareum model-provider endpoint.
- OS / Python: Linux / Python 3.13
- Approx. API spend: System 1 reported an estimated live-run cost of $0.1224. System 2 was executed through the course-provided model-provider resource and generated the required runtime evaluation artifacts.

---

## Part 1 — Per-system

### System 1 — Agentic loop

1. **Loop control.** 
   The live System 1 run generated a summary and eight per-claim JSONL traces, providing runtime evidence for the stop_reason-driven loop. The implementation is in `exercises/03-dynamic-decomposition/solution/claims_intake/loop.py`, where the loop continues on `tool_use`, returns on `end_turn`, and raises `UnexpectedStopReason` for unexpected stop reasons. The final System 1 test suite passed with **29 passed**, and the live run was recorded under `runs/20260926_105953`.

2. **Anti-pattern.**
   One anti-pattern checked by `tests/test_antipatterns.py` is using a hard-coded integer iteration limit instead of the model's `stop_reason`. Such a limit could terminate a valid multi-turn tool-use interaction before the model has completed the claim, or encourage arbitrary loop behavior. The local System 1 suite completed with **29 passed**, providing automated evidence that the implemented design satisfies the exercise checks.

3. **Tool design.**
   `lookup_policy` and `record_claim_fact` both operate on claim-related information, but their tool descriptions distinguish policy retrieval from recording a structured claim fact. The tool interface therefore gives the model a semantic distinction instead of relying on application-side text matching. Structured errors such as `is_error`, `error_category`, and `is_retryable` also allow the loop to distinguish permanent failures from transient failures and decide whether another attempt is appropriate. This behavior is covered by the passing System 1 test suite (**29 passed**).

4. **Your numbers.**
   The live run produced per-claim turn counts and token/cost information. The eight-claim run reported an estimated total cost of **$0.1224**. The final System 1 suite passed with **29 passed**.

---

### System 2 — Context strategy

5. **The reduction.**
   The live System 2 run generated `budget.json`. The baseline transcript measured **38,708 tokens**, while the assembled context measured **16,786 tokens**, giving a **56.63% reduction**. The final System 2 test suite produced **30 passed**.

6. **Summarize vs preserve.**
   The System 2 implementation separates information that can be compressed from information that must remain exact, particularly structured case facts and other information where byte-level fidelity matters. The final cumulative implementation was taken from `04-assemble-and-locate/solution` and the local suite produced **30 passed**. The live run generated the expected `budget.json` and recorded the assembled-context token budget.

7. **Facts block.**
   The live evaluation in `eval.jsonl` produced **6/6 passed**. The control evaluation in `eval_control.jsonl` showed Q1 as an unexpected pass and Q6 as the expected fail when the case-facts context was stripped. The final implementation also passed **30 tests**, while the live evaluation provides the runtime evidence for answerability and context preservation.

---

### System 3 — Claude Code config

8. **Path-scoped rules.**
   The file `.claude/rules/react.md` contains the path-scoped frontmatter:
   
   `paths:`
   `  - "src/components/**/*"`
   `  - "src/pages/**/*"`
   
   This is better than putting the convention in a directory-level `CLAUDE.md` when the same convention applies to selected file patterns across a repository. The rule activates based on the files being worked on rather than requiring the repository structure to mirror the convention boundaries. The System 3 aggregate suite produced **35 passed**.

9. **Forked skill.**
   The deploy skill contains `context: fork` and an explicit read-only allowlist including `Read`, `Grep`, `Glob`, and read-only Git/GitHub commands such as `Bash(git status:*)` and `Bash(git diff:*)`. Forking isolates the verbose deployment-check work from the main conversation, while the restricted tools prevent the skill from modifying files, pushing commits, or performing deployment actions. Without the fork/read-only restrictions, diagnostic work could pollute the main session or gain unnecessary mutation capability. The configuration was validated by the **35-passed** System 3 suite.

10. **Scope.**
    The project-level configuration is represented by `starter/CLAUDE.md` and the `.claude/standards/` and `.claude/rules/` files, which are intended to be shared with the repository. The user-level scope is represented by the `~/.claude/CLAUDE.md`, `~/.claude/commands/`, and `~/.claude/skills/` scope described in the project's `CLAUDE.md`. The completed project-level `CLAUDE.md` also imports `frontend.md`, `api.md`, `database.md`, and `testing.md` using `@` imports. The System 3 suite passed **35 tests**, and the explicit validator returned **OK**.

---

### System 4 — Orchestration

11. **Push work down.**
    The offline Shift C run reported **0 new defects** for the current query window while the resulting shift summary contained **3 high and 2 medium defects**, plus one low repeat defect. The warm-tier database is `data/warm.sqlite` and the run used the warm-store pipeline rather than exposing the entire historical dataset to the model. The resulting `data/hot_state.json` contains only the compact current summary and active alerts, demonstrating that historical retrieval and aggregation are pushed into deterministic application-side processing. The System 4 runtime completed successfully and the test suite produced **33 passed**.

12. **Crash recovery.**
    The System 4 recovery logic distinguishes whether existing state is sufficiently fresh to resume or stale enough to require a fresh run with an injected summary. A fresh start can be more reliable when the previous session's state is stale or incomplete because it reconstructs the working state from durable information instead of blindly trusting potentially inconsistent transient state. The concrete runtime artifact `data/hot_state.json` contains the current shift summary and threshold statuses, while the System 4 tests include recovery and state-management behavior and the complete suite produced **33 passed**.

13. **Small state.**
    My generated `data/hot_state.json` is **643 bytes**. It contains the current shift summary, three active alerts, and threshold statuses rather than the full defect history. Keeping this state bounded matters because the system is intended to run once per shift indefinitely; unbounded state would increase storage, context, parsing, and recovery costs over time. The actual 643-byte artifact demonstrates the bounded-state design.

---

## Part 2 — Synthesis

14. **Three layers.**

   **Model:** `Build a Claims Intake Agent with a stop_reason-Driven Loop/exercises/03-dynamic-decomposition/solution/claims_intake/loop.py` — the model-driven agent loop decides whether the interaction ends or requires additional tool use based on `stop_reason`.

   **Harness:** `Configure Claude Code for a Multi-Surface Monorepo Team/04-plan-mode-and-explore-decision-doc/solution/CLAUDE.md` — this defines shared project conventions and imports the modular frontend, API, database, and testing standards. The associated configuration suite produced **35 passed**.

   **Orchestration:** `Build a Multi-Shift Quality Monitoring System with Claude Orchestration/04-fork-scratchpad/solution/data/hot_state.json` — this is durable cross-session operational state containing the current shift summary, alerts, and threshold status. The System 4 test suite produced **33 passed** and the offline runtime generated the 643-byte state artifact.

15. **Deterministic vs prompt.**
    A deterministic behavior is the System 3 read-only tool allowlist in `.claude/skills/deploy-check/SKILL.md`, where permitted Git/GitHub commands are explicitly constrained. Another deterministic example is System 4's bounded hot-state artifact, which is measured at **643 bytes** in the actual run. Prompt guidance is appropriate for behavior such as asking an agent to summarize findings or choose an explanation style, where strict enforcement is unnecessary. Deterministic code is preferable when violating the behavior could cause data corruption, unauthorized mutation, or an uncontrolled resource cost.

16. **Context, two faces.**
    System 2 manages context **within a session** by pruning and assembling the most useful information into a bounded context, with a live run reducing the transcript from **38,708 to 16,786 tokens (56.63%)**, while its final test suite produced **30 passed**. System 4 manages context **across shifts/sessions** by persisting a compact `hot_state.json`, which was **643 bytes** in my actual run, and a **322-byte** scratchpad. Both apply the same principle: do not carry the entire history forward when a smaller, structured representation is sufficient. The mechanism differs: System 2 performs context engineering during assembly, while System 4 persists durable state between executions.

17. **Reliability you can't see in one run.**
    The System 4 test `test_two_forks_produce_independent_scratchpads` verifies that two forked hypotheses do not accidentally share mutable scratchpad state. A single successful run would not necessarily reveal this isolation problem because only one fork might be exercised. The test suite explicitly passed this test as part of the **33 passed** System 4 result. This matters because shared state could cause one hypothesis to contaminate another even when individual runs appear successful.

18. **Blast radius.**
    System 4 has a relatively contained blast radius because historical data access and state persistence are separated from the model-facing summary. The fork/scratchpad tests verify independent state, while `data/hot_state.json` provides a small durable state boundary of **643 bytes**. A practical kill switch is to stop the scheduled shift execution or disable the orchestration entry point before another shift processes the state. The read-only/forked design also limits what diagnostic work can mutate.

---

## Part 3 — Honest assessment

19. **What broke.**
    System 4 initially failed to collect tests because `shift_monitor/fork.py` contained accidental shell commands inside the Python source. I removed those invalid lines and reran the complete suite, which then produced **33 passed in 4.03s**. System 3 also initially had missing cumulative artifacts; copying the completed exercise artifacts into the cumulative starter allowed the full suite to reach **35 passed**. These fixes were verified by actual local test output rather than assuming the configuration was correct.

20. **What you'd change.**
    I would make the evidence-generation layer part of the implementation workflow instead of treating it as a final submission step. The first submission showed that passing tests alone were insufficient because runtime traces, budget reports, and reflection evidence were missing. In this run, I explicitly preserved `test-output.txt` for the systems and generated the System 4 offline artifacts: `data/hot_state.json` at **643 bytes** and `data/shift_scratchpad.jsonl` at **322 bytes**. The completed run now preserves the live evidence required for the API-backed systems, including System 1 traces/summary and System 2 budget/context/evaluation artifacts.