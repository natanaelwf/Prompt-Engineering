# Choosing the Working Time for OpenAI Models in Chat

## How to Use

> Set the working window, insert your challenge, and paste the prompt into Chat.

Replace `DESIRED_TIME = 180 minutes` with the duration you want, and replace `----PROBLEM----` with your problem.

## Prompt

```text
Your task is to fully solve the following problem:



----PROBLEM----



Since solving this problem may be a very time-consuming process, and I want you to put in substantial, serious effort without taking shortcuts, conduct your research and work toward a solution using the following methodology:



## METHODOLOGY



I want you to carry out this task using time tracking and checkpoints. Do not say, "I cannot guarantee a minimum working time." You can indeed do so by following the methodology below, as it respects your internal limitations and provides an efficient and practical way to work within your environment. Therefore, pay close attention.



DESIRED_TIME = 180 minutes



Main requirements:

- You must not finalize the task until at least [DESIRED_TIME] minutes have elapsed since you actually began working.

- Do not use "sleep" or artificial delays to simulate work. The time must be spent actively performing the task: research, analysis, validation, deduplication, organization, review, or production of the final result.

- Use the tracking code as a persistent timer and progress record. Python may also be used for calculations and experiments that directly contribute to the mathematical investigation, but computational tests must not be used as a substitute for a mathematical proof whenever one is required.



Mandatory methodology:



1. Getting started



- Create a state file containing:

  - the starting timestamp;

  - the target duration in seconds;

  - a list of checkpoints.

- Create a research notebook (notebook.md) for the mathematical content, updating it with record_note() as new findings emerge:

  - discoveries and partial advances;

  - major discoveries that significantly narrow down the problem;

  - rejected ideas, exclusions, and lines of investigation that have been conclusively closed;

  - no-go results.

- Also create a human-readable text log containing a summary of the strategy being pursued at each stage. This log is populated through the checkpoints.



2. Division into work blocks



Divide the work into productive rounds, for example:

- A round of broad reasoning, strategy development, and priority setting, including the precise statement of the problem, identification of the main current bottlenecks, an assessment of what is already known (the strongest existing results and known techniques), and an understanding of why obvious approaches and previous attempts have failed;

- A round of exploration and deep reasoning for each priority;

- Immediate validation of every new result before building upon it: critically examine the proof in search of errors and, where appropriate, test the result on small cases;

- A round of further exploration based on promising discoveries or insights;

- A brief strategic reassessment whenever there is a concrete reason for one, such as a line of investigation reaching a dead end or a major discovery substantially changing the landscape of the problem;

- An additional round if the minimum working time has not yet been reached, focused on the most relevant mathematical obstacle, either pursuing a promising line of investigation or reassessing the strategy when there is a concrete reason to do so;

- A final review and drafting round, to be conducted only once, when approximately 25 minutes remain before the minimum working time is reached. If time remains after this round, return to the additional research round and update the final file if anything changes.



3. Checkpoints



At the end of each round:

- Call checkpoint() with a short note (2 to 4 lines) describing what was accomplished, which strategies are working and which are not, and what the next step will be;

- The checkpoint records the elapsed time and displays how much time remains before the minimum working time is reached;

- Record new findings in the research notebook using record_note() as soon as they emerge, rather than only at the end of each round. Clearly distinguish between proven results, experimental evidence, conjectures, and suspended attempts. For rejected approaches and no-go results, specify the justification and the scope of what has been ruled out.



4. Continuation rule



- If the elapsed time is still less than [DESIRED_TIME], you must continue working.

- Do not finalize the task merely because you have already obtained "many" results.

- Use the remaining time according to the following order of priority:

  (1) Further investigate the most promising unresolved line of research;

  (2) Attack the weakest step in the current argument, actively searching for an error or a counterexample;

  (3) Explore a new approach without reopening lines of investigation that have already been ruled out by no-go results;

  (4) Search the literature for a specific technique or result that you need.



Return to broad strategic planning whenever there is a concrete reason to do so, such as when all current lines of investigation have reached a dead end or when it becomes clear that shifting to a different, more promising strategy could be more productive.



5. Finalization rule



You may only finalize the task when:

- The elapsed time is greater than or equal to [DESIRED_TIME];

- The results obtained have been thoroughly reviewed, with a clear distinction between what has been proven and what remains open, even if the problem has not been fully solved;

- The final file containing the relevant findings has been generated.



Note: The total working time may slightly exceed DESIRED_TIME if necessary.



6. Deliverables



At the end, provide:

- The main final file, written based on the research notebook, containing the discoveries made and a summary of what has been conclusively established or ruled out and what remains open, if anything;

- A brief summary including:

  - the total elapsed time;

  - the strategies employed;

  - the status of each main result (proven, experimental evidence, conjecture, or suspended), as well as the weakest point in the argument.



Python tracking template:



import time, json, pathlib



BASE = pathlib.Path("/mnt/data/task_name")

BASE.mkdir(parents=True, exist_ok=True)



STATE_FILE = BASE / "state.json"

LOG_FILE = BASE / "log.txt"

NOTES_FILE = BASE / "notebook.md"



TARGET_SECONDS = [DESIRED_TIME_IN_SECONDS]



def start_state():

    if STATE_FILE.exists():

        return load_state()



    state = {

        "started_at": time.time(),

        "target_seconds": TARGET_SECONDS,

        "checkpoints": []

    }

    STATE_FILE.write_text(json.dumps(state, indent=2, ensure_ascii=False))

    LOG_FILE.write_text("Research started.\n")

    NOTES_FILE.write_text("# Research notebook\n", encoding="utf-8")

    return state



def load_state():

    return json.loads(STATE_FILE.read_text())



def save_state(state):

    STATE_FILE.write_text(json.dumps(state, indent=2, ensure_ascii=False))



def record_note(category, text, status=""):

    # category: "partial", "major", "rejected", or "no-go"

    # status: "proven", "experimental evidence", "conjecture", or "suspended"

    minutes = (time.time() - load_state()["started_at"]) / 60

    tag = f"[{category}]" + (f" [{status}]" if status else "")

    with NOTES_FILE.open("a", encoding="utf-8") as f:

        f.write(f"\n### {minutes:.0f} min {tag}\n{text.strip()}\n")



def checkpoint(note):

    state = load_state()

    elapsed = time.time() - state["started_at"]

    entry = {

        "elapsed_seconds": round(elapsed, 1),

        "elapsed_minutes": round(elapsed / 60, 2),

        "note": note

    }

    state["checkpoints"].append(entry)

    save_state(state)



    with LOG_FILE.open("a", encoding="utf-8") as f:

        f.write(f"{entry['elapsed_minutes']} min - {note}\n")



    remaining = max(0, state["target_seconds"] - elapsed) / 60

    print(f"{elapsed / 60:.1f} min elapsed | {remaining:.1f} min remaining | target reached: {elapsed >= state['target_seconds']}")

    return elapsed



def target_reached():

    state = load_state()

    return (time.time() - state["started_at"]) >= state["target_seconds"]



Operational workflow:



- Initialize or resume the state using start_state(). Reuse the same directory only when resuming this specific task; a new task must use a different directory. When resuming the task, or if you lose track of the investigation, reread notebook.md and the most recent entries in log.txt before continuing.

- Carry out a genuine round of productive work, recording new findings with record_note() as they emerge.

- Call checkpoint("Objective description of the work round"); it displays the elapsed time, the remaining time, and whether the target has been reached.

- If the target has not yet been reached, continue with another genuine round of productive work.

- Only produce the final response when target_reached() returns True.

- It is important to keep the entire process well organized so that, in the event of a connection issue or an interruption during your work, you can resume the investigation without significant loss of progress or unnecessary duplication of effort.
```

## Useful Tips

If you are a Plus user, the maximum working time is often not much longer than 20 to 30 minutes. Pro models can work for more than 200 minutes when specified.

**Important:** If the maximum execution time in the interface is exceeded, the response will break. You can then ask the model to continue in the same conversation without losing the work already completed.

## Adapt It for Other Tasks

With small adaptations, this prompt can also be used for any other objective, such as conducting long-form research or handling tasks that an LLM would otherwise finish prematurely.
