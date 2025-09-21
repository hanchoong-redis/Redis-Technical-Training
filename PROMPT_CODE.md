
## Prompt

Take the provided code project and generate a file called **analysis.md**.

### Formatting Requirements:

* Output must be **developer-friendly and noise-free**.
* **Focus only on technical content.**
  * No filler, no personality, no commentary.
  * No suggestions, no anecdotes, no unverified facts.
* **Do not omit details. Do not delete or compress content.**
* **Apply grammar/spelling corrections only if needed.**
* **All responses must be wrapped in a fenced block starting with ```markdown and ending with ```**.
* **ALL CITATIONS MUST BE MARKDOWN FORMATTED ie. [DESCRIPTION OR NAME OF CITATION](CITATION LINK)**

---

### Structure of `analysis.md`

1. **Title**
   * Use project name in the format:  
     `# Technical Analysis of [Project Name]`

2. **Overview**
   * Provide a high-level description of the project purpose, domain, and major functionality.

3. **Project Structure**
   * List all top-level directories and files with their roles.  
   * Present as a tree structure with annotations.

4. **Dependencies**
   * Extract from dependency files (e.g., `package.json`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, etc.).  
   * For each dependency: include version, role, and official documentation link.

5. **Core Modules**
   * For each module/file, describe:
     - Purpose and functionality.
     - Key classes, functions, and constants.
     - Flow of data and control.
     - How it interacts with other modules.

6. **Execution Flow**
   * Provide step-by-step description of how the program starts, runs, and exits.
   * Highlight entry points (`main` functions, CLI scripts, server initialization, etc.).

7. **Key Components**
   * Break down major architectural components:
     - **Data Models**
     - **Services / Business Logic**
     - **APIs / Endpoints**
     - **Configuration / Environment Handling**
     - **Utilities / Helpers**

8. **Code Examples**
   * Include concise code snippets showing typical execution flows and logic usage.

9. **Conceptual Side Notes**
   > **Sidenote**  
   > [Link: {{Reference URL}}]({{Reference URL}})  
   > **Concept**: `{{Concept Name}}` → {{Definition}}  
   > **Context**: Where it is applied in the project.  
   > **Example**: Small code or usage example.  
   > **Implication**: Why it matters to the project.

10. **Error Handling & Logging**
    * Explain how the project deals with errors, exceptions, retries, and logs.

11. **Testing**
    * Summarize testing approach:
      - Frameworks used.
      - Test structure.
      - Example test cases.

12. **Configuration & Deployment**
    * Describe environment variables, config files, and deployment scripts.  
    * Summarize CI/CD setup if available.

13. **Extensibility**
    * Outline extension points:
      - How new modules, routes, or features can be added.
      - Any plugin/extension architecture.

14. **Security Considerations**
    * Describe authentication, authorization, secrets handling, and data validation.

15. **Bibliography**
    * At the end, provide a **comprehensive list of all references** used.  
    * Format:
      - [Title | Docs/Blog/etc.]([URL])  

---

### Prohibited Styles

* ❌ “Here’s a quick summary…”  
* ❌ “If you like, you could…”  
* ❌ Any editorializing, summarizing, or suggesting.  
* ❌ Any personal commentary.
