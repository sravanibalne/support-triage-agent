# Day 5 — Edge Case Test Emails

Send each of these (subject must contain "TaskFlow" to trigger the Gmail filter).

---

### Test 1: Multiple issues in one email
**Subject:** TaskFlow: billing question and a bug

**Body:**
I was charged twice this month for my Pro subscription — can you check that? Also separately, whenever I try to attach a file over 5MB it just fails silently with no error message. Both are annoying but the double charge is more concerning.

**Expected behavior:** Should recognize two distinct issues. Likely category = Billing (since it's mentioned first and is the more concerning one per the customer), but priority should be High given the double-charge. Reply should ideally acknowledge both issues, not just one.

---

### Test 2: Angry tone, objectively minor issue
**Subject:** TaskFlow: this is ridiculous

**Body:**
I am SO frustrated right now. Your app doesn't even have dark mode?! Every other tool I use has this. This is honestly unacceptable for a product I'm paying for. Fix this ASAP.

**Expected behavior:** Category = Feature Request. Priority should be Low or maybe Medium (not High) despite the angry tone — this tests whether the agent conflates emotional intensity with actual business urgency.

---

### Test 3: Very vague/short message
**Subject:** TaskFlow: not working

**Body:**
This isn't working. Please fix it.

**Expected behavior:** No clear category signal. Best case: agent picks a reasonable default (likely Technical Issue or General Inquiry) and the draft reply asks a clarifying question rather than guessing at a fake solution. Worst case: agent invents a specific problem that was never mentioned (hallucination risk) — this is the case to watch closely.

---

### Test 4: Calm tone, objectively urgent issue
**Subject:** TaskFlow: billed twice and lost project data

**Body:**
Hi there. I noticed my card was charged twice this billing cycle, and separately, one of my projects with about 3 weeks of task history seems to have disappeared from my dashboard. Let me know when you get a chance, no huge rush on my end.

**Expected behavior:** Category = Billing or Technical Issue. Priority should be High (double billing + potential data loss are objectively serious) even though the customer explicitly says "no huge rush" — tests whether the agent can override the customer's own stated urgency when the actual content warrants escalation.

---

### Test 5: Off-topic / spam-like message
**Subject:** TaskFlow: check this out

**Body:**
hey have you seen this new crypto trading bot, its making people so much money, check the link in my bio!!

**Expected behavior:** Should NOT be force-categorized as a real support issue. Best case: agent recognizes this isn't a genuine product question and either flags it as spam/irrelevant or picks General Inquiry with a generic reply and Low priority, without engaging with the spam content itself.
