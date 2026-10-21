# assign — make this session the team lead

1. Find the team lead block (`team-lead:start`…`end`) in the instruction files already loaded for this session. If there is none, use `templates/block.md` filled with init's recommended options, this platform's default subagent, and the role instructions from `templates/roles/` as briefs, and add one line at the end saying init can set it up.
2. Check each role's method on this platform (agent definition exists, CLI installed with `command -v`). Pick substitutes by the block's fallback order.
3. From now until the session ends, follow the block. Tell the user: one line that the lead is assigned, the roster (role · method · executor, substitutes marked), and the key rules in up to 3 lines.
4. If a task came after `assign`, break it down and dispatch it now. Otherwise ask for the first task.
