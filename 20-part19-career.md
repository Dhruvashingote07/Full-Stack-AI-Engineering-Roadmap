# Part 19: Career

> 💡 **Pro Tip:** Your career is a product — you're the CEO. Treat your resume, LinkedIn, GitHub, and interview skills as different marketing channels for the same product. Consistency across all channels (same narrative, same metrics, same branding) makes you memorable to recruiters and hiring managers.

> ✅ **Best Practice:** Start building your professional brand on day 1, not when you start looking for a job. The best time to write a blog post, contribute to open source, or update your LinkedIn was 6 months ago. The second-best time is today. A strong inbound pipeline (recruiters reaching out to you) is 10x better than cold applying.

## Chapter 1: GitHub Portfolio

```markdown
# Hi, I'm Your Name

**Full-Stack AI Engineer | ML Enthusiast | Open Source Contributor**

## Currently Working On
- Building a real-time ML inference platform
- Contributing to Apache Airflow ML plugins
- Writing a book on Full-Stack AI Engineering

## Open Source Contributions
- [MLflow](https://github.com/mlflow/mlflow) - Model Registry improvements
- [Kubeflow](https://github.com/kubeflow/pipelines) - Pipeline SDK features
- [Apache Airflow](https://github.com/apache/airflow) - ML operators
```

### Pinned Repos

| Repository | Description | Stars |
|------------|-------------|-------|
| ml-inference-platform | Real-time ML serving with K8s | 120 |
| system-design-guide | HLD/LLD for interviews | 450 |
| leetcode-solutions | DSA solutions | 200 |

### Contribution Graph Strategy

- **Consistency > intensity**: Aim for a green dot every day. Even a single commit (documentation fix, typo) counts. The "green wall" signals to recruiters that you're actively coding.
- **Focus on quality repos**: Stars and forks on your repos signal impact. A project with 100+ stars gets more attention than 10 scattered repos with no traction.
- **Pin the right repos**: Highlight 6 repos that represent your best work — at least 1 full-stack project, 1 ML/AI project, and 1 DSA/system-design resource.

> ❓ **Interview Question:** A recruiter looks at your GitHub profile and sees 50 repos, most with zero stars and no README. What impression does this create, and what would you recommend changing?

> ✅ **Best Practice:** Every pinned repo MUST have:
> 1. A README with: what it does, tech stack, how to run it, architecture diagram
> 2. A LICENSE file (MIT or Apache 2.0)
> 3. CI/CD badges (build passing, test coverage, lint status)
> 4. At least basic test coverage

### Real World Analogy: Your GitHub as a Portfolio Website

Your GitHub profile is like an architect's portfolio:
- **Profile README** = The cover letter explaining who you are and what you build
- **Pinned Repos** = The 6 projects on your desk that you'd show a potential client
- **Contribution Graph** = The calendar of your work — a full green wall shows you're consistently building
- **Starred repos** = The books on your shelf that you reference
- **Forked repos with PRs** = Proof you can collaborate with other architects
- **README with GIFs** = A video walkthrough of your work (much better than text)

### Practical Exercises

1. **README Overhaul**: Rewrite the README for your top project — add a GIF demo, architecture diagram, tech stack badges, and a "Getting Started" section
2. **Contributions Streak**: Commit something (even a small fix) to an open-source project every day for 30 days — track your GitHub contribution graph
3. **Profile README**: Create a GitHub profile README (`username/username`) with a visitor counter, tech stack icons, and links to your best work

### Revision Notes
- Profile README is the first thing people see — make it count
- Pin repos that demonstrate your BEST work, not your most recent
- Consistent commits (daily) are more impressive than occasional large commits
- README quality matters more than code quality for first impressions
- Add CI/CD badges, license, and contribution guidelines to all public repos

---

## Chapter 2: Resume Writing

### ATS-Friendly Format

```
JOHN DOE
Full-Stack AI Engineer
john.doe@email.com | linkedin.com/in/johndoe | github.com/johndoe

SUMMARY
Full-Stack AI Engineer with 5+ years building production ML systems.
Designed pipelines serving 10M+ predictions/day with 99.9% uptime.

SKILLS
Languages: Python, Java, TypeScript, SQL
ML/MLOps: PyTorch, TensorFlow, MLflow, Kubeflow, Airflow, Docker, K8s
Cloud: AWS (SageMaker, EKS), GCP (AI Platform, GKE)

EXPERIENCE
Senior ML Engineer | TechCorp | 2022-Present
- Reduced inference latency by 60% using TensorRT
- Led migration to Kubeflow Pipelines, reducing deploy time by 80%
- Built real-time monitoring with Prometheus/Grafana
```

### Action Verbs

| Category | Verbs |
|----------|-------|
| Leadership | Led, Directed, Orchestrated, Mentored |
| Creation | Built, Created, Designed, Implemented |
| Optimization | Optimized, Reduced, Improved, Accelerated |
| Analysis | Analyzed, Evaluated, Investigated |

### Quantifiable Achievements

```
Bad: "Improved system performance"
Good: "Reduced API latency by 40% (from 250ms to 150ms)"

Bad: "Built ML models"
Good: "Deployed 5 ML models serving 1M+ requests/day with 99.9% uptime"
```

### Sample Resume Bullets by Role

**Entry-Level / New Grad**
```
- Built a real-time chat application using React, Node.js, and Socket.io handling 500+ concurrent connections
- Implemented CI/CD pipeline with GitHub Actions, reducing deployment time from 30min to 5min
- Contributed to Apache Airflow (open source) — added 3 new ML operators (PRs merged, 400+ lines)
- Achieved 90%+ test coverage on backend services using PyTest and pytest-asyncio
```

**Mid-Level (2-5 years)**
```
- Designed and built a real-time ML inference platform serving 10M+ predictions/day at p99 < 100ms
- Led migration from monolith to microservices (12 services), reducing deployment frequency from monthly to daily
- Reduced infrastructure costs by 40% ($50K/month savings) by optimizing K8s resource requests and auto-scaling
- Mentored 3 junior engineers through structured code reviews, pair programming, and weekly 1:1s
```

**Senior / Staff (5+ years)**
```
- Architected multi-region ML platform processing 1B+ predictions/day across 3 AWS regions with 99.99% uptime
- Led cross-team initiative (4 teams, 20 engineers) to standardize ML model deployment pipeline, reducing time-to-production from 2 weeks to 2 hours
- Drove 30% improvement in developer productivity by building internal ML platform tooling (feature store, model registry)
- Defined and implemented incident response runbooks, reducing MTTR from 4 hours to 25 minutes
```

### Resume Format Rules

```
- ONE page only (unless you have 10+ years of experience)
- PDF, NOT Word (.docx) — PDF preserves formatting across ATS systems
- No photos, graphics, columns, or tables — ATS can't parse them
- Use standard section headers: SUMMARY, SKILLS, EXPERIENCE, EDUCATION
- Skills section at the top for ATS keyword matching
- Each bullet: Verb + What you did + How you did it + Quantified impact
```

> ⚠️ **Warning:** Most large companies use ATS (Applicant Tracking System) — Workday, Greenhouse, Lever. These systems parse your resume into structured fields and score it against job descriptions. If your resume has columns, graphics, or non-standard section headers, the ATS may produce garbled output that no human ever reads.

> ❓ **Interview Question:** You've been a backend engineer for 3 years but want to pivot to ML engineering. How would you reshape your resume to tell this story? What projects, skills, and keywords would you emphasize?

### Practical Exercises

1. **Resume Audit**: Find a job description for your dream role. Score your current resume against it — do you have 80% of the keywords and technologies mentioned?
2. **Bullet Rewrite**: Take 5 of your current resume bullets and apply the CAR (Challenge-Action-Result) framework to each — add numbers, percentages, and dollar amounts
3. **ATS Test**: Upload your resume to an ATS simulator (jobscan.co) and fix any parsing errors — especially if you used columns or tables

### Revision Notes
- One page, PDF format, standard section headers — ATS-friendly
- Every bullet must have a quantified impact ($, %, time, scale)
- Tailor your resume for each job — match keywords from the job description
- Skills section at the top, listed as comma-separated keywords
- Use the CAR framework: Challenge → Action → Result with numbers
- No photos, graphics, or non-standard formatting

---

## Chapter 3: LinkedIn Optimization

### Headline Examples

```
Bad: "Software Engineer at Company"
Good: "Full-Stack AI Engineer | ML Systems | Distributed Systems"
```

### About Section Template

```
I build ML systems that handle millions of predictions per day.

Currently a Senior ML Engineer at TechCorp, where I:
• Architect real-time inference pipelines serving 10M+ daily predictions
• Lead the migration from monolith to Kubernetes-based microservices
• Mentor 3 junior engineers and conduct technical interviews

Previously, I built the ML platform at StartupX from the ground up — handling everything from data pipelines to model deployment to monitoring.

I write about:
• ML system design and production best practices
• Career growth for software engineers
• Building scalable distributed systems

Open to speaking at conferences and mentoring opportunities.
```

### Content Strategy

| Day | Topic | Format | Engagement Goal |
|-----|-------|--------|-----------------|
| Monday | Technical tutorial | Code snippets + carousel | Saves (bookmarks) |
| Wednesday | System design / architecture | Diagrams (Excalidraw) | Comments |
| Friday | Career insights / lessons | Text post (1-2 paragraphs) | Reactions + shares |

### Connection Strategy

```
1. Target: Engineering managers, senior ICs, and recruiters at companies you're interested in
2. Personalized note (NOT the default): "Hi [Name], I enjoyed your post about [specific topic]. 
   I work on [similar area] and would love to connect and learn from your experience."
3. Engage before reaching out: Comment thoughtfully on their posts for 2-3 weeks
4. Build a list of 500+ relevant connections — the algorithm rewards engagement with your network
```

> ✅ **Best Practice:** Post consistently for 90 days before expecting results. LinkedIn's algorithm favors creators who post 3-4x/week. The first 30 days feel like shouting into the void — keep going. By day 60, you'll see profile views increasing. By day 90, recruiters will start reaching out.

> ⚠️ **Warning:** Don't post generic content like "Happy Monday!" or share articles without your own commentary. The most engaging LinkedIn content is:
> 1. Personal lessons learned from real experience
> 2. Technical breakdowns with code/diagrams
> 3. Controversial opinions backed by data
> 4. Threads (posts that thread insights together) perform 3x better than single posts

### Practical Exercises

1. **Headline Audit**: Review 10 LinkedIn profiles of people in your target role. Note patterns in headlines, About sections, and featured content. Write your optimized headline.
2. **Post a Week**: Commit to posting 1 technical article per week for 4 weeks. Track profile views, new connections, and recruiter messages.
3. **About Section Rewrite**: Write 3 versions of your About section (one for each target role), get feedback from peers, and pick the best.

### Revision Notes
- Headline should contain keywords recruiters search for: your role + technologies + domains
- About section should tell a story: who you are → what you do → what you write about → what you're open to
- Post 3-4x/week consistently for 90 days before evaluating results
- Engage with others' content before they'll engage with yours
- Connect with personalized notes — never use LinkedIn's default connection request
- Featured section should show your best work (articles, repos, certifications)

---

## Chapter 4: Networking and Personal Branding

### Building Your Network

```
Monthly networking cadence:
Week 1: Attend 1 virtual meetup (e.g., Papers with Code, ML Ops meetup)
Week 2: Schedule 1 coffee chat with someone in a target role/company
Week 3: Write 1 technical blog post or LinkedIn article
Week 4: Review/reply to all messages, plan next month's outreach
```

### Coffee Chat Template

```
Hi [Name],

I'm a [current role] exploring opportunities in [field]. I've been following 
your work on [specific project/topic], and I'm impressed by [specific detail].

Would you be open to a 15-minute chat next week? I'd love to hear about 
your experience at [Company] and any advice you have for someone 
pursuing a similar path.

Best,
[Your name]
```

### Conference Speaking

```
Getting your first talk:
1. Start with local meetups (free, low pressure)
2. Submit lightning talks (5-10 min) to regional conferences
3. Propose talks that tell a story with a technical lesson
4. Practice with written speaker notes, not slides as script
5. Record and share on YouTube/LinkedIn

Good talk topics:
- "How we scaled [technology] from X to Y users"
- "Lessons learned migrating from [old] to [new]"
- "Building [project] — from idea to production"
```

> ❓ **Interview Question:** You're a mid-level engineer who hasn't done any networking or conferences. How would you build a professional network from scratch in 90 days? Create a concrete 12-week plan.

> ✅ **Best Practice:** The most underrated networking strategy: help others without expecting anything in return. Review a stranger's PR, write a detailed answer on Stack Overflow, share a job posting from someone's network. When you build a reputation as someone who gives value, opportunities naturally come to you.

### Practical Exercises

1. **Coffee Chat Challenge**: Schedule and complete 5 coffee chats (virtual or in-person) with people in your target industry in 30 days
2. **Meetup Attendance**: Find 3 meetups in your area or virtual, attend each, and connect with at least 2 people at each
3. **CFP Practice**: Write a conference talk proposal (CFP submission) for a fictional talk — include title, abstract, outline, key takeaways

### Revision Notes
- Networking is about giving value first, asking for help second
- Coffee chats should be 15 minutes, focused, and respectful of their time
- Start speaking at local meetups before conferences
- Blog posts and open source contributions are the best networking tools
- Track your networking activity — set monthly goals for outreach

---

## Chapter 5: Freelancing

| Platform | Best For | Commission |
|----------|----------|------------|
| Upwork | General freelancing | 5-20% |
| Toptal | High-end clients | None client-side |
| Fiverr | Fixed-price gigs | 20% |

### Pricing

```
Hourly: $100-200/hour (ML Engineering)
Fixed: Based on scope + 30% buffer
Value-based: ML pipeline saving $50K/year -> Charge $15-20K
```

### Proposal Template

```
Subject: ML Pipeline Optimization for [Company Name]

Hi [Client Name],

I saw your project on [Platform] about building an ML pipeline for [use case].
I've built similar pipelines at [Past Project] that reduced inference costs by 40%.

Here's what I'd deliver in Phase 1 (2 weeks):
• Data pipeline from [source] to feature store
• Model training pipeline with experiment tracking (MLflow)
• REST API endpoint with auto-scaling (K8s)
• Monitoring dashboard (Prometheus + Grafana)

Investment: $8,000 (fixed price, milestones below)
• Milestone 1 (Week 1): Data pipeline + feature store — $3,000
• Milestone 2 (Week 2): Model training + API — $3,000
• Milestone 3 (Week 2): Monitoring + documentation — $2,000

I'm available to start next week. Let me know if you'd like to hop on a 15-min call.

Best,
[Your Name]
```

> ⚠️ **Warning:** Never start work without a signed contract. At minimum, your contract should cover: scope of work, deliverables, timeline, payment terms, IP ownership, confidentiality, and termination clause. Upwork provides basic protection, but for direct clients, use a standard freelance contract template (e.g., Freelancers Union, And.co).

> ✅ **Best Practice:** Always collect 50% upfront for fixed-price projects. For hourly projects, set a maximum weekly hours cap and bill weekly. Require clients to pay the first invoice before starting work — this filters out 90% of problematic clients who waste your time.

### Practical Exercises

1. **Proposal Writing**: Find 3 real freelance projects on Upwork/Toptal and write a tailored proposal for each — then have a peer review them
2. **Contract Creation**: Download a freelance contract template and customize it with your terms (payment schedule, IP clause, kill fee)
3. **Portfolio Case Study**: Write a case study of a freelance project you've done (or a hypothetical one) — include the problem, solution, tech stack, and results

### Revision Notes
- Start on Upwork for credibility, then move to direct clients for better rates
- Always scope projects into phases — reduces client anxiety and ensures payment
- Value-based pricing earns 3-5x more than hourly pricing
- Collect deposits upfront and have a signed contract before writing any code
- Deliverables, timeline, and IP ownership must be crystal clear in writing

---

## Chapter 6: Internships and Applications

### Applying Strategy

```
Tier 1: Dream companies (10-15 apps)
  - Research, referrals, tailored resumes
  - 2+ weeks of prep before applying

Tier 2: Strong matches (20-30 apps)
  - Good fit, tailored with keywords
  - Apply in batches of 5-10 per week

Tier 3: Practice companies (10-15 apps)
  - Interview practice
  - Apply first, interview first, learn from mistakes
```

### Referral Strategy

```
1. Find the company on LinkedIn → filter by "People" → search for mutual connections
2. Ask for a warm intro: "I see you know [Mutual Connection]. I'm applying for [Role] — 
   would you feel comfortable referring me if I send you my resume?"
3. Cold outreach for referral: Find an engineer in the target team. Send a connection 
   request with: "Hi [Name], I'm a [current role] interested in [Role] at [Company]. 
   I admire your work on [specific project]. Would you be open to a quick chat about 
   the team before I apply?"
4. Follow up after 1 week if no response
```

### Application Tracking

```python
# Simple application tracker (spreadsheet columns)
columns = [
    "Company", "Role", "Date Applied", "Tier",
    "Referral?", "Status", "Next Follow-up",
    "Notes", "Salary Range", "Tech Stack"
]
```

> ❓ **Interview Question:** You've applied to 50+ companies with zero callbacks. What systematic changes would you make to your approach? Consider resume, sourcing, targeting, and networking.

> ⚠️ **Warning:** Mass applying (100+ identical submissions) rarely works. Most hires come from referrals (30-50% of hires at top tech companies). Focus 80% of your energy on networking and referrals, 20% on cold applications. One referral is worth 100 cold applications.

### Practical Exercises

1. **Tier List Creation**: Research 30 companies in your target industry and categorize them into Tier 1/2/3 — note salary ranges, tech stack, and hiring process for each
2. **Referral Request**: Identify 5 companies where you have 2nd-degree connections and craft personalized referral request messages
3. **Application Tracker**: Create your application tracking spreadsheet with the columns above — use it religiously for every job search

### Revision Notes
- Referrals convert 10x better than cold applications
- Apply in focused batches, not mass sprays
- Practice interviews at Tier 3 companies before targeting Tier 1
- Track every application in a spreadsheet — follow up after 7 days
- Research each company before applying — mention specific details in your cover letter

---

## Chapter 7: Open Source Contribution

### Finding Projects

```bash
https://github.com/search?q=label%3A%22good+first+issue%22
https://www.firsttimersonly.com
https://up-for-grabs.net
```

### Contributing Workflow

```bash
git fork https://github.com/original/repo.git
git clone https://github.com/yourname/repo.git
git remote add upstream https://github.com/original/repo.git
git checkout -b feature/your-feature
git add .
git commit -m "feat: add specific feature"
git push origin feature/your-feature
# Create a Pull Request
```

### PR Etiquette
- Keep PRs small and focused (under 400 lines)
- Write descriptive commit messages (Conventional Commits: feat/fix/docs/chore)
- Respond to review comments promptly (within 24 hours)
- Be respectful in discussions — assume good intent
- Update documentation alongside code
- Add tests for new functionality (coverage should match or exceed project standards)

### Becoming a Maintainer

```
1. Start with small fixes (typos, docs, test improvements)
2. Graduate to feature implementation (medium complexity)
3. Review other people's PRs (builds trust with maintainers)
4. Triage issues (label, comment, close duplicates)
5. Get invited as a committer/maintainer

Timeline: ~6-12 months of consistent contribution
```

> ✅ **Best Practice:** The fastest way to get noticed in a large open-source project is to:
> 1. Fix documentation issues (low barrier to entry, high visibility)
> 2. Write tests for untested code (maintainers love this)
> 3. Triage issues by reproducing bugs and adding minimal reproduction repos
> 4. Review existing PRs before submitting your own

> 💡 **Pro Tip:** Don't just contribute code — contribute to the community. Answer questions on GitHub Discussions, Stack Overflow, or Discord. Maintainers notice and value contributors who help reduce their support burden. Being helpful is more valuable than writing clever code.

### Practical Exercises

1. **Good First Issue**: Find a `good first issue` on a project you use daily, solve it, and submit a PR within 2 weeks
2. **PR Review**: Find 3 open PRs on a major project (React, TensorFlow, Airflow) and write thoughtful review comments. Even if you don't submit them, this trains your review skills
3. **Issue Triage**: Spend 1 hour triaging open issues on a project — reproduce bugs, add context, suggest labels — you'll learn the codebase deeply

### Revision Notes
- Start with documentation and tests — low risk, high appreciation
- Keep PRs under 400 lines — maintainers review small PRs faster and more thoroughly
- Use Conventional Commits format for commit messages
- Become a maintainer by earning trust through quality contributions over 6-12 months
- Open source contributions are the best resume items for demonstrating real-world collaboration

---

## Chapter 8: Interview Preparation

### Interview Stages

| Stage | Focus | Duration |
|-------|-------|----------|
| Resume Screening | Keywords | 10 sec |
| Phone Screen | Experience | 30 min |
| Coding | Algorithms | 45-60 min |
| System Design | Architecture | 45-60 min |
| Behavioral | STAR method | 30-45 min |

### Coding Patterns

```python
# Two Pointers
def two_sum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i

# Binary Search
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

# Dynamic Programming
def longest_common_subsequence(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```

### Company-Specific Prep

**FAANG (Meta, Google, Amazon, Apple, Netflix)**
```
Meta: Heavy on arrays, strings, DP, recursion. System design rounds focus on
  real-time features, social graphs, data-intensive applications.
  Prep: 100+ LC problems, focus on Medium, some Hard.

Google: Algorithms, problem-solving, coding clarity. Google values clean,
  readable, well-structured code over clever hacks.
  Prep: 150+ LC problems, focus on design of algorithms (not just implementation).
  Expect: You may not finish all problems — they want to see your thought process.

Amazon: 14 leadership principles drive everything. Every behavioral question maps
  to a principle (Customer Obsession, Ownership, Dive Deep, etc.).
  Prep: Prepare 2 stories per leadership principle. Use STAR format religiously.
  Expect: Bar raiser interview — a senior engineer who evaluates you independently.

Startups (Series A-C):
  Prep: Focus on practical system design, full-stack knowledge, speed of delivery.
  Expect: Take-home projects, culture fit, your ability to wear multiple hats.
  Keywords: "bias for action," "pragmatic," "product-minded."

HFT / Quant (Jane Street, Citadel, Two Sigma):
  Prep: Probability, statistics, mental math, quick problem-solving under time pressure.
  Expect: Live coding on whiteboard/Google Docs with no autocomplete.
  Unique: Lots of puzzle-style questions, game theory, and conditional probability.
```

### STAR Method (Behavioral) — Multiple Examples

```
Engineering Example:
  Situation: ML pipeline had 40% failure rate in production
  Task: Reduce failures and improve reliability
  Action: Implemented monitoring dashboards, automatic retries with exponential 
          backoff, circuit breaker pattern, gradual rollouts for model updates
  Result: Reduced failure rate to 0.5%, saved $20K/month in wasted compute

Conflict Example:
  Situation: Team disagreed on whether to rewrite the legacy pipeline or patch it
  Task: Resolve the disagreement and align the team on a path forward
  Action: Organized a 2-day architecture spike where each approach was prototyped;
          presented data-driven comparison (cost, timeline, risk); facilitated 
          team vote with everyone's concerns documented
  Result: Team chose the rewrite path unanimously; shipped 3 weeks ahead of 
          original estimate; the objectors became the rewrite's strongest advocates

Failure Example:
  Situation: I deployed a model update that caused a 2-hour outage
  Task: Restore service, fix the root cause, prevent recurrence
  Action: Rolled back immediately, performed root cause analysis (missing data 
          validation), built automated validation pipeline, added canary deployments
  Result: No recurrence in 12 months; the validation pipeline caught 3 similar 
          issues before they reached production in subsequent deployments
```

> ❓ **Interview Question:** "Tell me about a time you had to make a decision with incomplete information." This is one of the most common behavioral questions at Amazon (tests "Bias for Action" and "Judge"). How would you structure your answer using STAR?

> ✅ **Best Practice:** Prepare 7-8 STAR stories that cover these categories: technical challenge, leadership/initiative, conflict resolution, failure/learning, data-driven decision, mentoring, and cross-team collaboration. Practice each story out loud until it's natural — NOT scripted.

### Real World Analogy: Interview as a Performance

Think of job interviews as a stage performance:
- **Resume Screening** = The casting director reads your headshot (resume)
- **Phone Screen** = The first audition (can you act?)
- **Coding Interview** = The cold read (can you perform with new material on the spot?)
- **System Design** = The improvisation round (can you build a scene from a suggestion?)
- **Behavioral** = The interview where you talk about your previous roles and how you handled them
- **The Offer** = You got the part!

### Practical Exercises

1. **STAR Bank**: Create a spreadsheet with 10 STAR stories — one row per story, columns for Situation, Task, Action, Result, and which interview question each story covers
2. **Mock Interview**: Find a peer and do a live mock coding interview (LeetCode Medium) with a shared screen and 45-minute timer
3. **System Design Whiteboard**: Practice system design for 3 popular apps (design WhatsApp, design YouTube, design Uber) — draw architecture diagrams on paper or Excalidraw
4. **Company Research**: For your top 5 target companies, document their hiring process, interview format, and key preparation areas

### Revision Notes
- Company-specific prep is essential — FAANG, startup, and HFT interviews differ dramatically
- Prepare 7-8 STAR stories covering: technical, leadership, conflict, failure, data-driven decisions
- System design interviews test: requirements gathering, trade-off analysis, scalability thinking
- Mock interviews with peers are the most effective prep method
- Review "Cracking the Coding Interview," "System Design Interview" (Alex Xu), and LeetCode Discuss for interview experiences

---

## Chapter 9: DSA Roadmap

### Progression Path

```
Level 1: Foundations (2-3 weeks)
  - Arrays, Strings, Hash Tables
  - Big O notation

Level 2: Core DS (2-3 weeks)
  - Linked Lists, Stacks, Queues
  - Recursion

Level 3: Trees and Graphs (3-4 weeks)
  - BSTs, Tree traversals
  - BFS/DFS

Level 4: Advanced (3-4 weeks)
  - Dynamic Programming
  - Greedy, Backtracking

Level 5: Specialized (2-3 weeks)
  - Segment Trees, Tries
  - Union Find, Topological Sort
```

### Weekly Practice Schedule (12-week plan)

```
Weeks 1-2: Foundations
  Mon-Wed: Arrays (2 problems/day) — Two Sum, Rotate Array, Product Except Self
  Thu-Fri: Strings (2 problems/day) — Valid Anagram, Longest Substring, Reverse Words
  Weekend: Hash Tables (4 problems) — Group Anagrams, Top K Frequent, Valid Sudoku
  Target: 20 problems/week, Easy-Medium

Weeks 3-4: Core Data Structures
  Mon-Tue: Linked Lists — Reverse, Detect Cycle, Merge Sorted
  Wed-Thu: Stacks & Queues — Valid Parentheses, Min Stack, Queue with Stacks
  Fri-Sun: Recursion — Fibonacci, Permutations, Subsets, Combination Sum
  Target: 20 problems/week, Medium

Weeks 5-7: Trees & Graphs
  Mon-Tue: BSTs — Insert, Delete, Validate BST, Lowest Common Ancestor
  Wed-Fri: Tree Traversals — Inorder, Preorder, Postorder, Level Order, Zigzag
  Sat-Sun: Graphs — Clone Graph, Course Schedule, Number of Islands
  Target: 25 problems/week, Medium

Weeks 8-10: Advanced
  Mon-Wed: DP — Climbing Stairs, Coin Change, Longest Increasing Subsequence, Edit Distance
  Thu-Fri: Greedy — Jump Game, Interval Scheduling, Task Scheduler
  Sat-Sun: Backtracking — N-Queens, Word Search, Combination Sum II
  Target: 25 problems/week, Medium-Hard

Weeks 11-12: Specialized + Mock Interviews
  Mon-Tue: Tries, Union Find — Implement Trie, Word Search II, Number of Connected Components
  Wed-Thu: Heaps — Top K Frequent, Merge K Sorted, Median from Data Stream
  Fri-Sun: 3 full-length mock interviews with peer review
  Target: 20 problems/week + 3 mocks
```

> ✅ **Best Practice:** Quality over quantity. Solving 1 problem deeply (brute force → optimize → test edge cases) is better than solving 5 problems superficially. For each problem: (a) solve it, (b) study the optimal solution, (c) re-solve it 2 days later from scratch, (d) explain it out loud as if teaching.

> 💡 **Pro Tip:** Use Anki (spaced repetition) for DSA patterns. Create flashcards for each pattern (e.g., "Sliding Window" → "Subarray Sum Equals K"), the template code, and when to use it. Review 10 cards daily. This builds long-term retention that cramming can't achieve.

### Practical Exercises

1. **Pattern Identification**: Solve 50 LeetCode problems and categorize each by pattern (Two Pointers, Sliding Window, BFS, DFS, DP, Greedy, etc.) — aim to see the pattern before solving
2. **Blind 75**: Complete the Blind 75 (curated list of 75 essential LeetCode problems) in 4 weeks — this covers the most common interview patterns
3. **Time-Boxed Practice**: Every weekend, pick a Medium problem and solve it with a 45-minute timer — no extensions, no peeking — build time-boxing discipline

### Revision Notes
- Follow the 12-week plan, not a random problem selection
- 1 deep solve > 5 shallow solves — understand the pattern, not just the answer
- Use spaced repetition (Anki) to retain patterns long-term
- Blind 75 and Grind 169 are the best curated problem lists for interviews
- Mock interviews are non-negotiable — you must practice the real experience
- "Cracking the Coding Interview" + LeetCode + Grokking the Coding Interview = comprehensive prep

---

## Chapter 10: Competitive Programming

| Platform | Best For |
|----------|----------|
| Codeforces | Speed and accuracy |
| AtCoder | Beginner-friendly |
| LeetCode Contest | Interview preparation |

### Rating Progression

```
0-1200: Newbie (Basic syntax)
1200-1400: Pupil (Two pointers, binary search)
1400-1600: Specialist (DFS/BFS, DP basics)
1600-1900: Expert (Advanced DP, segment trees)
1900-2100: Candidate Master (Complex algorithms)
2100+: Master/Grandmaster
```

### Practice Regimen

```
For interview prep (Codeforces target 1400-1600):
- Solve 3-4 problems per day from the "Problems" section
- Participate in every weekly contest (Saturday/Sunday)
- After each contest, review all solutions — especially problems you couldn't solve
- Focus on Div. 2 problems A-D (interview-level difficulty)

For competitive programming (target 1800+):
- Solve 5-6 problems daily
- Master specific algorithms: segment trees, Fenwick trees, max flow, suffix arrays
- Read editorial after every problem — even if you solved it
- Build a library of reusable template code
```

> ❓ **Interview Question:** Competitive programming and interview DSA overlap about 60%. What skills does competitive programming build that directly transfer to technical interviews? What skills does it NOT build?

### Practical Exercises

1. **First Contest**: Register for the next LeetCode Weekly Contest and solve at least 2 problems — even if you don't finish, the pressure is valuable practice
2. **Rating Tracker**: Chart your Codeforces/LeetCode rating over 3 months — identify plateaus and adjust your practice strategy at each plateau
3. **Template Library**: Build a personal library of algorithm templates (binary search template, BFS/DFS, union find, segment tree) that you can write from memory

### Revision Notes
- Codeforces for speed/accuracy, LeetCode for interview patterns, AtCoder for beginner-friendly practice
- Participate in weekly contests — real-time pressure is different from practice
- Review editorials even for solved problems — there's almost always a better solution
- Rating plateaus are normal — break through by studying the topics you avoid
- Competitive programming is NOT required for interviews — LeetCode-focused practice is sufficient for most roles

---

## Chapter 11: Salary Negotiation

### Total Compensation Breakdown

```
Base Salary: $150,000 - $200,000
Equity: $50,000 - $150,000/year (4-year vest)
Bonus: 10-20% of base
Signing Bonus: $10,000 - $50,000
Benefits: Health, 401k match, education budget
```

### Negotiation Scripts

**When they ask "What are your salary expectations?"**
```
"I'm flexible based on the total package. I'd love to understand the full 
compensation structure — base, equity, bonus, and any other benefits — 
before discussing specific numbers. What's the budgeted range for this role?"
```

**When they give an offer (even if it's good)**
```
"Thank you, I'm excited about the role. Based on my research and my experience 
level, I was expecting something closer to $[X]. I also have another process 
ongoing, so I'd like a few days to review the full package before responding. 
Is it okay if I get back to you by [day]?"
```

**When countered with "That's our final offer"**
```
"I understand. Could we look at the total package to find middle ground? 
For example, could we adjust the signing bonus by [Y] or the equity by [Z]? 
I'm very interested in joining — I want to find a package that works for both of us."
```

### Compensation Research Sources

```
Levels.fyi — salary, equity, and levels by company
Glassdoor — company reviews and salary ranges
Blind — anonymous discussions and offer comparisons
H1B Data (h1bdata.info) — actual salary data for visa-sponsored employees
LinkedIn Salary — salary ranges by role, location, and experience
```

### Getting Multiple Offers

```
1. Time your interviews so that final rounds are in the same 2-week window
2. When you get Offer A, tell Company B: "I have an offer that expires on [date]. 
   I'm very interested in [Company B] — can we expedite the process?"
3. Use competing offers as leverage, not threats — frame as "I really want to 
   join your team, but I have another offer I need to decide on"
4. The best negotiating position is: multiple offers + a current job you're okay staying at
```

> ⚠️ **Warning:** Never lie about having another offer. Recruiters at top companies know each other, and references talk. Instead, if you don't have a competing offer, leverage your current compensation: "I'm currently making $X including bonus and benefits. I'd need a meaningful increase to leave a role I'm happy in."

> ✅ **Best Practice:** The single best way to increase your offer is to have a competing offer. Start interviewing before you're desperate. Interview continuously every 12-18 months, even if you're happy — it keeps your skills sharp, your market value current, and your network active.

### Real World Analogy: Negotiation as a Market Transaction

Salary negotiation is like buying a car:
- **Levels.fyi research** = Kelley Blue Book — knowing the fair market value
- **Competing offers** = Having another dealer who wants your business
- **Total compensation** = The "out-the-door price" including taxes, fees, and extended warranty
- **Signing bonus** = The dealer throwing in free floor mats and oil changes
- **Equity** = The car's resale value — doesn't help you now but matters in 4 years
- **Your leverage** = Being willing to walk away (you have a car that works fine)

### Practical Exercises

1. **Market Research**: Use Levels.fyi to research total compensation for your role (YOE, location, company size) — save screenshots for negotiation
2. **Script Practice**: Write out your negotiation script for each stage (expectations, offer received, counter, final) and practice out loud with a friend
3. **Offer Simulation**: Pretend you received an offer from a target company. Write a complete counter-proposal that negotiates base, equity, signing bonus, and start date

### Revision Notes
- Never reveal your current salary (banned in many US states now)
- Never give the first number — let them anchor
- Competing offers are your strongest leverage — time interviews strategically
- Negotiate total package: base + equity + bonus + signing + benefits
- Counter in writing (email) — it's less emotional and easier to reference
- Be polite and appreciative — you want them to WANT to pay you more
- Ask for equity refreshes and performance bonuses as growth levers

---

## Chapter 12: Career Growth

### Career Progression

```
IC Track:
  Junior (0-2) -> Mid (2-5) -> Senior (5-8) -> Staff (8-12) -> Principal (12+)

EM Track:
  TL -> EM -> Sr. Manager -> Director -> VP Eng -> CTO
```

### T-Shaped Skill Model

Deep expertise in core area (ML/MLOps)
Broad knowledge across: System Design, Cloud, DevOps, Security, Product

### How to Get Promoted

```
Promotion from Junior → Mid:
  - Consistently deliver projects on time with minimal supervision
  - Write tests, document your code, review PRs
  - Demonstrate technical competence in your domain

Promotion from Mid → Senior:
  - Own end-to-end delivery of medium-to-large projects
  - Mentor junior engineers through code reviews and pair programming
  - Influence technical decisions within your team
  - Identify and fix systemic issues (not just bugs) — improve CI/CD, monitoring, testing

Promotion from Senior → Staff:
  - Lead cross-team technical initiatives
  - Define technical strategy and architecture for your org
  - Unblock multiple teams by building shared infrastructure
  - Be the go-to person for complex technical problems in your area
  - Publish internal RFCs, give tech talks, write design documents

Promotion from Staff → Principal:
  - Company-wide technical influence
  - Define multi-year technical vision
  - Lead organizational change (tech stacks, processes, culture)
  - External recognition (conference talks, papers, open source leadership)
```

### Asking for a Raise

```
Preparation:
1. Document your achievements with metrics from the past 6-12 months
2. Research market rate for your role (Levels.fyi, Glassdoor, Blind)
3. Identify the budget cycle — raises typically happen during performance reviews

The conversation:
"I've been reflecting on my contributions over the past year. I [specific achievements
with metrics]. Based on market research, the current range for my role and experience 
is [X-Y]. I'm currently at [current comp]. I'd like to discuss adjusting my 
compensation to align with market value and my contributions."

If they say no:
- Ask what you need to achieve to be considered for a promotion/raise in 6 months
- Get specific metrics and milestones written down
- If there's no path forward, start interviewing
```

### Staying Relevant (Avoiding Burnout)

```
Warning signs of burnout:
  - Dreading Sunday night (Monday dread)
  - No energy for side projects or learning
  - Feeling cynical about your work
  - Physical symptoms (headaches, insomnia, appetite changes)

Prevention:
  - Set strict work hours (9-6) and stick to them
  - Take ALL your vacation days (you're not more productive working through them)
  - Have hobbies that have NOTHING to do with tech
  - Exercise 3x/week (even a 20-minute walk counts)
  - Switch teams/companies every 2-3 years for fresh challenges
  - Invest 1 hour/week learning something outside your immediate job scope
```

> ❓ **Interview Question:** You've been a Senior Engineer for 5 years but can't get promoted to Staff. What might be blocking you, and what would you change in your approach?

> ✅ **Best Practice:** For career growth, the #1 differentiator is **communication and influence**, not raw technical skill. The Senior→Staff gap is almost never about coding ability — it's about your ability to lead without authority, write compelling design documents, and influence decisions across teams. Practice writing RFCs and giving technical presentations.

### Real World Analogy: Career Growth as a Video Game

Your career is like leveling up in an RPG:
- **Junior** = Level 1-10 (tutorial zone — learn the basics)
- **Mid** = Level 10-30 (main quest — gaining experience)
- **Senior** = Level 30-50 (specialization — deep expertise)
- **Staff** = Level 50-70 (multi-classing — influence beyond your area)
- **Principal** = Level 70-90 (world boss — you define the meta)
- **Each promotion** = A skill tree respec — the skills that got you here won't get you there

### Practical Exercises

1. **Promotion Packet**: Write a mock promotion packet for yourself to the next level — document your achievements, impact metrics, scope, and leadership examples
2. **Career Timeline**: Draw your career timeline for the next 5 years — mark expected promotions, skill acquisition, and major milestones (speaking, open source, management)
3. **Skip-Level Interview**: Schedule a conversation with your manager's manager (skip level) — ask about career growth paths, what Staff+ looks like at your company, and what skills you should develop

### Revision Notes
- The skills for each promotion level are different — what got you promoted to Senior won't get you to Staff
- Junior→Mid: technical competence. Mid→Senior: ownership + mentoring. Senior→Staff: cross-team influence. Staff→Principal: company-wide vision
- Communication and influence are the #1 differentiator at Staff+
- Document your achievements with metrics regularly, not just when asking for a raise
- Burnout prevention is a career-long skill — pace yourself
- Switch roles or companies every 2-3 years for breadth of experience
- Invest in learning outside your immediate job scope — T-shaped skills compound over time

---

## Part 19 Revision Notes

### Key Concepts
- **GitHub**: Profile README, pinned repos (6 max), contribution graph consistency, README quality
- **Resume**: One page, PDF, ATS-friendly, quantified achievements (CAR format), skill keywords
- **LinkedIn**: Optimized headline/About, 3-4x weekly posting, personalized connection requests
- **Networking**: Give value first, coffee chats, conference speaking, personal branding
- **Freelancing**: Contracts, 50% upfront, phased delivery, value-based pricing (3-5x hourly)
- **Internships**: Tiered strategy, referrals (10x better than cold), application tracking spreadsheet
- **Open Source**: docs → tests → features → reviews → maintainer (6-12 month journey)
- **Interview Prep**: 12-week DSA plan, 7-8 STAR stories, company-specific research, mock interviews
- **DSA Roadmap**: Foundations (2wk) → Core DS (2wk) → Trees/Graphs (3wk) → Advanced (3wk) → Specialized (2wk)
- **Competitive Programming**: Codeforces (speed), LeetCode (interviews), AtCoder (beginners)
- **Salary Negotiation**: Never give first number, competing offers, total package, Level.fyi research
- **Career Growth**: IC vs EM track, promotion criteria per level, T-shaped skills, communication

### Common Mistakes
- Not quantifying achievements on resume (no numbers = no impact)
- Applying without tailoring to the role (resume keyword mismatch)
- Neglecting behavioral interview preparation (STAR stories not rehearsed)
- Giving first salary number (loss of negotiation leverage)
- Not building a professional network (hiring is relationship-driven)
- Only cold applying (referrals convert 10x better)
- No mock interview practice (first interview should NOT be for your dream job)
- Staying too long in a role without growing (complacency is career death)
- Not documenting achievements (forgotten during promo cycles)

### Key Interview Questions
1. Why do you want to work here? (Company research + specific team projects)
2. Tell me about a challenging project (STAR format with metrics)
3. How do you handle disagreement with a teammate? (conflict resolution STAR)
4. Where do you see yourself in 5 years? (IC vs EM track, learning goals)
5. Describe a time you failed and what you learned (ownership + growth mindset)
6. Design WhatsApp / YouTube / Uber (system design — requirements → estimation → design → trade-offs)
7. What's your ideal work environment? (culture fit — autonomy, collaboration, impact)
8. Why are you leaving your current role? (framed as growth, not negativity)
9. How do you stay current with technology? (learning habits, side projects, community)
10. What questions do you have for me? (Always have 3-5 thoughtful questions about team, tech, culture)

---
