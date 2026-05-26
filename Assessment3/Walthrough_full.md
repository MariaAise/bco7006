# Assessment 3 — Walkthrough Manual

A guide for *how to approach* the assignment. Not the solution.

Read this once before you open your notebook. Refer back to Section 8 (rubric) and Section 9 (traps) before you submit.

-----

## 1. What you’re building

You’re building an **Enrolment Process Screening System (EPSS)** for the University. It’s the software equivalent of the staff member who looks at your transcript and tells you “no, you can’t enrol in Machine Learning yet — you haven’t done Coding for BA.”

- **Input**: a student record + a list of units they want to enrol in
- **Output**: for each requested unit, either an enrolment record (`enrol_code`, `unit_code`, `st_id`, `semester`) or a clear message explaining why it was rejected
- **The system is dumb on purpose**. It doesn’t advise students on what to take next, it doesn’t optimise timetables. It enforces *rules*. The rules are what the brief calls “conditions”

Why this matters: if you read the brief and think “I need to *recommend* courses to students”, you’ll over-build. The brief uses the word “recommend” once, in the introduction, but the **actual requirement** is screening — yes or no.

-----

## 2. Pull the three rules out of the brief

The brief states the rules in prose, spread across two pages. Extract them into a clean list before you start coding. This is the single most important step — if your rules list is wrong, your code will be wrong.

- **R1 — Prerequisites**: for a student to enrol in a unit, every unit listed in that unit’s `pre-req` column must appear in the student’s `unit_passed` list
  - Example: `BCO6008` requires `BCO7006`. A student with `unit_passed = ["BCO7006"]` can enrol; a student with `unit_passed = []` cannot
  - Example: `BCO7007` requires *both* `BCO7000` and `BCO7006`. Partial credit doesn’t count — they need both
- **R2 — Cap**: a student may enrol in at most **2** units per request. The brief says “students are allowed to enrol in two units”
  - This is a check on the *whole request*, not per unit
  - If they request 3, your code should reject the whole request, not silently take the first two
- **R3 — Special restrictions**: `WOM1000 (Women in STEM)` is marked “Only for female elective”
  - In the sample catalogue this is the only one, but treat it as a *category* of rule. A real system would have more restrictions later (postgrad-only, location-based, etc.)
  - This is why the full showcase version uses inheritance — see Section 6

If your code enforces R1 and R2 but ignores R3, you’ve missed part of the brief and will lose marks on Criterion 6 (Algorithm and Logic).

-----

## 3. Decide your data shapes first

Before writing any logic, decide what each piece of data looks like. This is harder than it sounds and worth doing on paper.

### Why this matters

Every line of logic you write depends on these shapes. If you decide halfway through that `unit_passed` should be a `set` instead of a `list`, you’ll be rewriting checks everywhere. Decide once, up front.

### The shapes you need

**Catalogue** — the table of available units. Sketch:

```python
catalogue = {
    "BCO7006": {"name": ..., "prereqs": [], "female_only": False, ...},
    "BCO7000": {...},
    ...
}
```

- Why a **dict**? Because you’ll constantly need to look up “what are the prereqs of `BCO6008`?” — that’s a lookup by key, which is what dicts are for
- The alternative (list of dicts where each dict has a `"code"` field) works but forces you to write a `find_unit` function every time
- Each unit’s prereqs is a **list**, not a string. `"BCO7000, BCO7006".split(",")` is a code smell — store structured data structured

**Student record** — the person trying to enrol. Sketch:

```python
student = {"st_name": ..., "st_id": ..., "gender": ..., "unit_passed": [...]}
```

- `unit_passed` is a **list**. Treating it as a comma-separated string causes the classic bug: `"BCO7006" in "BCO7006,BCO7000"` is `True`, but `"BCO70" in "BCO7006,BCO7000"` is *also* `True`. Use a list, use `in` on the list
- `gender` is a string — normalise to lowercase on input so `"Female"`, `"female"`, `"FEMALE"` all work

**Enrolment record** — the output. Sketch:

```python
{"enrol_code": 1001, "unit_code": "BCO6008", "st_id": "S001", "semester": 2}
```

- The brief specifies exactly these four fields. Match them. Don’t add extras unless they earn their place
- `enrol_code` is a unique ID — auto-increment from some starting number (1001 is fine)

### Don’t pick OOP yet

You’re tempted to jump straight to `class Student`, `class Unit`. Resist. Get the dict shapes right first; you can wrap them in classes in Step 4. If your shapes are wrong, your classes will be wrong too.

-----

## 4. Build order

Build in five steps. After each step, run it and confirm it works before moving on. The point is that **at every stage, you have something running** — you’re never sitting on broken code for more than one step.

### Step 1 — Catalogue + one student

Write the catalogue dict, write one hardcoded student, print them both.

- This proves your data shapes are sensible
- You can copy-paste from the brief’s tables directly
- Done? Move on

### Step 2 — One screening function

```python
def can_enrol(student, unit_code, catalogue):
    # check R1 (prereqs)
    # check R3 (gender)
    # return (True, "ok") or (False, "reason")
```

- Why a function? Because the screening logic for *one* unit is a self-contained piece of work. Get it right in isolation
- Why return `(bool, str)` and not just `bool`? Because the user needs to know *why* a rejection happened. The brief explicitly asks for “an appropriate message and advice”
- Test it manually on three cases: an obvious pass, an obvious R1 fail, an obvious R3 fail. If all three behave correctly, you have a solid foundation
- **Don’t** check R2 here — that’s a request-level rule, not a per-unit rule

### Step 3 — The loop over requested units

```python
if len(requested) > 2:
    # reject everything, return
for unit_code in requested:
    ok, msg = can_enrol(...)
    if ok:
        # build & store the enrolment record
```

- The cap check (R2) lives *outside* the loop because it applies to the whole request
- Inside the loop, you delegate to the function you already trust
- This is the assessment’s “iterative loops” requirement — the brief asks for loops; here they are
- Output a clear message for every unit (pass or fail), then print the final enrolments list

### Step 4 — Wrap in a class

```python
class EnrolmentSystem:
    def __init__(self, catalogue): ...
    def enrol(self, student, requested, semester): ...
    def show_enrolments(self): ...
```

- The functions from Step 2 and 3 become **methods**. Almost no logic changes — they get `self.` in front of state they used to take as arguments
- Why bother? Two reasons: (1) Criterion 3 (OOP) is in the rubric, (2) the system has *state* — the list of past enrolments, the auto-incrementing `enrol_code` — and state belongs to an object
- If you’ve never written a class before: a class is just a way of bundling data (`self.enrolments`) with the functions that work on that data (`self.enrol(...)`). That’s it

### Step 5 — Interactive input

Write a small function (not a method) that uses `input()` to collect a student and a request from the keyboard, then calls `epss.enrol(...)`.

- The brief says students should be able to “input the data” — this is where you satisfy that
- Keep it simple: prompt, read, strip whitespace, split where needed, call the system
- Don’t try to handle every malformed input. Reasonable defensive coding is fine

### Why this order works

Each step produces something you can run and verify. If Step 4 breaks, you know the bug is in the class wrapper, not in the screening logic (which you already tested in Step 2). Compare this to writing the whole class at once and then discovering it doesn’t work — you have no idea where the bug is.

-----

## 5. Test scenarios

Run these six before submitting. Each one exercises a specific branch of your logic.

|#|Scenario                                           |Rule it tests|Expected                         |
|-|---------------------------------------------------|-------------|---------------------------------|
|1|Student has all prereqs, requests 2 valid units    |R1 pass      |both enrol                       |
|2|Student missing one prereq                         |R1 fail      |partial: one enrols, one rejected|
|3|Student has no prereqs, requests two advanced units|R1 fail      |both rejected                    |
|4|Female student requests `WOM1000`                  |R3 pass      |enrols                           |
|5|Male student requests `WOM1000`                    |R3 fail      |rejected with reason             |
|6|Any student requests 3 units                       |R2 fail      |whole request rejected           |

### How to think about test cases

- The point isn’t to prove your code works *sometimes* — it’s to prove it works on every branch
- Each scenario above tests a different `if` in your code. If you can’t think of a scenario that hits a particular branch, that branch might be unreachable (dead code)
- Scenarios 2 and 3 look similar but matter independently. Scenario 2 proves *partial* enrolment works (one pass, one fail in the same request) — a common bug is to reject the whole request on any failure

### Putting tests in the notebook

Don’t write a separate `test_*.py` file. Just call `epss.enrol(...)` six times in a notebook cell with comments saying what each call is testing. This *is* your evidence that the code works.

-----

## 6. Minimal vs full showcase — which to aim for

Two valid solution shapes. Choose deliberately.

|                        |Minimal                         |Full showcase                                                             |
|------------------------|--------------------------------|--------------------------------------------------------------------------|
|Classes                 |One: `EnrolmentSystem`          |Several: `Unit`, `ElectiveUnit`, `Student`, `Enrolment`, `EnrolmentSystem`|
|Catalogue               |`dict` of dicts                 |List of `Unit` / `ElectiveUnit` objects                                   |
|Gender rule             |`if` inside the screening method|`ElectiveUnit` subclass overrides `is_eligible_for`                       |
|Uses                    |classes, methods                |classes + `@dataclass` + inheritance + composition + magic methods        |
|Rubric Criterion 3 (OOP)|passes                          |maxes                                                                     |
|Code volume             |~60 lines                       |~120 lines                                                                |
|Risk if rushed          |low                             |medium — easy to over-engineer and break                                  |

### Why the full version is “better” (the actual point)

The full version isn’t better because it’s longer. It’s better because **the gender rule lives next to the unit it applies to**, not buried inside the screening engine.

In the minimal version, the screening method has to know about every special rule:

```python
if unit["female_only"] and student["gender"] != "female":
    return False, "..."
# next term, add postgrad-only:
if unit["postgrad_only"] and not student["is_postgrad"]:
    return False, "..."
# and so on, forever
```

Every new restriction means editing the screening method. The method grows. It becomes a list of unrelated `if` statements.

In the full version, each kind of unit decides *for itself* whether a student is eligible:

```python
class Unit:
    def is_eligible_for(self, student):
        # default rule: check prereqs

class ElectiveUnit(Unit):
    def is_eligible_for(self, student):
        # call super(), then add gender check
```

The screening engine just asks the unit: “is this student eligible?” — and the unit answers based on its type. Adding a new restriction means adding a new subclass, not editing existing code. This is the **open-closed principle** and it’s why OOP exists.

### Which to choose

- **Pick minimal if** you’re new to classes and want a clean pass. The minimal version still satisfies every rubric criterion — it just doesn’t max Criterion 3
- **Pick full showcase if** you’re comfortable with `@dataclass`, `super()`, and what `__str__` does. Top marks need the full design
- **Don’t mix-and-match** unless you understand both. Half-dataclass / half-dict is the worst of both worlds — you get the complexity of OOP without the cleanliness

-----

## 7. The ChatGPT half — what to actually do

The brief wants two versions of your code (yours, then improved with ChatGPT), screenshots of the dialog, and a 500-word reflection.

### Do it in this order

- **Finish your own code first.** Get all six test scenarios passing without help. Save the file as `epss_v1_no_chatgpt.py` or equivalent — this is your “before”
- **Then** copy your code into ChatGPT and ask for specific improvements. Save the result as `epss_v2_with_chatgpt.py` — this is your “after”
- Don’t skip Step 1. The reflection asks you to compare *your* experience with and without ChatGPT. If you never wrote a version without, you have nothing to reflect on

### Prompts that get useful answers

The quality of ChatGPT output is determined almost entirely by the quality of your prompt. Vague prompts get vague answers. Be specific.

|Prompt                                                                                                                      |Why it works                                                                                          |
|----------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
|“Review this code for readability and suggest improvements without changing the logic.”                                     |Constrains it to style fixes — naming, comments, structure. Easy to evaluate                          |
|“How would you refactor this to use `@dataclass` for the student and enrolment records?”                                    |Names the specific tool. You get a concrete suggestion you can compare against your dict version      |
|“What edge cases am I not handling? Suggest test cases I missed.”                                                           |Generates evidence for Criterion 6. Often surfaces things like “what if `unit_passed` is empty?”      |
|“Rewrite the gender restriction so the system supports more restrictions in future without changing the screening function.”|Asks for the inheritance refactor — a textbook OOP improvement                                        |
|“Critique my algorithm — is the order of checks efficient?”                                                                 |Generates reasoning material for the reflection. The answer may not be useful, but the conversation is|

### Prompts that *don’t* work

- ❌ “Make this better” — too vague; you get a random rewrite you can’t evaluate
- ❌ “Write me an enrolment system” — that’s the brief, not your code; markers will notice
- ❌ “Fix the bug” — without saying what the bug is, ChatGPT will invent one to fix

### How to evaluate ChatGPT’s response critically

- Did it understand your code, or did it pattern-match on the topic? Check that its suggestions actually apply to *your* code, not some generic enrolment system
- Did it introduce bugs? Run the new version against your six test scenarios. About a third of the time, ChatGPT-suggested refactors break a corner case
- Did you agree with the change, or just accept it? “It suggested X and I disagreed because Y” is *better* reflection material than “it suggested X and I used it”

### Evidence to capture

- **Screenshot each prompt + response as you go.** Don’t try to reconstruct later — you’ll forget which response came from which prompt
- A short log: *what changed, why, and whether you agreed*. Three bullet points per change is enough
- The “before” and “after” code files, both runnable

### What to put in the 500-word reflection

The brief asks three specific questions. Answer each in ~150 words; the remaining ~50 are for an intro/conclusion.

1. **“Do you believe using ChatGPT will facilitate your learning of programming? In what manner?”** — Be concrete. “It explained why `@dataclass` reduces boilerplate code I had repeated three times” is far better than “it was helpful”. Use an example from *your* dialog
1. **“To what extent can you rely on the support of ChatGPT?”** — Markers want critical thinking, not endorsement. Give an example where ChatGPT was wrong, made up a function that doesn’t exist, or where you had to push back. If you didn’t catch it doing anything wrong, you weren’t checking
1. **“What part of your development process would benefit from ChatGPT the most — algorithm design, coding, or testing?”** — Pick one. Defend it with evidence from *this* task. There’s no right answer; markers reward defended reasoning

### Honesty note

Don’t paste ChatGPT’s code, rename three variables, and submit. The reflection asks what *you* did and *what you learned* — answers that don’t square with your code give that away immediately. Use ChatGPT to learn; submit work you can explain in a viva.

-----

## 8. Rubric self-check

Before submitting, open your notebook and point to where each criterion is satisfied:

- ☐ **Conditional statements** — `if` / `else` in the screening method (and in `enrol` for the cap check)
- ☐ **Iterative loops** — `for` or `while` over `requested_units`
- ☐ **OOP** — at least one class with state (attributes) and methods. More for the full showcase
- ☐ **Functions** — at minimum: screening, enrolment, display, interactive input. Methods count
- ☐ **Dataset management** — dicts and/or lists for the catalogue, students, enrolments
- ☐ **Algorithm and logic** — flowchart at the top of the notebook, and the code visibly follows it

For Part 2 (reflection): clarity, depth of analysis, comparison and contrast, application of learning, grammar, word count (500 words ±10%).

-----

## 9. Common traps

Each of these is a real failure mode I’d expect to see in submissions. The *why* explains how to avoid it.

- **Hardcoding the gender rule deep inside a function.** Why it happens: it’s the fastest way to make Scenario 4/5 pass. Why it bites: makes adding the next restriction painful, and it’s the exact thing OOP is supposed to fix. Even in the minimal version, give it its own block so the structure is visible
- **Forgetting the cap check.** Why it happens: you write the per-unit loop, the cap doesn’t fit inside it, you forget. Why it bites: silent failure — your code accepts 3-unit requests with no warning. Always check the cap *before* the loop starts
- **Returning `None` instead of a message.** Why it happens: you start with `return False`, never come back to add the reason. Why it bites: the brief explicitly requires “an appropriate message and advice”. `(False, "reason")` from the start avoids it
- **Capitalisation drift.** Why it happens: catalogue has `"BCO7006"`, user types `"bco7006"`, your `in` check fails silently. Why it bites: looks like a logic bug but is a data bug. Normalise input with `.upper()` immediately on read
- **No test output visible.** Why it happens: you write the class, define it, but never call it. Why it bites: markers can’t see your code work. The notebook must show outputs from calling the system
- **Treating `unit_passed` as a string.** Why it happens: you copy-pasted from the brief’s table where it looked like text. Why it bites: `"BCO7006" in "BCO7006,BCO7000"` is `True`, but `"BCO70" in "BCO7006,BCO7000"` is *also* `True`. Substring matching is a different operation from list membership. Always use lists
- **OOP for its own sake.** Why it happens: you read “rubric wants OOP” and create five classes wrapping nothing. Why it bites: an empty `class Catalogue: pass` that wraps a dict is worse than just using the dict. Classes earn their place when they have *both* state and behaviour
- **Submitting without the flowchart.** Why it happens: it’s the only deliverable that isn’t code, so it gets forgotten. Why it bites: Criterion 6 explicitly asks for it. Put it at the top of the notebook before you start coding — it doubles as your design

-----

### Final pre-submission check

Run every cell of your notebook top to bottom in a fresh kernel. If anything errors, fix it before submitting. “It works on my machine after I run cells in a specific order” is not a passing grade.

Make sure both parts are in the submission: the code (with evidence — outputs visible) and the 500-word reflection. They’re worth marks separately.
