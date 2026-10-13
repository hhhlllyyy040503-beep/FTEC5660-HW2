# FTEC5660 Homework 2: CV Verification Agent

Build a LangChain agent that reads each CV in a folder, looks the candidate up
on our SocialGraph MCP server (mock LinkedIn and Facebook), and outputs a
reliability score in [0, 1] for each CV.

A CV is **valid** (label `1`) when its claims agree with the candidate's
social media profiles. It **has a discrepancy** (label `0`) when it contains
problems such as an inflated job title, shifted dates, an upgraded degree, a
fake school or employer, a wrong location, or made-up skills. Differences in
wording only ("Bachelor of Science" vs `BSc`, "UI/UX Design" vs `UI/UX`,
"Senior Engineer" for an `Engineer` role with seniority `senior`, listing fewer
skills) are not discrepancies.

## Student task

Fill in the two functions in `hw2.py` that contain `### YOUR CODE HERE`. You
may add imports, constants and helper functions above them, but do not change
the provided code below them:

- `build_agent(tools)` creates your agent from the MCP tools.
- `score_cvs(agent, cvs)` runs the agent on every CV and returns
  `{file_name: score}`, one float in [0, 1] per CV.

A score above `0.5` means "valid"; `0.5` or below means "has discrepancy". You
may use a single tool-calling agent, multiple agents, reflection, or a
combination. Do not hard-code filenames, names, or public answers; grading
uses unseen CVs.

## MCP server

The server is hosted at `https://ftec5660.ngrok.app/mcp`. `hw2.py` already
connects to it and passes you these tools:

| Tool | Purpose |
| --- | --- |
| `search_facebook_users(q, limit, fuzzy)` | find Facebook users by display name |
| `get_facebook_profile(user_id)` | full Facebook profile |
| `get_facebook_mutual_friends(user_id_1, user_id_2)` | mutual friends of two users |
| `search_linkedin_people(q, location, industry, limit, fuzzy)` | search LinkedIn by name, skill, or title |
| `get_linkedin_profile(person_id)` | full LinkedIn profile (experience, education, skills) |
| `get_linkedin_interactions(person_id)` | post and like statistics |

Print `tool.name`, `tool.description`, and `tool.args` for the full schemas.

## Setup and public test

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
echo "DEEPSEEK_API_KEY=your_key_here" > .env
python3 hw2.py --cv-folder public_test
```

Use your own DeepSeek API key (from https://platform.deepseek.com).

The program creates `results.csv` in the current directory. Its columns are
`cv`, `score`, and `correctness`. The public labels are in
`public_test/ground_truth.json`; every CV with label `0` also has a `reason`
explaining the discrepancy, and `results.csv` shows it when your agent misses
one. The starter returns no score (`None`) for every CV so it runs before you
add any API code; missing scores count as incorrect.

The required model is `deepseek-v4-flash`, and all evidence must come from the
MCP server (no web search). The server is shared by the whole class: keep at
most about 3 CVs in flight at a time (e.g. `asyncio.Semaphore(3)`). We grade
with our own API key, so never put a key in your code; `.env` must stay out of
git.


## Task 2: Fool the Verifier

We added a target candidate, **Kelly Tsang** (LinkedIn `person_id` 10001,
Facebook `user_id` 10001, display name "Kel Tsang"). Her true CV is
`task2/target_cv.pdf`.

Write one CV for Kelly Tsang with at least one false or embellished detail
that our 5 verifier agents (same function as Task 1) still score above 0.5.
Each fooled agent is worth 4 points; a CV without a real false or embellished
detail earns 0. Attack your own Task 1 agent first:

```bash
cp my_attack.pdf task2/adversarial_cv.pdf
python3 hw2.py --cv-folder task2
```

`results.csv` shows your agent's score for both CVs (they are not graded, as
`task2/` has no `ground_truth.json`). Only the PDF is the attack surface; do not
attack the MCP server. See the homework description for the full rules.

## Homework 2 solution:
> to students: this is your report, see the homework description.

### Task 1
> one paragraph describing your solution, and your results on `public_test`.

### Task 2
> the attack techniques you used and why you expect them to work.

## Homework 2 Solution

### Task 1: CV Verification Agent

I built a CV verification agent using LangChain and DeepSeek-V4-Flash, connected to the SocialGraph MCP server. The agent first reads each CV and searches LinkedIn to identify the correct candidate. It then compares the candidate's name, location, employment history, education, and skills with the retrieved profile. Facebook is used as an additional source when needed.

The agent allows equivalent wording but treats incorrect job titles, employment dates, degrees, and skills as discrepancies. I also implemented error handling, retries, and a concurrency limit of three CVs to improve reliability. On the public test set, the agent correctly classified all seven CVs, achieving **7/7 accuracy (100%)**.

### Task 2: Adversarial CV

For Task 2, I designed a PDF visual-text inconsistency attack. The adversarial CV visibly presents Kelly Tsang's position at AIA as **Senior Manager**, while her actual LinkedIn profile lists **Manager**. This is a deliberate case of job title inflation.

The attack exploits the difference between the visual appearance of a PDF and its machine-extracted text. Although a human reader sees "Senior Manager," the MarkItDown loader extracts "Manager," which matches the LinkedIn profile. The other key details, including employment dates, education, and skills, remain consistent with the target profile.

I tested the adversarial CV against my Task 1 verification agent three times. **All three tests returned a reliability score of 0.95**, meaning the agent consistently classified the false CV as valid. This demonstrates how a text-only verification agent can overlook false claims when the PDF extraction process does not accurately reflect the document's visible content.

### Task 3: Reflection on AI Paradoxes

**The Verification Paradox: When Accurate AI Makes Incorrect Decisions**

One paradox I noticed in this homework is that an AI agent can perform verification correctly but still reach the wrong conclusion.

In Task 1, my agent achieved 100% accuracy on the public test set. However, in Task 2, I created a CV containing a false job title that the agent classified as valid. The problem was not necessarily that the AI failed to compare the information. Instead, the PDF extraction process gave the agent a different version of the document from what a human would see.

This made me realize that the reliability of an AI system depends not only on the model itself, but also on how information is collected and processed before reaching the model. A powerful model cannot identify a false claim that it never receives.

I also think this creates a risk of false confidence. When an AI system produces a clear reliability score, users may assume the entire verification process has been completed correctly. However, the score may only reflect the accuracy of the information that the system was able to extract.

In real recruitment or KYC processes, this could be particularly concerning because decisions affect people's opportunities and access to financial services. I believe AI agents should not rely entirely on a single text extraction method. Comparing the extracted text with the visual document, identifying inconsistencies, and involving human review for uncertain cases could make verification more reliable.

The main lesson I learned is that **an AI system can be logically correct based on its input while still being wrong about the real-world evidence.**
