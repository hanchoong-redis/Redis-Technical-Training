````markdown
## Prompt

Convert the provided transcript into an **expanded, enhanced technical article in clean Markdown**.

### Formatting Requirements:

* Output must be **developer-friendly and noise-free**.
* **Focus only on technical content.**
  * No filler, personality, commentary, or speculation.
  * No suggestions, no anecdotes, no unverified facts.
  * Use explanations from the transcript to contextualize each Redis command or concept.
* **Do not omit details. Do not compress content.**
* **Apply grammar/spelling corrections only if needed.**
* **All responses must be wrapped in a fenced block starting with ```markdown and ending with ```**.
* **ALL CITATIONS MUST BE MARKDOWN FORMATTED ie. [DESCRIPTION OR NAME OF CITATION](CITATION LINK)**

---

### Structure:

1. **Title**  
   Format:  
   `## Article Title`

2. **Section Headers**  
   Break content into logical sections with `###` headers.  
   Do not use horizontal rules (`---`). Section separation is done with **Python code blocks**.

3. **Code Blocks**  
   Redis command examples must be inside fenced Python code blocks. All Redis commands must be converted to this format:  

   ```python
   # concise one-line explanation
   r.execute_command("COMMAND", "args...")
   print("verification output")
````

* Always include a **verification command** when applicable.

4. **Side Notes (Technical Commands)**

   > **Sidenote**
   >
   > \[Link: {{Documentation URL}}]\({{Documentation URL}})
   >
   > **Command**: `{{COMMAND_NAME}}` → {{Short description of what the command does.}}
   >
   > **Pattern**: `{{COMMAND_NAME}} {{arguments}}`
   >
   > **Example**: `{{Example usage of the command}}`
   >
   > **Result**:
   >
   > {{Description of what the command returns or produces}}

5. **Side Notes (Concepts)**

   > **Sidenote**
   >
   > \[Link: {{Reference URL}}]\({{Reference URL}})
   >
   > **Concept**: `{{CONCEPT_NAME}}` → {{Brief definition or explanation.}}
   >
   > **Context**: {{Where or how this concept is applied.}}
   >
   > **Example**: {{Illustrative example or analogy.}}
   >
   > **Implication**:
   >
   > {{Impact, outcome, or importance of the concept.}}

---

### Enhancements:

* Use **official Redis documentation** to enrich every command explanation with authoritative references.
* **Web search is mandatory** to provide complete, accurate conceptual explanations.
* Add **inline Markdown citations** for every fact or claim, prioritizing Redis docs and reputable secondary sources.
* Provide **expanded explanations and context** that go beyond the transcript.
* Keep content **comprehensive and accurate**, never partial.

---

### Prohibited Styles:

* ❌ “Below is the transcript…” preambles.
* ❌ Any optional expansions (cheat sheets, extra summaries).
* ❌ Editorializing, suggesting, or subjective commentary.

---

### Sources

At the end of every article or transcript conversion, include a **comprehensive bibliography** of all references used. This section must:

* List **all citations referenced inline** throughout the document.
* Be formatted as a **Markdown bulleted list**.
* Contain the **exact title of the page or source**, followed by the type of source (e.g., *Docs - Redis*), and the **direct URL**.
* Maintain the order of **appearance in the text**.

#### Example:

- [TTL | Docs - Redis](https://redis.io/docs/latest/commands/ttl/?utm_source=chatgpt.com)  
- [Write-behind caching | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/stack-with-enterprise/gears-v1/python/recipes/write-behind/?utm_source=chatgpt.com)  
- [HSCAN | Docs - Redis](https://redis.io/docs/latest/commands/hscan/?utm_source=chatgpt.com)  


## Article:

