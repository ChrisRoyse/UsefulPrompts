LLMs predict the next token based on the input sequence and their training data. Prompt
engineering is about crafting input sequences to guide the LLM toward desired predictions,
often through deliberate iterative "thought" processes.
A powerful strategy for complex tasks is to guide the LLM through an iterative refinement loop,
now deepened by integrating self-consistency. Each loop involves:
This recursive process produces more robust answers—especially for ambiguous or complex
queries.
Recursive Learning Prompt Engineering Best
Practices (With Integrated Iterative SelfConsistency)
Foundational Concept: LLMs as Prediction Engines
Maximizing Accuracy: Iterative Refinement Loop with Self-Consistency
Generate an initial answer or plan (using creative sampling for diversity).
Critically evaluate or reflect on that output.
Revise or improve based on self-critique and/or new information.
Self-consistency: Prompt the LLM to generate multiple diverse (high-temperature)
solutions for each stage (especially the initial thought phase), then compare, synthesize, or
majority-vote to select the most robust or recurring themes before entering critique and
revision loops.
Repeat as needed.
I. LLM Output Configuration (Crucial to Prompting)
Output Length (Max Tokens)
Control output length based on task complexity.
Iterative and self-consistency loops need extra tokens for all rounds.
Sampling Controls: Temperature, Top-K, Top-P
Temperature:
High: For creative, diverse initial thoughts/self-consistency runs.
Low: For critique, evaluation, and final synthesis.
Top-K/P:
High: For broad solution pools in self-consistency rounds.
Low: For focus and determinism at critique/synthesis.
Always consider interaction between these settings to avoid edge cases like repetition
loops.
II. Core Prompting Techniques (With Iterative Self-Consistency)
General/Zero-Shot Prompting
State the task clearly.
Self-consistency: For complex queries, run the prompt several times with high
temperature, collect diverse outputs, then review for consistency or synthesize before
finalizing.
Example instruction: "Explain X. Before finalizing, review and compare multiple answers for
accuracy and coherence. Refine as needed."
One-Shot & Few-Shot Prompting
Use 1 or several examples to show ideal structure and steps.
Self-consistency: For each example, especially nuanced ones, prompt for multiple possible
responses, then aggregate, compare, and refine. Use rounds with diverse examples to seed
broad reasoning.
System, Contextual, and Role Prompting
System Prompting: Set context and process. Instruct the LLM to generate, selfconsistently vary, critically review, and refine all outputs.
Role Prompting: Assign roles that both generate and self-critique with self-consistency, e.g.
"Emulate multiple expert reviewers and merge their consensus before revision."
Context Prompting: Supply context, then invoke self-consistency to ensure context is
applied similarly across multiple generations, with post-run synthesis/critique.
Step-Back Prompting
Ask a general/abstract principle, then apply it.
Self-consistency: For both the abstract and application steps, run multiple variants and
select aggregates or most convincing responses for each stage.
Chain of Thought (CoT) Prompting
Request step-by-step reasoning.
Self-consistency: For each CoT stage, prompt for multiple reasoning paths, then majorityvote or synthesize, then proceed to further critique and revision.
Example prompt: "Let's think step by step, providing several possible chains. Review each
for flaws, then synthesize the most comprehensive path and answer."
Self-Consistency Prompting
Extend any multi-step prompt with several high-temperature runs.
Compare, synthesize, or majority-vote results for each loop. Use this not only for final
answers, but also for intermediate steps and critiques.
Tree of Thoughts (ToT)
Explore multiple reasoning paths in branching structure.
At each branch, generate self-consistent variants; compare, prune, or aggregate before
exploring further. Synthesize the most promising branches and refine at final output.
ReAct (Reason & Act)
Combine reasoning with tool usage steps.
For each major reasoning or observed step, generate self-consistent alternatives (especially
when multiple tools could yield different outputs), then compare and use consensus to guide
the next action or revision.
Automatic Prompt Engineering (APE)
Use LLM to generate/refine prompts.
For prompt generation and evaluation, run multiple self-consistent prompt candidates,
compare their effectiveness, and select/refine by synthesizing best features from each run.
Code Prompting
For code write, explain, translate, debug, or review: Use multi-run self-consistency at each
output or critique stage.
Example: "Write code for X. Now run several bug checks and propose multiple fixes. Review
and synthesize most robust version with explanations."
III. General Best Practices (With Iterative Self-Consistency)
1. Embrace both iterative and self-consistency loops: Always prompt for multiple initial outputs
at high diversity, critique or compare, synthesize consensus, and refine.
2. Provide diverse examples and variant outputs: Use examples that demonstrate the process
of generating, comparing, critiquing, and refining—especially for edge cases.
3. Design clear, multi-stage instructions: Specify how to generate, self-consistently vary,
compare, critique, and finally revise outputs.
4. Be specific: Detail output requirements for each stage, and instruct the model to compare
variants/self-consistent runs before finalizing.
5. Prefer detailed instructions: "Generate three possible answers, compare for agreement,
critique, then synthesize best features into your final output."
6. Control token length generously for all refinement and self-consistency rounds.
Task: [Describe task]
Step 1: Generate [N] diverse initial outputs (high temperature sampling).
Step 2: Critically compare these outputs. Identify commonalities, flaws, and strengths.
Step 3: Synthesize or majority-vote among the variants to determine the most robust answe
Step 4: Refine the selected answer by reviewing it for flaws or omissions. Propose improv
Step 5: Produce the revised, final output integrating all improvements and consensus insi
Step 6: Document the sequence and rationale for each stage.
This paradigm can be overlaid onto all prompting strategies for higher accuracy and robustness.
For further details and technique-specific instructions, see context from the original source as
integrated above.
7. Use variables to make self-consistency and iterative prompts reusable.
8. Experiment: Test alternative phrasings for generation, variation, critique, and synthesis.
9. Mix up classes/examples for robust classification and synthesis.
10. Adapt and update: New models handle refinement and self-consistency differently—
experiment and document results.
11. Try intermediate output formats (JSON, schemas) for structured self-consistency at each
stage.
12. Track and review: Document every prompt, variant, critique, consensus, and final answer for
the full recursive/self-consistency loop.
Template for Fully Integrated Iterative Self-Consistency Prompt