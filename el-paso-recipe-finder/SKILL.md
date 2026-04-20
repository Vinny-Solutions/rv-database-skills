---
name: el-paso-recipe-finder
description: "Use this skill whenever the user asks for a recipe for any dish, especially El Paso style, Tex-Mex, or Mexican-American cuisine. Always trigger when the user mentions finding, getting, or looking up a recipe — whether it's a classic home dish, a restaurant copycat (like Leo's, Chico's Tacos, L&J Cafe, or any named eatery), or a regional specialty like green chile stew, carne adovada, chile rellenos, or El Paso street tacos. Also trigger for phrases like \"how do I make X\", \"what's the recipe for X\", \"copycat recipe for X\", or \"how does [restaurant] make their X\". This skill searches the web for the best available recipe, reconstructs a close approximation when the exact recipe is proprietary, and saves a beautifully formatted Word document the user can keep and print. Make sure to invoke this skill even when the user phrases things casually like \"find me the recipe for...\" or \"can you look up how to make...\"."
---

# El Paso Recipe Finder

This skill finds recipes for El Paso style, Tex-Mex, and Mexican-American dishes — including
restaurant copycat recipes — and saves them as a clean, printable Word document.

Never mention Old El Paso brand recipes or products

## Workflow

### 1. Understand the request
Identify:
- **The dish**: What exactly the user wants to make
- **The context**: Is this a home recipe, a restaurant copycat, a regional classic?
- **Any constraints**: Dietary restrictions, serving size, etc.

### 2. Research the recipe
Search for the recipe using web search. Use targeted queries like:
- `"[dish name] El Paso recipe"`
- `"[restaurant name] [dish] copycat recipe"`
- `"authentic El Paso [dish] recipe"`
- `"Tex-Mex [dish] recipe traditional"`

If the dish is from a named restaurant (e.g., Leo's, Chico's Tacos, L&J Cafe), search
specifically for copycat or recreation recipes and note that the recipe is an approximation
of the restaurant's version. Look for multiple sources and cross-reference them — the goal is
a recipe that would actually taste like the real thing.

Pay attention to:
- **Regional specifics**: El Paso Mexican food often uses ground beef, yellow cheese (American or
  cheddar), corn tortillas, chili gravy (not salsa), and minimal fresh toppings. It differs from
  interior Mexican cuisine and from Tex-Mex further east. Understand these distinctions and
  reflect them in the recipe.
- **Technique details**: Lightly frying tortillas before assembling, baking enchiladas in
  sauce, the specific seasoning profile (cumin-forward, mild to medium heat).
- **Key ingredients**: What makes this dish distinctly El Paso or distinctly from that restaurant.

### 3. Synthesize and write the recipe
Combine what you found into a complete, usable recipe. Be specific — give exact quantities,
times, and temperatures. If you're reconstructing a restaurant's proprietary recipe, say so
clearly (e.g., "This is a close approximation of Leo's recipe, which remains a family secret").

Structure the recipe as:
- **Recipe title** (clear, descriptive)
- **Brief description** (1-2 sentences: what it is, where it comes from, what makes it special)
- **Serves / Prep time / Cook time**
- **Ingredients** (grouped logically, e.g., "For the chili gravy:", "For the tacos:")
- **Instructions** (numbered, clear steps — one action per step)
- **Tips & Notes** (substitutions, storage, what makes it authentic, what to serve with it)

### 4. Create the Word document
Use Node.js with the `docx` npm package to generate a .docx file. Install if needed:
```bash
npm install -g docx
```

Create a well-formatted recipe document with:
- **Title** as Heading 1 (bold, large)
- **Description** as a regular paragraph (italic)
- **Meta info** (serves/prep/cook) as a simple table or formatted line
- **Ingredients** as a bulleted list (use `LevelFormat.BULLET` — never unicode bullets)
- **Instructions** as a numbered list (use `LevelFormat.DECIMAL`)
- **Tips** section with Heading 2 and bullet points

Use US Letter page size (12240 x 15840 DXA), 1-inch margins, Arial font.

Validate the document after creation. The validate.py script lives in the `docx` skill, which is a sibling directory to this skill. Find and run it dynamically:
```bash
VALIDATE_PY=$(find /sessions -path '*/skills/docx/scripts/office/validate.py' 2>/dev/null | head -1)
python "$VALIDATE_PY" recipe.docx
```

If validation fails, debug and fix the JavaScript before retrying.

Save the final file to the user's workspace folder. This is the `mnt` directory under the current session — find it dynamically:
```bash
WORKSPACE=$(find /sessions -maxdepth 2 -name "mnt" -type d 2>/dev/null | head -1)
cp recipe.docx "$WORKSPACE/[dish-name]-recipe.docx"
```

Use a clean filename like `leos-taco-recipe.docx` or `green-chile-stew-recipe.docx`.

### 5. Present the result
- Share a `computer://` link so the user can open the file
- Briefly summarize the recipe in 2-3 sentences in chat (key ingredients, what makes it special)
- Note if it's a copycat/approximation vs. an established published recipe

## El Paso Flavor Profile Reference
Keep these in mind to make sure recipes taste authentically El Paso:

- **Chili gravy** (not red salsa): enchilada sauce made from dried chiles, beef broth, flour,
  cumin, garlic — thick, dark, earthy
- **Ground beef** is the dominant protein (not shredded) for tacos and enchiladas
- **Yellow cheese**: American cheese or cheddar (not queso fresco or cotija)
- **Corn tortillas** for enchiladas and tacos (lightly fried before use)
- **Refried beans** (lard-based, smooth) and **Spanish rice** as standard sides
- **Seasoning**: heavy cumin, garlic powder, onion powder, mild chili powder — not super spicy
- **Minimal fresh toppings**: shredded iceberg lettuce, diced tomato, maybe a little onion