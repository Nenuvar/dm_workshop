---
aliases:
source: homebrew
image: z/assets/npcs/dwarf-graveslayer.webp
age:
race:
class:
gender:
status:
data:
  npc:
    backstory: This is albemarles backstory
    role: ""
    secrets:
      first:
        revealed: false
        title: Albies secret
        truth: Albies truth
        trigger: Albies trigger
      second:
        revealed: false
related_template: "[[npc]]"
---
ADD THIS

> [!success] Home Location: `INPUT[suggester(optionQuery('"_worldbuilding/Locations"')):rel.parent]` 

> [!column|no-i no-t]
>>[!note|no-title] NPC Info
>> ~~~meta-bind
>> INPUT[select(
>> option(1, 🌐 Image),
>> option(2, ⚔️ Info),
>> class(tabbed)
>> )]
>> ~~~
>>>[!tabbed-box-maxh]
>>> `INPUT[imageSuggester(optionQuery("z/assets/npcs")):image]`
>>>
>>> >[!div-m|no-title]
>>> > ![[#Info|no-h clean]]
>
>> [!note|no-title] NPC Info
>> ~~~meta-bind
>> INPUT[select(
>> option(1, 🌐 Overview),
>> option(2, ⚔️ Statblock),
>> option(3, 🔒 DM Secrets),
>> class(tabbed)
>> )]
>> ~~~
>>>[!tabbed-box-maxh]
>>> >[!div-m|no-title]
>>> > ![[#Overview|no-h clean]]
>>>
>>> >[!div-m|no-title]
>>> > ![[#Statblock|no-h clean]]
>>>
>>> > [!div-m|no-title]
>>> > ![[#DM Secrets|no-h clean]]
>>> 

> [!NOTE|no-title]
> ~~~meta-bind
> INPUT[select(
> option(1, 📜 Backstory),
> option(2, 🧑‍🤝‍🧑 Relationships),
> option(3, 🔗 Connections),
> class(tabbed)
> )]
> ~~~
> >[!tabbed-box]
> >
> > >[!div-m|no-title]
> > > ![[#Backstory|no-h clean]]
> >
> > >[!div-m|no-title]
> > > ![[#Relationships|no-h clean]]
> > 
> > >[!div-m|no-title]
> > > ![[#Connections|no-h clean]]
> >


## INPUT FIELDS

### Info

```dataviewjs
// --- CONFIGURATION ---
const raceFolder = '"z/CLI/races"';
const classFolder = '"z/CLI/classes"';

// --- DYNAMIC LISTS (Race & Class) ---

// 1. Races
const races = dv.pages(raceFolder)
    .where(p => p.file.name !== "races")
    .map(p => (p.aliases && p.aliases.length > 0) ? p.aliases[0] : p.file.name)
    .distinct().sort(n => n, "asc").array();
const raceOptions = races.map(r => `option('${r}')`).join(", ");

// 2. Classes
const classes = dv.pages(classFolder)
    .where(p => p.file.name !== "classes")
    .map(p => (p.aliases && p.aliases.length > 0) ? p.aliases[0] : p.file.name)
    .distinct().sort(n => n, "asc").array();
const classOptions = classes.map(c => `option('${c}')`).join(", ");


// --- STATIC LISTS (Age, Gender, Status) ---

// 3. Age
const ageOptions = "option('🧸 baby'), option('🪀 child'), option('🗡️ adolesence'), option('⚔️ adult'), option('🗝️ elder'), option('🐲 ancient')";

// 4. Gender (Corrected typo 'netural' to 'neutral')
const genderOptions = "option('⚥ neutral'), option('♂️ male'), option('♀️ female')";

// 5. Status
const statusOptions = "option('💛 alive'), option('⚰️ dead'), option('❔ unknown')";


// --- RENDER ---
// This creates the input strings bound to specific metadata keys
const raceInput = `\`INPUT[inlineSelect(${raceOptions}):data.npc.race]\``;
const classInput = `\`INPUT[inlineSelect(${classOptions}):data.npc.class]\``;
const ageInput = `\`INPUT[inlineSelect(${ageOptions}):data.npc.age]\``;
const genderInput = `\`INPUT[inlineSelect(${genderOptions}):data.npc.gender]\``;
const statusInput = `\`INPUT[inlineSelect(${statusOptions}):data.npc.status]\``;

// Display them in a simple list
dv.paragraph(`
|  |  |
| :--- | :--- |
| **Race** | ${raceInput} |
| **Class** | ${classInput} |
| **Age** | ${ageInput} |
| **Gender** | ${genderInput} |
| **Status** | ${statusInput} |
`);

```

### Statblock

```dataviewjs
// 1. Define the path to your bestiary folder
const folderPath = "z/CLI/bestiary"; 

// 2. Fetch all pages in that folder
const pages = dv.pages(`"${folderPath}"`);
let monsterNames = [];

// 3. Loop through files and extract the exact name
for (let page of pages) {
    const content = await dv.io.load(page.file.path);
    const statblockMatch = content.match(/```statblock[\s\S]*?```/);
    
    if (statblockMatch) {
        const nameMatch = statblockMatch[0].match(/^"name":\s*"([^"]+)"/m);
        if (nameMatch && nameMatch[1]) {
            // No escaping needed! We take the exact name as it is.
            monsterNames.push(nameMatch[1]);
        }
    }
}

// 4. Sort alphabetically and remove duplicates
monsterNames = [...new Set(monsterNames)].sort();

// 5. Generate a Native Dropdown (Bypassing Meta Bind's limits)
if (monsterNames.length > 0) {
    dv.paragraph("**Statblock**");
    
    // Create the standard HTML dropdown element
    const dropdown = document.createElement("select");
    dropdown.style.padding = "4px"; 
    dropdown.style.borderRadius = "4px";
    dropdown.style.background = "var(--background-modifier-form-field)";
    dropdown.style.color = "var(--text-normal)";
    
    // Add a default empty option
    const defaultOpt = document.createElement("option");
    defaultOpt.text = "-- Select a Monster --";
    defaultOpt.value = "";
    dropdown.add(defaultOpt);
    
    // Populate the monster options
    monsterNames.forEach(name => {
        const opt = document.createElement("option");
        opt.text = name;
        opt.value = name;
        dropdown.add(opt);
    });

    // Set the dropdown to the currently selected value in your YAML (if any)
    const currentSelection = dv.current().data?.npc?.statblock;
    if (currentSelection) {
        dropdown.value = currentSelection;
    }

    // Add the dropdown to our note
    dv.container.appendChild(dropdown);

    // Listen for changes and update the YAML frontmatter directly
    dropdown.addEventListener("change", async (e) => {
        const selectedValue = e.target.value;
        const currentFilePath = dv.current().file.path;
        const file = app.vault.getAbstractFileByPath(currentFilePath);
        
        // Obsidian's official API to safely update YAML
        await app.fileManager.processFrontMatter(file, (frontmatter) => {
            // Ensure the nested data structure exists
            if (!frontmatter.data) frontmatter.data = {};
            if (!frontmatter.data.npc) frontmatter.data.npc = {};
            
            // Update the statblock property
            frontmatter.data.npc.statblock = selectedValue;
        });
    });

} else {
    dv.paragraph("No monsters found in the specified folder.");
}
```


```dataviewjs
// 1. Read the choice from the current file's frontmatter
const currentFile = dv.current();
const selectedMonster = currentFile.data?.npc?.statblock;
// Update this path to match your Bestiary folder
const folderPath = "z/CLI/bestiary"; 

if (selectedMonster) {
    // 2. Fetch all pages in the bestiary to find the matching monster
    const pages = dv.pages(`"${folderPath}"`);
    let foundStatblock = null;

    for (let page of pages) {
        const content = await dv.io.load(page.file.path);
        const statblockMatch = content.match(/```statblock[\s\S]*?```/);
        
        if (statblockMatch) {
            // We escape the monster name safely so parentheses don't break the search
            const escapedName = selectedMonster.replace(/[-\/\\^$*+?.()|[\]{}]/g, '\\$&');
            // Check if this specific statblock contains our selected monster's name
            const nameRegex = new RegExp(`^"name":\\s*"${escapedName}"`, 'm');
            
            if (nameRegex.test(statblockMatch[0])) {
                // If it's a match, save the exact codeblock text and stop searching
                foundStatblock = statblockMatch[0];
                break; 
            }
        }
    }

    // 3. Render the exact codeblock found in the source file
    if (foundStatblock) {
        dv.paragraph(foundStatblock);
    } else {
        dv.paragraph(`> *Could not find the statblock code for **${selectedMonster}** in the bestiary folder.*`);
    }
} else {
    dv.paragraph("> *Please select a monster from the dropdown above to view its statblock.*");
}

```

---

### DM Secrets

1. **Secret Title** --> **Revealed**: `INPUT[toggle:data.npc.secrets.first.revealed]`
	🔒 `INPUT[text(placeholder(the hidden truth)):data.npc.secrets.first.truth]`
	⚔️`INPUT[text(placeholder(the reveal trigger)):data.npc.secrets.first.trigger]`

2. **Secret Title** --> **Revealed**: `INPUT[toggle:data.npc.secrets.second.revealed]`
	🔒 `INPUT[text(placeholder(the hidden truth)):data.npc.secrets.second.truth]`
	⚔️ `INPUT[text(placeholder(the reveal trigger)):data.npc.secrets.second.trigger]`

3. **Secret Title** --> **Revealed**:`INPUT[toggle:data.npc.secrets.third.revealed]`
	🔒 `INPUT[text(placeholder(the hidden truth)):data.npc.secrets.third.truth]`
	⚔️ `INPUT[text(placeholder(the reveal trigger)):data.npc.secrets.third.trigger]`

---

>[!tip]- Explanation
>🔒 The *hidden truth* is the core of the secret - `what` is it? 
>
>⚔️ The *reveal trigger* is the way the PCs will know the secret - `how` will it be revealed?

> [!example]- Example
>**The Imprisoned Heroes**
>	🔒 `the hidden truth:` The ancient "Battlefield of Heroes" is actually a prison; the heroes didn't die defending the land, they were executed here to bind a dark god beneath the soil.
>	⚔️ `the reveal trigger:` The players find the "memorial stones" are actually wards written in Abyssal, designed to keep something **in**, not to honor the dead.

### Overview

📜 `Three Point Info`: 
- **Role:** 
- **Goal:** 
- **Quirk:** 

✒️ `More Info`
- g

---

> [!tip]- Tip
> 1. Describe the NPC's primary `role` in the world.
> 	*What is their job, function, or narrative purpose?*
> 	`A retired adventurer who now runs the sleepy tavern; but still has their old sword under the bar.`
> 
> 2. Describe the NPC's primary `goal` or motivation.
> 	*What do they want to achieve? What drives their actions?*
> 	`To find a worthy successor to take over their adventuring legacy before it is too late.`
> 
> 3. Describe the NPC's primary `quirk` or memorable trait.
> 	*What is a trait with this NPC they can be remembered for that you can use in roleplay?*
> 	`e.g.; always polishing the same mug they have with them in their backpack.`

### Backstory

**Backstory**
 `INPUT[textArea(placeholder(The defining events in the NPCs life)):data.npc.backstory]`

---


### Relationships
Relationships
*The NPC's personal web of allies, enemies, and family.*

---

```dataviewjs
var parents = dv.current().rel.family.parents ?? [];
var partner = dv.current().rel.family.partner ?? [];
var children = dv.current().rel.family.children ?? [];
var siblings = dv.current().rel.family.siblings ?? [];
var allies = dv.current().rel.allies ?? [];
var enemies = dv.current().rel.enemies ?? [];
var current = dv.current().file.name;

// Logic: If true, the Partner also points to the Children node
var isJointFamily = dv.current().rel.family.co_parent ?? false;

dv.paragraph("```mermaid\nflowchart LR\n" +
  // --- PARENTS ---
  (parents.length > 0 ? parents.map((parent, index) => `P${index + 1}[${parent}]:::internal-link\nP${index + 1} --> Current\n`).join('') : '') +
  
  // --- CURRENT NODE ---
  `Current[${current}]\n` +
  
  // --- PARTNER & CHILDREN LOGIC ---
  (function() {
    let mermaid = "";
    
    // 1. Draw the Partners
    if (partner.length > 0) {
        // If joint family, connect with a solid line (---) to show marriage/equal status
        // If not joint, use arrow (-->) to show subordinate/separate relationship
        let linkType = isJointFamily ? "---" : "-->";
        
        // Loop through partners (usually just one)
        mermaid += partner.map((p, index) => {
            // Create the Partner Node
            let pNode = `PT${index + 1}[${p}]:::internal-link`;
            // Link Current to Partner
            return `${pNode}\nCurrent ${linkType} PT${index + 1}\n`;
        }).join('');
    }

    // 2. Draw the Children
    if (children.length > 0) {
        // Create the Main Children Group Node
        mermaid += `C[Children]\n`;
        
        // Connect Current NPC to Children Group
        mermaid += `Current --> C\n`;
        
        // IF JOINT: Connect The Partner(s) to the Children Group too!
        if (isJointFamily && partner.length > 0) {
             mermaid += partner.map((_, index) => `PT${index + 1} --> C\n`).join('');
        }

        // Draw individual children coming out of the Group Node
        mermaid += children.map((child, index) => `C${index + 1}[${child}]:::internal-link\nC --> C${index + 1}\n`).join('');
    }
    
    return mermaid;
  })() +

  // --- SIBLINGS ---
  (siblings.length > 0 ? `S[Siblings]\nCurrent --> S\n${siblings.map((sibling, index) => `S${index + 1}[${sibling}]:::internal-link\nS --> S${index + 1}\n`).join('')}` : '') +

  // --- ENEMIES ---
  (enemies.length > 0 ? `E[Enemies]\nCurrent --> E\n${enemies.map((enemy, index) => `E${index + 1}[${enemy}]:::internal-link\nE --> E${index + 1}\n`).join('')}` : '') +

  // --- ALLIES ---
  (allies.length > 0 ? `A[Allies]\nCurrent --> A\n${allies.map((ally, index) => `A${index + 1}[${ally}]:::internal-link\nA --> A${index + 1}\n`).join('')}` : '') +

  // --- STYLING ---
  `class ${parents.length > 0 ? parents.map((_, index) => `P${index + 1},`).join('') : ''}Current${children.length > 0 ? children.map((_, index) => `C${index + 1},`).join('') : ''}${siblings.length > 0 ? siblings.map((_, index) => `S${index + 1},`).join('') : ''}${enemies.length > 0 ? enemies.map((_, index) => `E${index + 1},`).join('') : ''}${allies.length > 0 ? allies.map((_, index) => `A${index + 1},`).join('') : ''} internal-link;`
)
```

---
> [!NOTE]- Relationship Config
> | Parents | Partner | Children | 
> | --- | --- | --- | 
> | `INPUT[list:rel.family.parents]` |  **Co-parent?**: `INPUT[toggle:rel.family.co_parent]` `INPUT[list:rel.family.partner]`  | `INPUT[list:rel.family.children]` 
>
> | Siblings | Allies | Enemies |
> | --- | --- | --- |
> | `INPUT[list:rel.family.siblings]` | `INPUT[list:rel.allies]` | `INPUT[list:rel.enemies]` |
*If the partner is a co-parent of the children, toggle this. If not, leave it as is.*
**Co-parent**: `INPUT[toggle:rel.family.co_parent]`  

---

### Connections
`BUTTON[button_settlement]` `BUTTON[button_location]` `BUTTON[button_ex_site]`
	
`BUTTON[button_quest]` `BUTTON[button_faction]` `BUTTON[button_item]` `BUTTON[button_lore]`

> [!multi-column|no-title]
> >  **Settlement**
> > `INPUT[inlineListSuggester(optionQuery(#settlement)):rel.settlements]`
> > 
> > **Locations**
> > Parent: `INPUT[suggester(optionQuery(#parent)):rel.parent]`
> > `INPUT[inlineListSuggester(optionQuery(#location)):rel.locations]`
> > 
> > **Explorable Sites**
> > `INPUT[inlineListSuggester(optionQuery(#ex_site)):rel.ex_sites]`
> > 
> 
> >**Quests**
> > `INPUT[inlineListSuggester(optionQuery(#quest)):rel.quests]`
>>
> > **Factions**
> > `INPUT[inlineListSuggester(optionQuery(#faction)):rel.factions]`
> > 
> > **Items**
> > `INPUT[inlineListSuggester(optionQuery(#item)):rel.items]`
> >
> > **Lore**
> > `INPUT[inlineListSuggester(optionQuery(#lore)):rel.lore]`

### Inventory
%% NOT ADDED %%
The following items belong to `= this.file.name`.

```dataviewjs
// This dataviewjs code grabs a random item(s) from the folder below. You can remove this if that's not useful. It's an example of what's possible. 

// THE CODE BELOW IS AI GENERATED AND SHOULD NOT BE CHANGED. 

(async () => {
  let pages = dv.pages('"z/CLI/items"').values;

  // shuffle array
  for (let i = pages.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [pages[i], pages[j]] = [pages[j], pages[i]];
  }

  let pick = pages.slice(0, 2);
  const rows = [];

  // fallback prices based on rarity
  const rarityPrices = {
    "common": "50 gp",
    "uncommon": "100 gp",
    "rare": "500 gp",
    "very rare": "5,000 gp",
    "legendary": "50,000 gp",
    "wondrous, needs attunement": "Varies",
    "—": "—"
  };

  for (const p of pick) {
    // human-friendly display name
    let displayName =
      p.aliases?.[0] ||
      p.file.name.replace(/[-_]/g, " ").replace(/\b\w/g, c => c.toUpperCase());

    // default rarity (prefer frontmatter if present)
    let rarity = p.rarity ?? "—";
    let cost = "—"; // default if not found

    try {
      const fileContent = await dv.io.load(p.file.path);
      if (typeof fileContent === "string") {
        const lines = fileContent.split(/\r?\n/);

        // Extract rarity (next line after first # heading)
        const headingIdx = lines.findIndex(line => /^#\s+/.test(line));
        if (headingIdx !== -1) {
          let nextIdx = headingIdx + 1;
          while (nextIdx < lines.length && lines[nextIdx].trim() === "") nextIdx++;
          if (nextIdx < lines.length) {
            let candidate = lines[nextIdx].trim();
            candidate = candidate.replace(/^>\s*/, "").replace(/(^\*\*|^\*|^_+|^`+)|(\*\*$|\*$|_+$|`+$)/g, "").trim();
            if (candidate) rarity = candidate;
          }
        }

        // Extract cost by searching the entire content for something like "123 gp"
        const costMatch = fileContent.match(/(\d{1,3}(?:[,\d]*)(?:\.\d+)?\s*gp)/i);
        if (costMatch) {
          cost = costMatch[1].trim();
        } else {
          // fallback based on rarity
          cost = rarityPrices[rarity.toLowerCase()] ?? "—";
        }
      }
    } catch (err) {
      // fallback based on rarity if file load fails
      cost = rarityPrices[rarity.toLowerCase()] ?? "—";
    }

    rows.push([
      dv.fileLink(p.file.path, false, displayName),
      cost,
      rarity
    ]);
  }

  dv.table(["Random Item", "Cost", "Notes"], rows);
})(
);

```


