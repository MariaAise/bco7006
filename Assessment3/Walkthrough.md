# Assessment 3 — Walkthrough Manual

A guide for *how to approach* the assignment. Not the solution.

-----

## 1. What you’re building

- An **Enrolment Process Screening System (EPSS)**
- Input: a student record + a list of units they want to enrol in
- Output: either an enrolment record (`enrol_code`, `unit_code`, `st_id`, `semester`) or a message explaining why it was rejected

-----

## 2. Pull the three rules out of the brief

The brief mixes the rules into prose. Extract them:

- **R1 — Prerequisites**: every prereq for the unit must be in `unit_passed`
- **R2 — Cap**: at most **2** units per enrolment request
- **R3 — Special restrictions**: `WOM1000` is female-only (this is the only one in the sample catalogue, but treat it as a *category* of rule, not a one-off)

If your code doesn’t enforce all three, you’ve missed part of the brief.

-----

## 3. Decide your data shapes first

Before writing any logic, sketch what each thing looks like:

```python
# Catalogue — sketch
catalogue = {
    "BCO7006": {"prereqs": [], "female_only": False, ...},
    ...
}

# Student — sketch
student = {"st_name": ..., "st_id": ..., "gender": ..., "unit_passed": [...]}

# Enrolment record — sketch
{"enrol_code": ..., "unit_code": ..., "st_id": ..., "semester": ...}
```

- Keys, not positions. `student["gender"]` beats `student[2]`
- Lists for things you’ll iterate (`unit_passed`, `enrolments`)
- Don’t decide on OOP yet — get the shapes right first

-----

## 4. Build order (each step runs on its own)

**Step 1 — Catalogue + one student** as plain dicts. Print them. Done.

**Step 2 — One screening function**:

```python
def can_enrol(student, unit_code, catalogue):
    # check R1 (prereqs)
    # check R3 (gender)
    # return (True, "ok") or (False, "reason")
```

Test it manually on 3 cases before moving on.

**Step 3 — The loop** over requested units, with the cap check:

```python
if len(requested) > 2:
    # reject everything, return
for unit_code in requested:
    ok, msg = can_enrol(...)
    if ok:
        # build & store enrolment record
```

**Step 4 — Wrap in a class**. This is where you satisfy the OOP criterion:

```python
class EnrolmentSystem:
    def __init__(self, catalogue): ...
    def enrol(self, student, requested, semester): ...
    def show_enrolments(self): ...
```

The functions from Step 2–3 become methods. Almost no logic changes.

**Step 5 — Add `interactive_enrol()`** that uses `input()` to collect a student + request from the keyboard.

-----

## 5. Test scenarios

Run these before submitting. Map each to a rule:

|#|Scenario                                       |Rule it tests|Expected                         |
|-|-----------------------------------------------|-------------|---------------------------------|
|1|Student has all prereqs, requests 2 valid units|R1 pass      |both enrol                       |
|2|Student missing one prereq                     |R1 fail      |partial: one enrols, one rejected|
|3|Student has no prereqs, requests advanced unit |R1 fail      |both rejected                    |
|4|Female student requests `WOM1000`              |R3 pass      |enrols                           |
|5|Male student requests `WOM1000`                |R3 fail      |rejected with reason             |
|6|Any student requests 3 units                   |R2 fail      |whole request rejected           |

If your output is missing one of these branches, your logic is incomplete.

-----

## 6. Minimal vs full showcase — which to aim for

|                        |Minimal                         |Full showcase                                                          |
|------------------------|--------------------------------|-----------------------------------------------------------------------|
|Classes                 |One: `EnrolmentSystem`          |Four: `Unit`, `ElectiveUnit`, `Student`, `Enrolment`, `EnrolmentSystem`|
|Catalogue               |`dict` of dicts                 |List of `Unit` / `ElectiveUnit` objects                                |
|Gender rule             |`if` inside the screening method|`ElectiveUnit` subclass overrides `is_eligible_for`                    |
|Uses                    |classes, methods                |classes + dataclasses + inheritance + composition + magic methods      |
|Rubric Criterion 3 (OOP)|passes                          |maxes                                                                  |
|Code volume             |~60 lines                       |~120 lines                                                             |
|Risk if rushed          |low                             |medium — easy to over-engineer and break                               |

- **Pick minimal if**: you’re new to classes and want a clean pass
- **Pick full showcase if**: you’re comfortable with `@dataclass`, `super()`, and `__str__` — and want top marks on Criterion 3
- Don’t mix-and-match unless you understand both. Half-dataclass / half-dict is the worst of both worlds

-----

## 7. The ChatGPT half — what to actually do

The brief wants **two versions** of your code, with evidence:

1. Your original (no ChatGPT)
1. An improved version (with ChatGPT)

Plus screenshots of the dialog and a 500-word reflection.

### Do it in this order

- **Finish your own code first.** Get all six test scenarios passing without help. Save the file (`epss_v1_no_chatgpt.py`)
- **Copy your code into ChatGPT.** Use specific prompts, not “make this better”

### Prompts that get useful answers

|Prompt                                                                                                                      |What it gets you                     |
|----------------------------------------------------------------------------------------------------------------------------|-------------------------------------|
|“Review this code for readability and suggest improvements without changing the logic.”                                     |naming, structure, comments          |
|“How would you refactor this to use `@dataclass` for the student and enrolment records?”                                    |OOP upgrade you can defend           |
|“What edge cases am I not handling? Suggest test cases I missed.”                                                           |extra test scenarios                 |
|“Rewrite the gender restriction so the system supports more restrictions in future without changing the screening function.”|inheritance / open-closed            |
|“Critique my algorithm — is the order of checks efficient?”                                                                 |reasoning material for the reflection|

### What to capture as evidence

- **Screenshot each prompt + response** as you go. Don’t try to reconstruct later
- Save the improved code as `epss_v2_with_chatgpt.py`
- Keep a short list: *what changed, why, and whether you agreed with ChatGPT*

### What to put in the 500-word reflection

The brief asks three questions — answer each in ~150 words:

1. **Does ChatGPT help you learn programming, and how?** — be specific. “It explained why `@dataclass` reduces boilerplate” beats “it was helpful”
1. **How much can you rely on it?** — give an example where it was wrong, or where you had to push back. Markers want critical thinking, not endorsement
1. **Which stage benefits most — algorithm design, coding, or testing?** — pick one, defend it with evidence from *your* experience on this task

### Honesty note

Don’t paste ChatGPT’s code, change three variable names, and call it yours. The reflection asks what *you* did. Markers can tell.

-----

## 8. Rubric self-check

Before submitting, point to where in your notebook each criterion lives:

- ☐ **Conditional statements** — `if` / `else` branches in the screening function
- ☐ **Iterative loops** — `for` or `while` over `requested_units`
- ☐ **OOP** — at least one class with state and methods
- ☐ **Functions** — at minimum: screening, enrolment, display, interactive input
- ☐ **Dataset management** — dicts and/or lists for catalogue, students, enrolments
- ☐ **Algorithm and logic** — flowchart at the top of the notebook matches the code

For Part 2: clarity, depth, comparison, application, grammar, word count.

-----

## 9. Common traps

- **Hardcoding the gender rule deep inside a function** — works, but makes adding the next restriction painful. Even in the minimal version, give it its own `if` block, not a one-liner
- **Forgetting the cap check** — easy to write the per-unit loop and skip R2 entirely
- **Returning `None` instead of a message** — markers want feedback the user can read
- **Capitalisation drift** — `"bco7006"` vs `"BCO7006"` will fail your prereq check silently. Pick one case and normalise input
- **No test output** — if your notebook only defines the class but never calls it, there’s nothing to grade
- **Treating `unit_passed` as a string** — it’s a list. `"BCO7006" in "BCO7006,BCO7000"` is `True` for the wrong reasons
- **OOP for its own sake** — five classes wrapping nothing is worse than one class doing real work
- **Submitting without the flowchart** — Criterion 6 explicitly asks for one

-----

*Last check before you submit: does running every cell top-to-bottom produce clean output with no errors?*
