# The Art of (C)Lean Code in the Era of Vibe-Coding
*How my AI-built spaghetti turned into a clean kitchen (but not without some broken plates)*

![spagetti](https://github.com/cvranjith/cvranjith.github.io/blob/main/_posts/spagetti.png)

---

## The Beginning — When Code Felt Like Handcraft
Not too long ago, coding felt like woodworking.  
You’d measure, cut, sand, polish. Everything took time.

Then came **vibe-coding**.  
Now you just tell your AI what you want —  
*"Build me a dashboard with charts, export to Excel, and a dark mode"* —  
and it does it in minutes.

It’s intoxicating. You feel like a magician.  
But magic always comes with fine print.

---

## My First Vibe-Coding Mess
I was building a tool using vibe-coding.  
I didn’t touch the code much — I just described errors from the logs back to the AI.  
Every time something broke, I’d paste the stack trace, and AI would “fix” it.

It kept working… kind of.  
But inside, the code was becoming **spaghetti** —  
pieces added over pieces, quick fixes on top of quick fixes.  
Some functions were 200 lines long, doing five unrelated things.

Then one day, I decided: *Let’s clean this up.*  
I asked AI to refactor the code to be shorter, modular, and easier to read.

The result?  
Beautiful, lean, elegant code.  
…Except it also broke three important features.

That’s when I learned an important truth:  
**Clean code isn’t just about looking clean — it’s about keeping things working while cleaning.**

---

## The Pitfalls of Vibe-Coding
From that day, I started spotting patterns in what goes wrong when you rely on AI without guardrails:

1. **The Illusion of “It Works”**  
   Just because your app runs doesn’t mean it’s solid.  
   AI can fix one bug while planting seeds for three more.

2. **Invisible Complexity**  
   AI might import six packages for a one-line feature.  
   Like buying a screwdriver and getting a forklift delivered.

3. **Maze Code**  
   Without naming and structure discipline, you get a codebase no one wants to touch.

4. **Overfitting Today’s Problem**  
   AI often solves the exact issue you describe —  
   but the next small change might require rewriting everything.

---

## How I Turned Spaghetti into Clean Pasta
Here’s what I started doing after my “broken features” incident:

### 1. Refactor in Small Steps  
Instead of one big “clean-up”, break it into small safe changes.  
Run tests after each change.

### 2. Skim, Don’t Skip  
Even if you don’t read every line, scan enough to see:
- Hidden dependencies
- Redundant logic
- Odd variable names

### 3. Add Guardrails Early  
Use:
- **Linters** for style
- **Automated tests** for core features
- **Static analysis** to catch silly mistakes

### 4. Save Good Prompts  
If you find a prompt that gives clean, modular code — reuse it.  
Bad prompts often produce messy code.

### 5. Document Decisions  
Write *why* you solved it a certain way.  
Future-you will thank you when debugging at 2 a.m.

---

## A Practical Example — My Spaghetti Incident (Python)

### Messy AI-Generated Python (Evolved Over Time)
```python
import os, sys, json, datetime, time

def processData(stuff, filePath=None):
    try:
        if filePath == None:
            filePath = "out.json"
        res = []
        for i in range(len(stuff)):
            if "date" in stuff[i]:
                try:
                    d = stuff[i]["date"]
                    if d != None and d != "" and not d.startswith(" "):
                        try:
                            stuff[i]["date"] = datetime.datetime.strptime(d, "%Y-%m-%d").strftime("%d/%m/%Y")
                        except Exception as e:
                            print("err date", e)
                    else:
                        pass
                except:
                    pass
            res.append(stuff[i])
        try:
            j = json.dumps(res)
            with open(filePath, "w+") as f:
                f.write(j)
        except:
            print("write err")
    except:
        print("error")
    time.sleep(0.2)
    return "ok"
```

**Why it’s so messy:**
- Multiple unused imports (`os`, `sys`, `time` just to sleep for no reason)
- Deeply nested `try/except` swallowing real issues
- Redundant `None` and empty string checks
- Weird variable names (`stuff`, `i`, `d`, `j`)
- Logic scattered and repetitive
- Random `sleep` at the end, serving no purpose

---

### Clean 4-Liner Version
```python
from datetime import datetime
import json

def process_records(records, output_file="out.json"):
    for r in records: 
        if r.get("date"): r["date"] = datetime.strptime(r["date"], "%Y-%m-%d").strftime("%d/%m/%Y")
    json.dump(records, open(output_file, "w"), indent=2)
```

---

## Another “Seen in the Wild” — JavaScript

### Messy AI-Generated JavaScript (Evolved Over Time)
```javascript
function calcTotals(arr, output){
    var total = 0;
    var t = 0;
    if (!arr) { console.log("no data"); return; }
    for (var i = 0; i < arr.length; i++) {
        var itm = arr[i];
        if (itm.p != null && itm.q != null) {
            try {
                var p = parseInt(itm.p);
                var q = parseInt(itm.q);
                if (!isNaN(p) && !isNaN(q)) {
                    t = p * q;
                    total = total + t;
                } else {
                    console.log("bad data at index", i);
                }
            } catch(e) {
                console.log("err", e)
            }
        } else {
            console.log("missing fields");
        }
    }
    console.log("total is", total);
    if (output) {
        try {
            require('fs').writeFileSync(output, JSON.stringify({total: total}));
        } catch (err) {
            console.log("write fail");
        }
    }
    return total;
}
```

**Why it’s a mess:**
- Variable naming chaos (`total`, `t`, `itm`, `p`, `q`)
- Multiple redundant null and NaN checks
- Logging everywhere instead of proper error handling
- Old-school `for` loop where a `reduce` would be cleaner
- Random optional file-writing mixed into calculation logic
- Overly verbose try/catch for simple operations

---

### Clean 3-Liner Version
```javascript
function calculateTotal(items) {
  return items.reduce((sum, { price, quantity }) => sum + (Number(price) * Number(quantity) || 0), 0);
}
```

---

![spagetti](https://github.com/cvranjith/cvranjith.github.io/blob/main/_posts/refactored.png)

## Red Flags to Watch
- Magic numbers or strings everywhere
- Too many dependencies for small features
- Overcomplicated solutions to simple problems
- Mixing coding styles without reason
- No security checks

---

## The Mindset Shift
Vibe-coding is powerful, but the AI is not your senior architect.  
It’s a fast builder who never sweeps the floor.

Your job?  
Make sure the house it builds won’t collapse when someone slams the door.

---

## The Ending — Clean, Lean, and Battle-Tested
After my spaghetti incident, I learned:
- Code can be both fast to write *and* safe to maintain
- Refactoring is not just “making it pretty” — it’s making it future-proof
- Tests are your safety net when cleaning up AI’s work

Vibe-coding is here to stay.  
But clean, lean code is what will keep you from living in a haunted house.
