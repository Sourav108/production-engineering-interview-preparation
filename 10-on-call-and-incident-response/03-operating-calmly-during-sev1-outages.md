# 03. Operating Calmly Under Sev-1 Outage Pressure

## 1. Problem
Under extreme pressure (hundreds of failing requests, executive scrutiny, and alarm bells ringing), engineers often experience tunnel vision, execute destructive commands without backups, or forget to record timestamps.

## 2. Production Context
Maintaining psychological safety, disciplined communication, and emotional composure is the hallmark of a Senior/Staff Production Engineer.

## 3. Mental Model: The 5 Ground Rules of Sev-1 Incident Execution

1. **One Change at a Time**: Never execute multiple concurrent infrastructure changes simultaneously, or you won't know which one fixed or exacerbated the outage.
2. **Always Have a Rollback Plan**: Before running any emergency command or configuration edit, know how to reverse it in $<30$ seconds.
3. **Capture Diagnostics Before Restarting**: Run `jcmd <pid> Thread.print` or take a quick VM snapshot before terminating a rogue process so forensic evidence is not lost.
4. **Speak in Direct, Declarative Language**: Avoid ambiguous phrases ("Maybe we could try restarting?"). Use clear assignments: *"@sarah, please capture a thread dump on pod-4 and report back in 3 minutes."*
5. **Blameless Culture is Non-Negotiable**: No finger-pointing or assigning personal blame during or after an incident.

---

## 4. Interview Questions & Model Answers

**Q1: Describe a time you led a high-stakes production incident where the initial hypothesis was wrong.**
**Answer Model**:
> *"During a Black Friday traffic surge at my previous company, our checkout service error rate spiked to 12%. The initial hypothesis was database connection pool exhaustion because connection timeouts were appearing in logs. The team proposed doubling the connection pool size. Before approving that change, I asked the operations lead to inspect database CPU and lock metrics: the database CPU was only at 18%, and active connections were well below maximum. I hypothesized that worker threads were hung on an external call before reaching the pool. Distributed traces confirmed that a third-party tax calculation API had degraded, taking 4,000ms per request. Instead of overloading the database, we tripped the tax service circuit breaker, falling back to cached regional tax tables, which immediately dropped checkout errors to 0% in under 90 seconds."*
