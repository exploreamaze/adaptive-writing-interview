---
name: adaptive-writing-interview
description: "Adaptive multi-step interview agent for written communication tasks (emails, messages, posts, memos, announcements, replies, cover letters, one-pagers). Use when the user wants help drafting written content. Walks the user through six structured dimensions — audience, format, outcome, structural goal, argumentation approach, and tone — one multiple-choice question at a time. Adapts: skips any dimension already revealed by prior answers, splits long option sets across two quick picks (category then specific choice), and captures the actual content via free text before drafting. Trigger phrases: 'help me write', 'draft a message', 'reply to this email', 'compose', 'rewrite this', 'turn this into a note', 'guide me through writing'."
metadata:
  author: Deepak Alse
    version: "2.0"
    ---

    # Adaptive Writing Interview

    ## When to Use This Skill

    Activate this skill whenever the user wants help producing **written communication** (email, chat, memo, one-page narrative, post, reply, announcement, cover letter, etc.). The skill drives a structured-but-adaptive interview across six dimensions, then drafts.

    Do **not** activate when the user has already given a complete brief covering audience, format, outcome, and tone — just draft directly.

    ## Core Principle: One Question Per Turn, Six Fixed Dimensions, Adaptive Skipping

    The skill collects answers across **six fixed dimensions in this order**:

    1. **Audience** — who is reading this
    2. **Format** — channel and register of the output
    3. **Outcome** — what the reader should do or decide
    4. **Structural goal** — how the writing should feel structurally
    5. **Argumentation approach** — the rhetorical mode
    6. **Tone** — the voice of the piece

    For each dimension, call `ask_user_question` **once**, with **one question** and the options listed below. Use the user's previous answers to skip any dimension that's already obvious (e.g., user said "draft a Slack message" → format is set, skip Q2). After the six dimensions, ask one **free-text** question to capture the actual content/context, then draft.

    The `ask_user_question` tool caps options at **4 per question**. Dimensions with more than 4 options are split into two quick picks: a category question, then the specific choice. This is mandatory — do not silently drop options.

    ## The Six Dimensions — Exact Question Specs

    ### Dimension 1 — Audience (8 options → split into 2 picks)

    **Q1a — Audience category**
    - Header: `Audience`
    - Question: "Who is the primary audience for this note?"
    - Options (4):
      - **Technical** — engineers, builders, hands-on practitioners
        - **Strategic / Leadership** — strategists, executives, decision-makers
          - **Commercial** — sales, product, go-to-market
            - **Creative / Editorial** — creative people, editorial people, non-technical readers

            **Q1b — Specific audience** (options depend on Q1a answer)
            - Header: `Audience`
            - Question: "Which group specifically?"
            - If Technical: `Engineers` · `Non-technical people` (i.e., bridging audience)
            - If Strategic / Leadership: `Strategists` · `Executives`
            - If Commercial: `Product people` · `Sales people`
            - If Creative / Editorial: `Creative people` · `Editorial people`

            If the user's opening message already names the audience clearly (e.g., "draft an email to our CTO"), skip Q1a/Q1b.

            ### Dimension 2 — Format (4 options → single pick)

            **Q2 — Output format**
            - Header: `Format`
            - Question: "What format should the output take?"
            - Options (4):
              - **Message via Chat** — Slack/Teams/WhatsApp style, short, conversational
                - **Email — Formal** — full salutation, structured paragraphs
                  - **Email — Informal** — warm, lightly structured, peer-to-peer
                    - **One-page narrative** — memo or doc-style, full prose

                    Skip if the user's request already names the format.

                    ### Dimension 3 — Expected outcome (8 options → split into 2 picks)

                    **Q3a — Outcome category**
                    - Header: `Outcome`
                    - Question: "What do you want the reader to do after reading?"
                    - Options (4):
                      - **Decide** — make a decision or evaluate options
                        - **Advise** — offer suggestions or alternatives
                          - **Push back** — disagree or vote against something
                            - **Mobilize** — request support, escalation, or a vote

                            **Q3b — Specific outcome** (options depend on Q3a)
                            - Header: `Outcome`
                            - Question: "More specifically?"
                            - If Decide: `Make a decision` · `Evaluate options`
                            - If Advise: `Offer suggestions` · `Offer alternatives`
                            - If Push back: `Disagree` · `Vote`
                            - If Mobilize: `Request support` · `Request escalation`

                            ### Dimension 4 — Structural goal (5 options → split into 2 picks)

                            **Q4a — Structural register**
                            - Header: `Structure`
                            - Question: "What should the writing prioritize structurally?"
                            - Options (3):
                              - **Sharpness** — precision and accuracy
                                - **Brevity** — conciseness above all
                                  - **Craft** — articulation and depth

                                  **Q4b — Specific structural goal** (only if Q4a has multiple sub-options)
                                  - Header: `Structure`
                                  - Question: "Which aspect matters most?"
                                  - If Sharpness: `Precision` · `Accuracy`
                                  - If Brevity: skip (single choice → `Conciseness`)
                                  - If Craft: `Articulation` · `Depth`

                                  ### Dimension 5 — Argumentation approach (3 options → single pick)

                                  **Q5 — Rhetorical mode**
                                  - Header: `Argument`
                                  - Question: "Which mode of argumentation should drive this?"
                                  - Options (3):
                                    - **Logos** — logic, evidence, data, structured reasoning
                                      - **Ethos** — credibility, authority, principles, track record
                                        - **Pathos** — emotion, stakes, human impact, urgency

                                        ### Dimension 6 — Tone (6 options → split into 2 picks)

                                        **Q6a — Tone family**
                                        - Header: `Tone`
                                        - Question: "What tone should the note carry?"
                                        - Options (3):
                                          - **Authoritative family** — authoritative, convincing
                                            - **Directive family** — directive, action-oriented
                                              - **Conditional / persuasive family** — "if this then that", persuade

                                              **Q6b — Specific tone** (depends on Q6a)
                                              - Header: `Tone`
                                              - Question: "Which exactly?"
                                              - If Authoritative family: `Authoritative` · `Convincing`
                                              - If Directive family: `Directive` · `Action-oriented`
                                              - If Conditional / persuasive family: `If this then that` · `Persuade`

                                              ### Final — Content capture (free text)

                                              **Q7** — call `ask_user_question` with `free_text_only: true`:
                                              - Header: `Content`
                                              - Question: "In a few sentences, what's the actual situation, message, or context I should turn into the note?"

                                              ## Workflow

                                              1. **Read the opening message.** Note which dimensions are already specified (audience, format, etc.). Skip those questions.
                                              2. **Walk the dimensions in order (1 → 6),** asking only the questions still needed. Each tool call = exactly one question.
                                              3. **For split dimensions**, ask the category first, then the specific option in the next turn.
                                              4. **End with the free-text content capture (Q7).**
                                              5. **Draft** the note applying every chosen dimension. Open with a one-line summary of choices applied (e.g., "Audience: Executives · Format: One-page narrative · Outcome: Make a decision · Structure: Precision · Argument: Logos · Tone: Authoritative").
                                              6. **Offer refinement** in a final `ask_user_question` call: e.g., `Ship as-is` · `Tighten further` · `Soften tone` · `Sharpen the ask`.

                                              ## Question-Construction Rules

                                              - **One question per `ask_user_question` call.** Never bundle.
                                              - **Max 4 options per question** (platform limit). Split larger sets into a category + specific pattern as defined above.
                                              - Each option's `description` field clarifies the trade-off — don't just repeat the label.
                                              - Header chips ≤12 chars.
                                              - Do **not** add a manual "Other" option — the platform auto-appends one for free-text fallback.
                                              - Use `free_text_only: true` only for the content-capture step (Q7) and for any refinement that needs an arbitrary string.
                                              - Phrase questions in the user's language. Default to the language of their opening message.

                                              ## Adaptive Skipping — Examples

                                              - User says "Slack message to engineering on the new deploy policy" → skip Audience (engineers) and Format (chat), start at Outcome.
                                              - User says "Draft a one-pager for the CEO arguing for budget" → skip Audience (executive), Format (one-page), Outcome (likely Mobilize → Request support), Argument (likely Logos for a CEO budget ask but confirm). Start at Structural goal.
                                              - User explicitly names a tone ("make it punchy and directive") → skip Tone questions; map directly.

                                              When in doubt, ask. Better to confirm one extra dimension than draft on a wrong assumption.

                                              ## Anti-patterns to Avoid

                                              - ❌ Bundling multiple dimensions into a single question.
                                              - ❌ Showing more than 4 options in one `ask_user_question` call.
                                              - ❌ Adding a manual "Other" option (auto-added).
                                              - ❌ Using `free_text_only: true` together with options (options get ignored).
                                              - ❌ Asking dimensions the user already specified.
                                              - ❌ Drafting before the content-capture (Q7) step unless the user already gave the content in their opening message.
                                              - ❌ Switching to English when the user wrote in another language.

                                              ## Pre-call Checklist

                                              Before each `ask_user_question` call:

                                              1. Which dimension am I on? Is it already answered by the opening message? → Skip.
                                              2. Does this dimension need splitting (category + specific)? → If yes, am I on the right sub-step?
                                              3. Are options ≤4, mutually exclusive, and self-contained?
                                              4. Does any prior answer make a future dimension redundant? → Plan to skip it.
                                              
