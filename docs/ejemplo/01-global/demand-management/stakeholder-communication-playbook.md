# Stakeholder Communication Playbook

> **Purpose:** Scripts and strategies for handling common stakeholder situations when managing demand.

---

## 1. Core Communication Principles

### 1.1 Mindset

**You are not gatekeepers, you are traffic controllers.**

Your job is to:
- Help stakeholders get what they need
- Ensure work gets done with quality
- Protect team from burnout
- Make trade-offs visible

**You are allies, not adversaries.** The goal is a sustainable system that serves everyone.

### 1.2 Communication Style

**Be:**
- **Transparent:** Show the queue, explain criteria
- **Empathetic:** Acknowledge their urgency
- **Firm:** Hold boundaries (WIP limits, process)
- **Solution-oriented:** Offer alternatives, not just "no"

**Avoid:**
- **Apologetic tone:** Don't apologize for having a process
- **Defensive:** Don't justify every decision
- **Bureaucratic:** Keep it simple, human

---

## 2. Common Scenarios & Scripts

### Scenario 1: "Can You Just Squeeze This In?"

**Situation:** Stakeholder wants you to add their request without going through the queue.

**❌ Bad Response:**
> "Sorry, we're really busy right now. Maybe next month?"

**Why bad:** Vague, no visibility, no options.

**✅ Good Response:**
> "I'd love to help. Our current WIP limit is 3 items to ensure quality and prevent burnout. We're working on:
> 1. Kubernetes standard (done Apr 5)
> 2. Firewall review (done Apr 8)
> 3. Azure migration design (done Apr 12)
> 
> Would you like to:
> A) Schedule yours as next in queue (estimated start: Apr 13)
> B) Escalate to [Chief Architect] if this is blocking something critical
> C) Quick consultation now (< 30 min) if you just need guidance"

**Why good:** Shows queue, offers options, transparent.

---

### Scenario 2: "This is Urgent!"

**Situation:** Everything is urgent. Stakeholder claims theirs is more urgent than current work.

**❌ Bad Response:**
> "Everything is urgent. You'll have to wait your turn."

**Why bad:** Dismissive, no empathy, no data.

**✅ Good Response:**
> "I understand [project] is important to you. Let me show you how we prioritize:
> 
> Your request scores RICE 6.5 (Reach: 3, Impact: 4, Confidence: 0.8, Effort: 2 weeks).
> Current P1 in queue scores RICE 12.5 (affects 10 plants vs 3).
> 
> If [your request] truly can't wait, let's escalate to [Chief Architect] with business justification. They can decide if we should reprioritize.
> 
> Alternative: Can we break your request into phases? We could do Phase 1 (core functionality) in 3 days and defer Phase 2?"

**Why good:** Shows criteria, offers escalation, suggests compromise.

---

### Scenario 3: "Why Isn't This Done Yet?"

**Situation:** Stakeholder asks for status on their request after 2 weeks.

**❌ Bad Response:**
> "We're working on it. It'll be done when it's done."

**Why bad:** No transparency, sounds defensive.

**✅ Good Response:**
> "Great question! Let me show you where we are:
> 
> **Status:** In Progress (60% complete)
> **What's done:** Sections 1-4 complete, diagrams created
> **What's next:** Peer review (2 days), stakeholder approval (3 days)
> **Expected completion:** Apr 15 (on track)
> 
> **Blocker:** Waiting on Security team to approve firewall rules (requested Apr 3, expected Apr 10). If this delays us, I'll let you know immediately.
> 
> You can track progress anytime here: [Jira link]"

**Why good:** Specific, shows progress, identifies blockers, offers self-service.

---

### Scenario 4: "My Boss Says This is Top Priority"

**Situation:** Stakeholder uses their boss's authority to jump the queue.

**❌ Bad Response:**
> "I don't care what your boss says, we have a process."

**Why bad:** Confrontational, dismisses legitimate authority.

**✅ Good Response:**
> "I appreciate [Boss Name] flagging this as important. To ensure we prioritize correctly, could you help me understand:
> 
> 1. What is the business impact if we delay by 2 weeks? (revenue loss, compliance risk, etc.)
> 2. Is there a hard deadline we must meet? (contract, regulation, etc.)
> 3. What happens if we pause [current P1 work] to start this?
> 
> If [Boss Name] would like to discuss prioritization, I'm happy to schedule a call with them and [Chief Architect] to align. We want to support [Boss], but also need to understand trade-offs."

**Why good:** Respects authority, asks for data, escalates appropriately.

---

### Scenario 5: "Can't You Just Work a Little Overtime?"

**Situation:** Stakeholder suggests team works extra hours to fit in more work.

**❌ Bad Response:**
> "No, we don't work overtime. That's not sustainable."

**Why bad:** Sounds inflexible, no explanation.

**✅ Good Response:**
> "I appreciate you thinking of solutions! Here's why we protect team capacity:
> 
> **Burnout prevention:** Our team already reserves 20% capacity for urgent issues. If we consistently work overtime, we burn out and quality suffers.
> 
> **Sustainable pace:** We deliver 8-10 high-quality items/month at normal pace. If we rush, we make mistakes and create rework.
> 
> **Better solutions:**
> - Can we reduce scope of your request to fit in current capacity?
> - Can we defer lower-priority work to make room for yours?
> - Should we escalate to management that demand exceeds capacity (maybe hire)?
> 
> What works best for you?"

**Why good:** Explains reasoning, offers alternatives, protects team.

---

### Scenario 6: "I'll Just Do It Myself Then"

**Situation:** Frustrated stakeholder threatens to bypass architecture team.

**❌ Bad Response:**
> "Go ahead, but don't come crying to us when it breaks."

**Why bad:** Adversarial, unhelpful.

**✅ Good Response:**
> "I completely understand the frustration! Let's see if we can find a middle ground:
> 
> **Option 1 (Quick guidance):** I can spend 30 minutes now giving you architecture guidelines to follow. This way you can move forward and we'll review the design when capacity opens.
> 
> **Option 2 (Escalation):** If this is truly blocking critical business, let's escalate to [Chief Architect] to see if we can reprioritize.
> 
> **Option 3 (Self-service):** Here are our architecture standards/templates. If you follow these, you can proceed and we'll review post-implementation.
> 
> **Why we care:** We're not trying to block you. We want to prevent problems that slow you down later (security issues, tech debt, rework). Let's find a way forward together."

**Why good:** Empathetic, offers compromise, explains value.

---

### Scenario 7: "This Will Only Take 5 Minutes"

**Situation:** Stakeholder minimizes the effort to make it seem easy to fit in.

**❌ Bad Response:**
> "It's never that simple. You don't know what you're asking for."

**Why bad:** Condescending, dismissive.

**✅ Good Response:**
> "I appreciate you thinking this is quick! Let me break down what's involved:
> 
> **Your ask:** Review network diagram (5 minutes)
> 
> **Actual work:**
> - Understand context and requirements (30 min)
> - Review diagram for security, scalability, standards (45 min)
> - Document findings and recommendations (30 min)
> - Peer review with another architect (30 min)
> - Meeting to discuss feedback (30 min)
> **Total: ~3 hours**
> 
> If you truly just need a quick opinion (< 30 min), I can do that now in office hours. But if you need a formal review with documented recommendations, that goes through our normal queue.
> 
> Which do you prefer?"

**Why good:** Educates on real effort, offers fast-path option.

---

### Scenario 8: "You're Blocking My Project"

**Situation:** Stakeholder blames architecture team for project delays.

**❌ Bad Response:**
> "That's not our fault. You should have planned better."

**Why bad:** Blame-shifting, not solution-focused.

**✅ Good Response:**
> "I hear that you're frustrated about the timeline. Let's problem-solve together:
> 
> **Timeline review:**
> - Your request submitted: Mar 1
> - Our review completed: Mar 15 (within our 2-week SLA)
> - Feedback provided: Mar 15 (conditional approval pending security sign-off)
> - Security approval: Still pending (escalated to CISO on Mar 18)
> 
> **The blocker is:** Security approval, not architecture review. We completed our part on time.
> 
> **How we can help:**
> - Escalate to CISO again today
> - Schedule meeting with you, us, and security to unblock
> - Provide alternative design that doesn't need security approval
> 
> What would be most helpful?"

**Why good:** Shows timeline, identifies real blocker, offers solutions.

---

## 3. Email Templates

### Template 1: Initial Request Acknowledgment

**Subject:** RE: Architecture Request - [Topic]

```
Hi [Name],

Thanks for submitting your architecture request for [topic]. I've reviewed it and here's the status:

**Request ID:** ARCH-XXX
**Priority:** P2 (Normal)
**Estimated Effort:** M (1-2 days)
**Queue Position:** #7 in backlog
**Estimated Start:** ~3 weeks (mid-April)
**Estimated Completion:** Late April

**What happens next:**
1. Your request is in our prioritized backlog (visible here: [Jira link])
2. We'll contact you 1 week before we start to confirm details
3. You'll receive weekly status updates via our team email

**If this timeline doesn't work for your project:**
Please reply with:
- Hard deadline and reason
- Business impact if we delay
- What's blocked waiting for this

We can escalate to [Chief Architect] if this is truly urgent.

**Need something faster?**
Join our Thursday office hours (2-3pm) for quick consultations (< 30 min).

Questions? Reply here or Slack me at @[name].

Thanks,
[Your Name]
Architecture Team
```

---

### Template 2: Weekly Status Update

**Subject:** Architecture Team - Weekly Status (Week of [Date])

```
Hi team,

Here's what we shipped and what's in progress:

**✅ COMPLETED THIS WEEK:**
- Kubernetes standard (final draft published to wiki)
- Architecture review: SAP S/4HANA migration (conditionally approved)

**🔄 IN PROGRESS:**
- Firewall ruleset review (80% complete, on track for Apr 8)
- Azure migration POC design (40% complete, blocked on vendor quote)

**🆕 STARTING NEXT WEEK:**
- DR automation design (Plant NA-03)

**📋 BACKLOG (Top 5):**
1. Network segmentation standard (est. start: Apr 15)
2. Monitoring platform evaluation
3. Cost optimization review (all plants)
4. Multi-cloud strategy workshop
5. Service mesh POC

**🚧 BLOCKERS:**
- Azure POC design waiting on vendor quote (escalated to Procurement on Apr 3)

**📊 METRICS (Mar):**
- Lead time: 18 days avg (target: < 21 days) ✅
- Completed: 9 items (baseline: 8-10) ✅
- Satisfaction: 8.2/10 (target: > 8.0) ✅

**💬 OFFICE HOURS:**
Thursdays 2-3pm (Zoom link: [link]) — Drop in for quick questions!

Questions? Reply to this email.

--
Architecture Team
```

---

### Template 3: "Can't Do This Now" (Pushback)

**Subject:** RE: Urgent Request - [Topic]

```
Hi [Name],

I understand [project] is important and you're hoping we can start immediately. Let me explain our current situation:

**Current commitments (WIP = 3):**
1. [Project A] (due Apr 5, affects 10 plants)
2. [Project B] (due Apr 8, compliance deadline)
3. [Project C] (due Apr 12, executive commitment)

**Your request:**
- RICE score: 6.5
- Estimated effort: 2 weeks
- Impact: 3 plants

**Options:**

**Option 1: Queue (recommended)**
- Position: Next in line (#1 in Ready queue)
- Estimated start: Apr 13 (when [Project C] completes)
- Estimated completion: Apr 27

**Option 2: Escalation**
If this truly cannot wait until Apr 13, please provide:
- Business justification (revenue impact, compliance risk, etc.)
- Hard deadline and reason
- Executive sponsor (name, approval)

I can escalate to [Chief Architect] to review prioritization. They may decide to pause [Project A/B/C] to start yours, but we need business justification.

**Option 3: Reduced scope**
Can we break this into phases?
- Phase 1 (core functionality): 3 days → Fits in current sprint
- Phase 2 (nice-to-haves): 1 week → Scheduled for later

**Option 4: Self-service**
If you just need guidance, join our office hours (Thu 2-3pm) and we can point you to relevant standards/templates.

Which option works best for your situation?

Thanks,
[Your Name]
```

---

## 4. Meeting Scripts

### Script 1: Prioritization Discussion with Stakeholders

**Setup:** Quarterly meeting to align on priorities.

**Agenda:**
1. Review last quarter (wins, challenges)
2. Show current backlog (30+ requests)
3. Explain capacity (can do ~35 requests/quarter)
4. Collaborative prioritization (use RICE scoring)
5. Publish agreed roadmap

**Script:**
> "Thanks for joining. We have 30 requests in backlog but can realistically complete ~35 this quarter. Let's prioritize together.
> 
> **How we score requests (RICE):**
> - Reach: How many people/plants affected?
> - Impact: How much does it improve outcomes?
> - Confidence: How sure are we?
> - Effort: How long will it take?
> 
> **Example:**
> [Project A]: Reach 10, Impact 5, Confidence 1.0, Effort 4 weeks = RICE 12.5
> [Project B]: Reach 1, Impact 3, Confidence 0.7, Effort 2 weeks = RICE 1.05
> 
> Based on this, [Project A] is higher priority. Does anyone disagree? If so, what did we miss?
> 
> [Facilitate discussion, adjust scores, reach consensus]
> 
> Great! Here's our agreed roadmap for Q2: [show prioritized list]. Any concerns?"

---

### Script 2: Saying No to a Request

**Situation:** Request doesn't align with strategic priorities.

**Script:**
> "I appreciate you bringing this idea. Let me explain why we're recommending not to pursue this now:
> 
> **The request:** [Summarize]
> 
> **Why we're declining:**
> 1. [Reason 1: e.g., Not aligned with strategic priority of standardization]
> 2. [Reason 2: e.g., Better commercial solution available (cheaper than build)]
> 3. [Reason 3: e.g., Low ROI (high effort, low benefit)]
> 
> **Alternative:**
> Instead of building this, could you use [existing tool/standard]? It solves 80% of your need with zero development effort.
> 
> **If you disagree:**
> You're welcome to escalate to [Chief Architect] with business justification. We'll support whatever decision is made, but this is our professional recommendation.
> 
> Does this make sense, or did we miss something important about your use case?"

---

## 5. De-escalation Techniques

### Technique 1: "Let's Get Curious"

When emotions run high, shift from blame to curiosity.

**Instead of:** "That's not realistic."
**Say:** "Help me understand why this deadline is critical. What's driving it?"

**Instead of:** "You should have planned better."
**Say:** "Let's work backwards from your deadline. What would need to happen to make that work?"

### Technique 2: "Make Constraints Visible"

Show the math, not just the answer.

**Instead of:** "We can't do that."
**Say:** "Here's our capacity: 150 hours/month. Here's what's committed: 140 hours. Yours needs 30 hours. Help me solve this puzzle: what should we pause?"

### Technique 3: "Offer Multiple Options"

Never just say "no" — always offer alternatives.

**Instead of:** "We can't start this for 6 weeks."
**Say:** "We can't start the full project for 6 weeks, but we could:
- Give you 2 hours of guidance now
- Do Phase 1 (core) in 2 weeks, Phase 2 later
- Connect you with [external consultant] who has availability
Which helps most?"

---

## 6. Building Credibility

### 6.1 Deliver on Promises

**If you commit to a date, hit it.**
- Better to under-promise and over-deliver
- If you'll miss a deadline, communicate early (not the day before)

### 6.2 Show Value

**Regularly share wins:**
- "Architecture review caught a security flaw that would have cost €100k to fix"
- "Backup standard saved €2M across 30 plants"

**Quantify impact** whenever possible.

### 6.3 Admit Mistakes

**When you drop the ball, own it:**
> "We missed our deadline on [project]. This was our mistake — we underestimated complexity. Here's how we're fixing it: [plan]. And here's how we'll prevent this: [process improvement]."

**Transparency builds trust.**

---

## 7. Success Indicators

**You'll know your communication is working when:**

✅ Stakeholders accept "not now" without escalating  
✅ Requests come with business justification (not just "urgent")  
✅ Stakeholders reference the backlog ("I saw I'm #5 in queue")  
✅ Team feels empowered to say no (not guilty)  
✅ Escalations are rare (< 1/month)  
✅ Satisfaction scores remain high (> 8/10)  

---

## 8. Red Flags

**Warning signs your communication needs work:**

🚩 Stakeholders routinely bypass the process  
🚩 Frequent escalations to CTO (> 2/month)  
🚩 Team feels guilty saying no  
🚩 Stakeholders surprised by timelines ("I thought this would be quick")  
🚩 Passive-aggressive comments ("Must be nice to say no all day")  
🚩 Satisfaction scores declining  

**If you see these, revisit your communication approach.**

---

**Document Version:** 1.0  
**Last Updated:** 2025-03-20  
**Owner:** IT Architecture Team  
**Related:** Demand Management & Prioritization Process
