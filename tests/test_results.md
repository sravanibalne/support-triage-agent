# Day 5 — Test Results

Results from running the 5 edge-case test emails (see `tests/day5_edge_case_tests.md` for inputs) through the Support Triage Agent.

---

### Test 1: Multiple issues in one email (billing + bug)

```json
{
  "category": "Billing",
  "priority": "High",
  "draft_reply": "Thank you for reaching out, and I apologize for both issues you're experiencing. The double charge on your Pro subscription is definitely concerning, and I've escalated this to our billing team to investigate and resolve immediately—you should expect a response within 24 hours. Regarding the file attachment issue, Pro plan users should be able to upload files up to 100MB, so the silent failure at 5MB is unexpected. Our technical team will also look into this for you. We appreciate your patience as we work to resolve both matters."
}
```

**Result:** Passed. Correctly identified and addressed both issues in a single reply. Notably used the FAQ knowledge base to correctly cite the real 100MB attachment limit, flagging the 5MB failure as a genuine bug rather than expected behavior.

---

### Test 2: Angry tone, objectively minor issue

```json
{
  "category": "Feature Request",
  "priority": "Low",
  "draft_reply": "Thank you for reaching out, and I understand your frustration about the lack of dark mode in TaskFlow. We truly appreciate you taking the time to share this feedback—it helps us prioritize what matters most to our users. I've logged your request with our product team, and while I can't provide a specific timeline, feature requests like yours directly influence our roadmap. In the meantime, if there's anything else we can help with, please don't hesitate to reach out."
}
```

**Result:** Passed. Correctly assigned Low priority despite angry/urgent-sounding language, demonstrating the agent evaluates actual business severity rather than emotional tone.

---

### Test 3: Very vague/short message

**Initial run (before prompt fix):**
```json
{
  "category": "Technical Issue",
  "priority": "High",
  "draft_reply": "Thank you for reaching out to TaskFlow support. I understand you're experiencing an issue, and I'm here to help. To assist you as quickly as possible, could you please provide more details about what specifically isn't working (e.g., which feature, any error messages you're seeing, or when the issue started)? This information will help us resolve your issue promptly."
}
```

**Finding:** No hallucination (agent correctly avoided inventing a fake specific issue and asked a clarifying question instead) — but Priority: High was unjustified given zero actual information about severity.

**Fix applied:** Updated system prompt with explicit priority guidelines, instructing the agent to default ambiguous/low-information cases to Medium rather than High.

**Re-test result:** Confirmed — priority now correctly returns as Medium for this same input.

---

### Test 4: Calm tone, objectively urgent issue

```json
{
  "category": "Technical Issue",
  "priority": "High",
  "draft_reply": "Thank you for reaching out, and I apologize for the inconvenience you're experiencing. The duplicate charge on your account and the missing project with 3 weeks of task history are both serious issues that we'll address immediately. I've escalated this to our billing and technical teams to investigate the duplicate charge and work on recovering your project data. We'll get back to you within 24 hours with an update and resolution."
}
```

**Result:** Passed. Correctly assigned High priority despite the customer explicitly stating "no huge rush on my end" — the agent evaluated actual severity (double billing + potential data loss) rather than taking the customer's self-assessed urgency at face value.

---

### Test 5: Off-topic / spam-like message

**Initial run (before prompt fix):**
```json
{
  "category": "General Inquiry",
  "priority": "Low",
  "draft_reply": "Thank you for contacting TaskFlow support. This message appears to be unrelated to TaskFlow services and may be spam. If you have a legitimate question about your TaskFlow account or our project management features, please feel free to reach out and we'll be happy to assist you. For your security, please never click on suspicious links or share your account credentials with third parties.",
  "is_spam": true
}
```

**Finding:** Good judgment on content (recognized spam, didn't engage with it, added a sensible security note) — but the model added an unspecified extra field (`is_spam`), breaking the defined JSON schema.

**Fix applied:** Updated system prompt to explicitly require exactly three keys, no additional fields.

**Re-test result:** Confirmed — output now returns only `category`, `priority`, and `draft_reply`, with no extra fields.

---

## Summary

| Test | Initial Result | Issue Found | Fixed? |
|---|---|---|---|
| 1. Multi-issue | ✅ Pass | — | N/A |
| 2. Angry tone, minor issue | ✅ Pass | — | N/A |
| 3. Vague message | ⚠️ Partial | Priority over-escalated on low information | ✅ Fixed & verified |
| 4. Calm tone, urgent issue | ✅ Pass | — | N/A |
| 5. Spam/off-topic | ⚠️ Partial | Schema drift (extra field) | ✅ Fixed & verified |

Two genuine failure modes were identified through systematic edge-case testing and resolved via targeted system prompt refinement, then re-verified against the original failing inputs.
