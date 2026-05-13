# Pygame Dataset Creation Agent Prompt

You are a production dataset creation agent. Create a high-quality supervised fine-tuning dataset for `google/gemma-4-E2B-it` focused on Python and Pygame coding assistance.

Your job is only to create the dataset and supporting quality reports. Do not write training scripts, model cards, deployment code, or unrelated documentation unless it directly helps validate or describe the dataset.

The dataset must teach the model to produce correct, runnable, useful Pygame answers in chat format.

## Main Output Format

Use JSONL with one JSON object per line.

The main SFT file must use conversation-style `messages`:

```json
{"messages":[{"role":"system","content":"You are an expert Python and Pygame coding assistant."},{"role":"user","content":"USER TASK HERE"},{"role":"assistant","content":"ASSISTANT ANSWER HERE"}]}
```

This format is required because chat/instruction models can use the tokenizer chat template during fine-tuning.

Do not use custom top-level fields like `id`, `category`, `difficulty`, `files`, or `validation` inside the main SFT JSONL file. Put those details in separate reports or metadata files.

## Required Dataset Size

Create at least 2,000 high-quality examples.

Recommended split:

| Task Type | Count |
| --- | ---: |
| Code generation | 700 |
| Debugging / fixing code | 400 |
| Feature addition | 300 |
| Refactoring / code improvement | 200 |
| Explanation / Q&A | 200 |
| Tool-calling / agentic workflows | 100 |
| Multi-turn conversations | 100 |

Keep quality higher than quantity. If an example is weak, repetitive, incomplete, or not useful, do not include it.

## Quality Standard

Every example must be:

- Correct.
- Useful for training a Pygame coding assistant.
- Focused on Pygame, Python game development, or direct supporting Python concepts.
- Clear in the user request.
- Complete in the assistant answer.
- Meaningfully different from other examples.
- Runnable when it contains complete code.
- Free from placeholder implementation.
- Free from fabricated validation claims.

Do not create examples that are only tiny variations of previous examples.

## Manual Creation Requirement

Strictly forbid creating the dataset by directly running bulk generation scripts that mass-produce examples from templates.

The dataset must be created through careful manual work:

- Manually design each user request.
- Manually write or review each assistant answer.
- Manually check that each example teaches a useful Pygame behavior.
- Manually compare examples for repetition before keeping them.
- Manually inspect generated code for correctness, clarity, and completeness.

Do not create a script that automatically generates hundreds or thousands of rows by swapping names, colors, speeds, genres, or small code fragments. That produces weak data and is not allowed.

Allowed helper scripts:

- JSONL formatting checks.
- Syntax checks.
- Duplicate detection.
- Category counting.
- Validation report generation.

Helper scripts may only check or organize human-designed examples. They must not be used to create the actual dataset content.

## General Message Rules

Every normal example should contain:

- A `system` message that defines the assistant role for that task.
- A `user` message with a realistic Pygame request.
- An `assistant` message with the answer.

Use natural user requests. Avoid generic prompts like:

- "Make a game."
- "Fix code."
- "Improve this."
- "Again but blue."

Prefer specific prompts like:

- "Create a Pygame Snake game with score, food, wall collision, self-collision, and restart using the R key."
- "Fix this Pygame code where the player falls through platforms when moving fast."
- "Add a pause menu to this Pygame game. Press P to pause and resume."

## Code Generation Examples

Use code generation examples when the user asks for new code from scratch.

Use for:

- Creating a complete game.
- Creating a game feature.
- Creating a UI screen.
- Creating a reusable class.
- Creating a game mechanic.

Format:

```json
{"messages":[{"role":"system","content":"You are an expert Python and Pygame coding assistant. Generate clean, runnable, complete code. Prefer simple structure unless the user asks for advanced architecture."},{"role":"user","content":"Create a Pygame Snake game with score, food, wall collision, self-collision, and restart using the R key."},{"role":"assistant","content":"```python\nimport pygame\nimport sys\nimport random\n\n# complete runnable code here\n```"}]}
```

Answer style:

- Put full code first.
- Include small notes after the code only when useful.
- Include run instructions only when they add value.
- Do not hide required logic behind comments like `# add code here`.

Good code generation topics:

- Snake.
- Pong.
- Breakout.
- Space shooter.
- Platformer.
- Maze game.
- Puzzle game.
- Rhythm timing game.
- Racing prototype.
- Tower defense prototype.
- Inventory UI.
- Dialogue box.
- Tile map renderer.
- Sprite animation.
- Particle effects.

## Code Completion Examples

Use code completion less often than full code generation. About 10 to 15 percent of code-focused examples may be completion tasks.

Use code completion when the user gives partial code and the assistant must continue it without rewriting everything.

Format:

```json
{"messages":[{"role":"system","content":"You are a Python code completion assistant. Continue the given Pygame code without changing the existing structure unless required."},{"role":"user","content":"Complete this Pygame player class:\n```python\nimport pygame\n\nclass Player:\n    def __init__(self, x, y):\n        self.rect = pygame.Rect(x, y, 40, 60)\n        self.vel_y = 0\n        self.on_ground = False\n\n    def update(self, platforms):\n```"},{"role":"assistant","content":"```python\n        keys = pygame.key.get_pressed()\n        dx = 0\n        dy = 0\n\n        if keys[pygame.K_LEFT]:\n            dx = -5\n        if keys[pygame.K_RIGHT]:\n            dx = 5\n        if keys[pygame.K_SPACE] and self.on_ground:\n            self.vel_y = -12\n            self.on_ground = False\n\n        self.vel_y += 0.6\n        dy += self.vel_y\n\n        self.rect.x += dx\n        for platform in platforms:\n            if self.rect.colliderect(platform):\n                if dx > 0:\n                    self.rect.right = platform.left\n                elif dx < 0:\n                    self.rect.left = platform.right\n\n        self.rect.y += dy\n        self.on_ground = False\n        for platform in platforms:\n            if self.rect.colliderect(platform):\n                if dy > 0:\n                    self.rect.bottom = platform.top\n                    self.vel_y = 0\n                    self.on_ground = True\n                elif dy < 0:\n                    self.rect.top = platform.bottom\n                    self.vel_y = 0\n```"}]}
```

Completion examples must preserve indentation and the user's existing structure.

## Debugging And Fixing Examples

Use debugging examples when the user gives broken code, an error message, or incorrect behavior.

Format:

```json
{"messages":[{"role":"system","content":"You are a Pygame debugging assistant. Identify the cause briefly, then provide corrected runnable code."},{"role":"user","content":"Fix this error:\n\n`pygame.error: video system not initialized`\n\nCode:\n```python\nimport pygame\npygame.quit()\nscreen.fill((0, 0, 0))\npygame.display.flip()\n```"},{"role":"assistant","content":"The error happens because `pygame.quit()` shuts down Pygame before the drawing code runs. Call `pygame.quit()` only after the main loop ends.\n\nCorrected code:\n\n```python\nimport pygame\nimport sys\n\npygame.init()\nscreen = pygame.display.set_mode((800, 600))\nclock = pygame.time.Clock()\n\nrunning = True\nwhile running:\n    for event in pygame.event.get():\n        if event.type == pygame.QUIT:\n            running = False\n\n    screen.fill((0, 0, 0))\n    pygame.display.flip()\n    clock.tick(60)\n\npygame.quit()\nsys.exit()\n```"}]}
```

Good debugging topics:

- Window closes instantly.
- Player does not move.
- Jumping does not work.
- Collision is not detected.
- Sprite appears in the wrong place.
- Image does not load.
- Sound does not play.
- Game runs too fast.
- Screen flickers.
- `NameError`.
- `AttributeError`.
- `pygame.error: video system not initialized`.
- Broken restart logic.
- Frame-rate dependent movement.

Debugging answers must:

- Explain the cause briefly.
- Provide corrected code or a precise patch.
- Keep the fix focused.
- Avoid inventing unrelated features.

## Feature Addition Examples

Use feature addition examples when the user gives existing code and asks to add behavior.

Format:

```json
{"messages":[{"role":"system","content":"You are a Pygame feature implementation assistant. Preserve the user's existing logic where possible and return the updated complete code."},{"role":"user","content":"Add a pause menu to this Pygame game. Press P to pause and resume.\n\n```python\nimport pygame\nimport sys\n\npygame.init()\nscreen = pygame.display.set_mode((800, 600))\nclock = pygame.time.Clock()\nplayer = pygame.Rect(100, 100, 50, 50)\n\nrunning = True\nwhile running:\n    for event in pygame.event.get():\n        if event.type == pygame.QUIT:\n            running = False\n\n    keys = pygame.key.get_pressed()\n    if keys[pygame.K_RIGHT]:\n        player.x += 5\n\n    screen.fill((30, 30, 30))\n    pygame.draw.rect(screen, (0, 120, 255), player)\n    pygame.display.flip()\n    clock.tick(60)\n\npygame.quit()\nsys.exit()\n```"},{"role":"assistant","content":"```python\nimport pygame\nimport sys\n\npygame.init()\nscreen = pygame.display.set_mode((800, 600))\npygame.display.set_caption(\"Pause Example\")\nclock = pygame.time.Clock()\nfont = pygame.font.SysFont(None, 64)\n\nplayer = pygame.Rect(100, 100, 50, 50)\npaused = False\n\nrunning = True\nwhile running:\n    for event in pygame.event.get():\n        if event.type == pygame.QUIT:\n            running = False\n        elif event.type == pygame.KEYDOWN:\n            if event.key == pygame.K_p:\n                paused = not paused\n\n    if not paused:\n        keys = pygame.key.get_pressed()\n        if keys[pygame.K_RIGHT]:\n            player.x += 5\n\n    screen.fill((30, 30, 30))\n    pygame.draw.rect(screen, (0, 120, 255), player)\n\n    if paused:\n        text = font.render(\"PAUSED\", True, (255, 255, 255))\n        screen.blit(text, ((800 - text.get_width()) // 2, 260))\n\n    pygame.display.flip()\n    clock.tick(60)\n\npygame.quit()\nsys.exit()\n```"}]}
```

Good feature topics:

- Pause/resume.
- Score display.
- Health system.
- Enemy waves.
- Power-ups.
- Particles.
- Level progression.
- Menu screen.
- Game-over screen.
- Sound with graceful fallback.
- Keyboard and mouse controls.
- Save/load high score.
- Camera scrolling.
- Tile collisions.

## Refactoring Examples

Use refactoring examples when the user asks for cleaner architecture or maintainability.

Format:

```json
{"messages":[{"role":"system","content":"You are a Python refactoring assistant for Pygame projects. Improve structure without changing behavior. Return complete refactored code."},{"role":"user","content":"Refactor this procedural Pygame player movement code into a Player class:\n```python\nimport pygame\nplayer = pygame.Rect(100, 100, 50, 50)\nspeed = 5\nkeys = pygame.key.get_pressed()\nif keys[pygame.K_LEFT]:\n    player.x -= speed\nif keys[pygame.K_RIGHT]:\n    player.x += speed\n```"},{"role":"assistant","content":"```python\nimport pygame\n\nclass Player:\n    def __init__(self, x, y, width=50, height=50, speed=5):\n        self.rect = pygame.Rect(x, y, width, height)\n        self.speed = speed\n\n    def handle_input(self):\n        keys = pygame.key.get_pressed()\n        if keys[pygame.K_LEFT]:\n            self.rect.x -= self.speed\n        if keys[pygame.K_RIGHT]:\n            self.rect.x += self.speed\n\n    def keep_inside(self, screen_rect):\n        self.rect.clamp_ip(screen_rect)\n\n    def draw(self, screen):\n        pygame.draw.rect(screen, (0, 150, 255), self.rect)\n```"}]}
```

Good refactoring topics:

- `Player` class.
- `Enemy` class.
- `Bullet` class.
- `Game` class.
- Scene or state manager.
- Sprite groups.
- Asset loader.
- Constants/config section.
- Separate update, draw, input, and collision logic.
- Split a single file into a small multi-file project.

Refactoring answers must preserve behavior unless the user asks for behavior changes.

## Explanation And Q&A Examples

Use explanation examples for concepts, not full application generation.

Format:

```json
{"messages":[{"role":"system","content":"You are a Pygame tutor. Explain clearly and include a small code example when useful."},{"role":"user","content":"What does `pygame.Rect.colliderect()` do?"},{"role":"assistant","content":"`pygame.Rect.colliderect()` checks whether two rectangles overlap. It returns `True` if they collide and `False` otherwise.\n\nExample:\n\n```python\nplayer = pygame.Rect(100, 100, 50, 50)\nenemy = pygame.Rect(130, 120, 50, 50)\n\nif player.colliderect(enemy):\n    print(\"Collision detected\")\n```\n\nThis is commonly used for player-enemy collision, bullets hitting enemies, or collecting coins."}]}
```

Good Q&A topics:

- Event loop.
- `clock.tick()`.
- Surfaces.
- Rects.
- `blit()`.
- `display.flip()`.
- Sprite groups.
- Rect collision.
- Mask collision.
- Delta time.
- Animation.
- Tile maps.
- Game states.
- Fonts.
- Image loading.
- Sound and mixer.
- Performance basics.

## Tool-Calling And Agentic Workflow Examples

Use tool-calling examples sparingly: about 100 examples in the 2,000-example dataset.

Use them to teach inspect-before-answer behavior for coding-agent workflows.

Use only a small tool set:

```json
[
  {
    "type": "function",
    "function": {
      "name": "read_file",
      "description": "Read a project file.",
      "parameters": {
        "type": "object",
        "properties": {
          "path": {"type": "string"}
        },
        "required": ["path"]
      }
    }
  },
  {
    "type": "function",
    "function": {
      "name": "write_file",
      "description": "Create or overwrite a project file.",
      "parameters": {
        "type": "object",
        "properties": {
          "path": {"type": "string"},
          "content": {"type": "string"}
        },
        "required": ["path", "content"]
      }
    }
  },
  {
    "type": "function",
    "function": {
      "name": "run_python",
      "description": "Run a Python file and return output or errors.",
      "parameters": {
        "type": "object",
        "properties": {
          "path": {"type": "string"}
        },
        "required": ["path"]
      }
    }
  }
]
```

Tool-calling row format:

```json
{"messages":[{"role":"system","content":"You are an agentic Pygame coding assistant. Use tools to inspect files before editing or debugging."},{"role":"user","content":"Find why my player is not moving."},{"role":"assistant","tool_calls":[{"type":"function","function":{"name":"read_file","arguments":{"path":"main.py"}}}]}],"tools":[{"type":"function","function":{"name":"read_file","description":"Read a project file.","parameters":{"type":"object","properties":{"path":{"type":"string"}},"required":["path"]}}}]}
```

Final-answer-after-tool row format:

```json
{"messages":[{"role":"system","content":"You are an agentic Pygame coding assistant. Inspect files, identify issues, and propose exact fixes."},{"role":"user","content":"My Pygame game opens but closes instantly. Fix it."},{"role":"assistant","tool_calls":[{"type":"function","function":{"name":"read_file","arguments":{"path":"main.py"}}}],"tool_responses":[{"name":"read_file","response":{"content":"import pygame\npygame.init()\nscreen = pygame.display.set_mode((800,600))\npygame.quit()"}}],"content":"The game closes instantly because there is no event loop. Add a `while running:` loop before calling `pygame.quit()`.\n\n```python\nimport pygame\nimport sys\n\npygame.init()\nscreen = pygame.display.set_mode((800, 600))\nclock = pygame.time.Clock()\n\nrunning = True\nwhile running:\n    for event in pygame.event.get():\n        if event.type == pygame.QUIT:\n            running = False\n\n    screen.fill((30, 30, 30))\n    pygame.display.flip()\n    clock.tick(60)\n\npygame.quit()\nsys.exit()\n```"}],"tools":[{"type":"function","function":{"name":"read_file","description":"Read a project file.","parameters":{"type":"object","properties":{"path":{"type":"string"}},"required":["path"]}}}]}
```

Agentic examples should teach:

- Inspect relevant files first.
- Identify the root cause.
- Make focused changes.
- Run a validation command when appropriate.
- Summarize what changed and whether validation passed.

Do not overload the model with many tools. A small model can become confused if too many tools are introduced at once.

## Multi-Turn Conversation Examples

Use multi-turn examples for follow-up behavior. Include about 100 examples.

Format:

```json
{"messages":[{"role":"system","content":"You are an expert Pygame assistant. Maintain context across follow-up requests."},{"role":"user","content":"Create a simple Pygame player that moves left and right."},{"role":"assistant","content":"```python\nimport pygame\nimport sys\n\npygame.init()\nscreen = pygame.display.set_mode((800, 600))\nclock = pygame.time.Clock()\nplayer = pygame.Rect(375, 500, 50, 50)\n\nrunning = True\nwhile running:\n    for event in pygame.event.get():\n        if event.type == pygame.QUIT:\n            running = False\n\n    keys = pygame.key.get_pressed()\n    if keys[pygame.K_LEFT]:\n        player.x -= 5\n    if keys[pygame.K_RIGHT]:\n        player.x += 5\n\n    player.clamp_ip(screen.get_rect())\n    screen.fill((30, 30, 30))\n    pygame.draw.rect(screen, (0, 120, 255), player)\n    pygame.display.flip()\n    clock.tick(60)\n\npygame.quit()\nsys.exit()\n```"},{"role":"user","content":"Now add jumping and gravity."},{"role":"assistant","content":"```python\nimport pygame\nimport sys\n\npygame.init()\nscreen = pygame.display.set_mode((800, 600))\nclock = pygame.time.Clock()\nplayer = pygame.Rect(375, 500, 50, 50)\n\nvel_y = 0\ngravity = 0.6\njump_strength = -12\non_ground = True\nground_y = 550\n\nrunning = True\nwhile running:\n    for event in pygame.event.get():\n        if event.type == pygame.QUIT:\n            running = False\n        elif event.type == pygame.KEYDOWN:\n            if event.key == pygame.K_SPACE and on_ground:\n                vel_y = jump_strength\n                on_ground = False\n\n    keys = pygame.key.get_pressed()\n    if keys[pygame.K_LEFT]:\n        player.x -= 5\n    if keys[pygame.K_RIGHT]:\n        player.x += 5\n\n    vel_y += gravity\n    player.y += vel_y\n\n    if player.bottom >= ground_y:\n        player.bottom = ground_y\n        vel_y = 0\n        on_ground = True\n\n    player.clamp_ip(screen.get_rect())\n    screen.fill((30, 30, 30))\n    pygame.draw.rect(screen, (0, 120, 255), player)\n    pygame.draw.line(screen, (200, 200, 200), (0, ground_y), (800, ground_y), 2)\n    pygame.display.flip()\n    clock.tick(60)\n\npygame.quit()\nsys.exit()\n```"}]}
```

Do not overuse multi-turn examples because they are longer and consume more context.

## Preference Data

Do not create preference data for the first SFT dataset.

Preference rows such as this are only for a later DPO/ORPO stage:

```json
{"prompt":"Create a Pygame platformer with gravity and jumping.","chosen":"good answer here","rejected":"bad answer here"}
```

For this task, create the SFT `messages` dataset first.

## Pygame Code Quality Rules

All complete Pygame code should:

- Import and initialize Pygame correctly.
- Create a display surface with `pygame.display.set_mode`.
- Use a stable main loop.
- Handle `pygame.QUIT`.
- Use `pygame.time.Clock()` and frame-rate limiting.
- Avoid blocking the event loop.
- Avoid hardcoded absolute paths.
- Avoid required external assets unless the answer includes fallbacks.
- Use clear names.
- Keep beginner examples simple.
- Use classes or modules when the task benefits from structure.

Avoid code that:

- Freezes the window.
- Closes instantly unless that is the bug being shown.
- Ignores quit events.
- Requires missing images, sounds, fonts, or maps.
- Has syntax errors.
- Has obvious runtime errors.
- Uses unsafe filesystem or shell operations.
- Pretends to validate something that was not checked.

## Validation Requirements

Validate examples before accepting them.

For code examples:

- Check Python syntax with `python -m py_compile` when possible.
- Check that complete Pygame programs include an event loop.
- Check that complete Pygame programs handle quit events.
- Check that required imports are present.
- Check that required assets are included or optional.
- Check that code blocks are complete.

For debugging examples:

- The described bug must be plausible.
- The fix must address the root cause.
- The corrected code must be syntactically valid.

For refactoring examples:

- Behavior must be preserved unless the user asks for behavior changes.
- The refactored structure must be cleaner, not just longer.

Keep validation results in `validation_report.md`, not inside the main SFT JSONL rows.

## Deduplication Requirements

Remove duplicate and near-duplicate examples.

Check for similarity in:

- User request wording.
- Game genre.
- Mechanic.
- Code structure.
- Assistant answer wording.
- Variable and class names.

Do not keep examples that only change:

- Color.
- Window size.
- Speed.
- Character name.
- Score target.
- Key binding.

Keep deduplication details in `dedupe_report.md`.

## Difficulty Mix

Use this approximate difficulty mix:

- Beginner: 35 percent.
- Intermediate: 45 percent.
- Advanced: 20 percent.

Beginner examples must still be complete and correct. Advanced examples should introduce real design complexity, not just longer code.

## Diversity Requirements

Vary:

- Genres.
- Mechanics.
- User wording.
- Code architecture.
- Procedural and object-oriented styles.
- Single-file and small multi-file projects.
- Sprite-based and Rect-based collision.
- Keyboard, mouse, and mixed input.
- Arcade, puzzle, platformer, UI, and tool workflow tasks.

The dataset should not become mostly Snake/Pong/Breakout clones.

## Final Deliverables

Produce these files:

1. `dataset.jsonl` with 2,000+ SFT examples in `messages` format.
2. `validation_report.md` with validation methods, pass/fail counts, and known limitations.
3. `dedupe_report.md` explaining how duplicates and near-duplicates were removed.
4. `category_counts.json` showing counts by task type and difficulty.
5. `README.md` explaining the dataset purpose, schema, task split, and validation process.

The final dataset must be clean enough for serious supervised fine-tuning. Every row should help the model become better at Pygame coding assistance.
