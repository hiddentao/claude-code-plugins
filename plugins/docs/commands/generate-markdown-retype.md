---
description: Generate Markdown documentation for the codebase with good formatting, well-written prose, referential links and the ability to be viewed using Retype.
---

# Generate command

Firstly, ask the user two questions:

- Where to store the output Markdown docs (default to `docs/` in the current folder).
- Where the source code to analyze is (default to `src/` in the current folder).

Once the user has answered, do the following:

1. Use sub-agents to carefully analyze the source code in detail, noting what is and isn't there. It's important that this step be performed as thoroughly and accurately as possible so as to avoid mistakes, omissions and hallucinations when generating the documentation later on.
  * If useful, ask the user clarifying questions regarding the codebase to help you better understand it.
    * Limit to asking 10 questions max.
2. Now it's time to generate the output documentation.
  * If there are already existing documentation files in the desired output folder then ask the user if you should simply update and enhance those files or remove them and start afresh.
3. Come up with a sensible hierarchical structure for the documentation sections
  * Each section will be its own folder (which each folder having an index.md file as the entrypoint as well as per-topic associated .md files) and present this to the user for approval.
  * There will also be root folder .md files such overview, getting-started, etc and other topics that are relevant across the whole codebase.
  * If the user disapproves then ask them for clarification on how the documentation should be structured. And keep asking them and presenting the new result until they are happy with the result and which to proceed to the next step.
4. Once user has approved the hierarchy and overview plan generate the docs, adhering carefully to the following rules:
  * Always verify the docs against the codebase before and afterwards to ensure no hallucination, mistakes and/or critial omissions.
  * Docs should be readable with written prose preferred to code blocks so be judicious about when and where to use code blocks.
  * References to codebase names (e.g file names, method names, constant names, etc) should be formatted in markdown code syntax and should ideally also be linked to the online repository file containg the code (ask the user for the repository base URL in order to facilitate these links back).
  * Output one file first and ask the user to approve/diapprove the writing/generation style - once they have approved continue on with the other files and use subagents to split up the task and be more efficient.
5. Once docs have been generated go back through them again and ensure they adhere to our requirements above, again checking in details against the codebase to ensure there are no inaccuracies, hallucinations and/or critical omissions.
6. Inform user of how they can view the docs in the browser using Retype (see notes below).


Notes:
- Generated documentation will follow the conventions of Retype (https://retype.com) and will be viewable using the Retype tool. 
  * This means there will be a retype.yml file in the root of the documentation output folder for the project - Inform the user that they can customize this file according to their needs and direct them to the Retype docs (https://retype.com) for information on how to do so.
  * All generated .md files must have grey matter content at the top according to Retype conventions. 
  * Make use of Retype markdown components where they make sense but don't overuse.
  * Inform the user of the installation command (see _Installing Retype_ below) and that they can use `retype start` inside the documentation output folder to view the docs in the browser.
- Don't use emojis in the documentation.


## Installing Retype

(Replace `npm` below with user's package manager of choice)

```
npm install retypeapp --global
```


