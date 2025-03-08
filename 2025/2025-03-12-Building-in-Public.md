---
title: "Building in Public: Managing Side Projects During Job Transitions"
series: CTO At Small Scale
excerpt: "Career transitions offer a unique opportunity for hands-on technical exploration. After leaving my CTO role, I'm keeping skills sharp by building my Hero Wars Helper project in public. This post covers my strategies for maintaining momentum, prioritizing MVP over perfection, and using side projects to demonstrate ongoing technical relevance — all while navigating the job search process. For developers in transition, here's how to turn uncertainty into productive coding time."
tags:
  - herowars
  - ctointransition
  - sideprojects
date: 2025-03-12
---

# Building in Public: Managing Side Projects During Job Transitions

Career transitions, whether planned or unexpected, create a unique window of opportunity for developers and tech leaders. As I navigate my own transition after leaving my CTO position at NBI, I'm channeling my energy into side projects—specifically my [Hero Wars: Alliance Helper](https://herowars.rovani.net)—to maintain technical momentum, explore new technologies, and demonstrate ongoing growth. This is in addition to spending more quality time with my family, searching for another employer, and training for another marathon.

This post outlines my approach to building in public during this transition period, with strategies and insights that may help other developers in similar situations.

## Why Build in Public During Career Transitions?

Building in public during a career transition offers several advantages. The gap between roles isn't just empty space; it's fertile ground for growth and exploration. Throughout my career, I've observed that the most successful technologists leverage transitions as catalysts for development rather than experiencing them as panic-inducing interruptions.

**Skills continuity** ensures your technical edge remains sharp. Our industry evolves at breakneck speed, and even a few months away from hands-on development can create surprising knowledge gaps. I've seen too many senior leaders struggle to re-enter technical conversations after prolonged absences from the IDE. By maintaining an active coding practice, you're sending a clear signal to potential employers that your skills remain relevant.

**Exploration freedom** represents an opportunity rarely available within the constraints of a full-time role. When leading engineering teams, I often found myself wishing for space to properly evaluate emerging technologies before committing to their adoption. This transition period grants the license to experiment without the pressure of production deadlines or existing technical debt.

The **portfolio enhancement** aspect cannot be overstated. Having recent, relevant work to discuss provides tangible evidence of capabilities during interviews. I advise my mentees to always maintain something fresh in their portfolio that reflects their current skills and interests.

**Community connection** and **accountability** provide structure during an otherwise unstructured time. When you commit publicly to building something, you create external motivation to follow through. I've found that maintaining visibility in technical communities consistently yields unexpected benefits while self-imposed deadlines rarely have the same power as those shared with an audience.

## Focus on Progress Over Perfection

One of the key challenges for experienced developers and tech leaders working on side projects is the tendency to over-engineer or pursue perfection. This mindset shift has been one of the most challenging aspects of my transition from CTO to independent builder. In leadership roles, we often emphasize quality and thoughtful architecture—but I've had to consciously recalibrate my approach for this season.

During this transition period, I'm deliberately focusing on:

**Shipping MVP features first** means embracing a certain level of discomfort with what is released. When I pushed the first version of my Helper, it lacked many features I considered "essential." I haven't been brave enough just yet to post about it anywhere public, but when I do, I look forward to the feedback on these initial features to validate (or refute) my vision. Remember: perfect is the enemy of shipped, and shipped is the prerequisite for feedback.

**Setting clear iteration boundaries** requires disciplined self-management. Moving forward, I will define explicit acceptance criteria for each feature before coding begins. For example, the "collection management" feature needs to support basic CRUD operations and permission controls—but I am deliberately excluding audit logging and analytics for the initial release. By documenting these boundaries upfront, I give myself permission to call a feature "complete" without the nagging feeling of unfinished work.

**Creating public deadlines** transforms productivity during transitions. When I announce that a feature will be available by a specific date, I'm far more likely to focus my efforts appropriately. These self-imposed but publicly declared milestones create the external accountability structure that I find myself missing, having left the organizational environments.

**Documenting known limitations** transparently has been liberating. Rather than hiding imperfections, I call them out explicitly in release notes. This practice manages user expectations, demonstrates awareness of the technical landscape, and creates natural segues for future improvements.

As a former CTO, the goal isn't to showcase perfect code; it's to demonstrate ongoing learning and execution ability. This transition project proves that I can still build effectively in changing circumstances.

## My Technical Exploration Stack

The Hero Wars: Alliance Helper project serves as my playground for exploring several technologies. Technology selection during a transition project offers a strategic opportunity to position yourself for your next role. I've deliberately chosen a stack that balances proven enterprise adoption with forward-looking innovations.

**[Remix/React Router v7](https://reactrouter.com/)** is "a user-obsessed, standards-focused, multi-strategy router you can deploy anywhere." From my perspective, it represents the evolution of frontend architecture toward more seamless data flow patterns. The skill development here isn't just about syntax—it's about internalizing mental models that will likely influence frontend development for years to come.

**[Tailwind](https://tailwindcss.com/) & [shadcn](https://ui.shadcn.com/)** provide a compelling solution to UI development efficiency challenges. These tools enforce consistency while reducing cognitive overhead—a balance I've rarely seen achieved with other approaches. For leaders looking to accelerate frontend development, understanding these patterns firsthand is invaluable.

**[TypeScript](https://www.typescriptlang.org/)** continues to be a non-negotiable foundation for any project I build. The productivity gains and error reduction I've witnessed across numerous enterprise projects have convinced me of its value. As a leader who has inherited many codebases, I consider strong typing an act of professional courtesy that dramatically improves maintainability.

**[TypeORM](https://typeorm.io/)**, **[Supabase](https://supabase.com/)**, and **[Netlify](https://www.netlify.com/)** round out the stack with implementations of patterns I believe will dominate the next wave of web development: type-safe database interactions, backend-as-a-service for common functionality, and simplified deployment that reduces DevOps overhead.

This stack reflects both current industry trends and technologies I'm personally interested in deepening my knowledge of. Each piece addresses specific aspects of modern web development that remain relevant across many potential future roles. And best of all, for side projects, the cost of entry is only my time—all platforms have free tiers to get started!

## Establishing a Sustainable Rhythm

The absence of external structure during a transition can be disorienting, even for seasoned leaders accustomed to managing their own time. I've discovered that recreating structured productivity patterns is essential for maintaining momentum.

**Time-boxed development sessions** have become the cornerstone of my productivity approach. I schedule specific 2-3 hour blocks dedicated exclusively to coding. This approach leverages the focus benefits of timeboxing that I've previously applied to team sprints, but at an individual scale. The defined start and end times create clear boundaries that prevent both procrastination and burnout. I really like using a non-computer visual timer for this, like the [Time Timer](https://www.timetimer.com/pages/time-timer-at-home).

**Weekly planning cycles** provide the strategic direction that daily work needs. Each Monday morning, I conduct a personal retrospective and planning session—a practice adapted from the team rituals I led as CTO. This regular cadence creates a natural accountability loop and prevents the project from drifting off course.

**Public check-ins** serve as external commitment devices that maintain forward momentum. By sharing regular updates, I create soft deadlines for myself while also building an audience interested in my work. I consider these check-ins as my virtual stand-up meetings during a period without teams.

**Balancing job search activities** with project work requires intentional time management. I've found that dedicating mornings to chores and job search activities, while keeping afternoons to development creates a natural rhythm that addresses both priorities and prevents job search anxiety from contaminating the joy of building.

**Feature-based milestones** translate ambiguous project goals into concrete deliverables. Rather than measuring progress by time spent, I focus on completing functional features that deliver actual value—a project management approach that mirrors how I've led engineering teams by prioritizing outcomes over activities.

## Documentation as You Go

Documentation has always separated good engineers from great ones. During my leadership career, I've observed that well-documented projects attract more contributors, experience smoother handoffs, and maintain velocity over time. A key part of my "building in public" approach is consistent documentation:

**README updates with each significant feature** ensure that the project remains accessible to newcomers. I treat these updates as first-class deliverables rather than afterthoughts. This discipline reflects my belief that documentation isn't separate from development—it's an integral part of it.

**Blog posts about technical decisions** force me to articulate my reasoning, create artifacts I can reference in job discussions, and potentially attract like-minded professionals. When documenting a significant architectural decision, I approach it as if explaining to a senior peer—with appropriate context, considered alternatives, and clear reasoning.

**Inline code comments** reflect my commitment to creating maintainable systems. By commenting non-obvious decisions and complex logic, I create a more accessible codebase that others can understand and contribute to. This practice also forces me to revisit and sometimes simplify approaches that prove difficult to explain clearly.

**Architecture diagrams** translate complex systems into accessible visual representations. Having led many technical discussions with cross-functional stakeholders, I've seen firsthand how visual documentation bridges communication gaps. They also provide excellent artifacts for portfolio discussions and technical interviews.

For tech leaders especially, demonstrating clear communication about technical choices showcases a valuable skillset to potential employers.

## Setting Boundaries and Expectations

Managing expectations—both your own and others'—becomes crucial when building in public. Throughout my leadership career, I've found that explicit expectation-setting prevents disappointment and builds trust. Building in public creates accountability and it is important to set realistic expectations:

**This is not my full-time job** is a boundary I state explicitly in my project documentation. Progress will happen at a sustainable pace dictated by the realities of my job search and personal responsibilities. This boundary also models healthy work patterns—something I've always advocated for as a leader.

**Experimentation is welcome** signals my openness to changing approaches as I learn. Rather than viewing pivots as failures, I frame them as valuable learning opportunities. This mindset has served me well when leading engineering teams through technology transitions—embracing the iterative nature of knowledge acquisition.

**Not every feature will be perfect** acknowledges that some technical debt is acceptable in service of progress. By explicitly accepting that parts of the implementation will be "good enough for now," I create space for iteration. This pragmatic approach balances quality with forward momentum—a trade-off every technical leader must master.

**Learning is the primary goal** reframes the entire project around personal growth rather than product success. This perspective reduces performance anxiety and creates space for experimentation—a mindset I've encouraged in team members during their professional development.

I've found that transparency about these boundaries actually leads to more meaningful engagement from others following the project.

## The Job Search Connection

Through multiple career transitions, I've observed that candidates who maintain active technical engagement stand out in competitive hiring processes. While my side project isn't directly aimed at landing a new position, it still supports my job search by:

**Demonstrating continued technical relevance** becomes increasingly important at senior levels. Your transition project provides concrete proof that you're not just talking about modern development practices—you're actively implementing them. This ongoing engagement becomes particularly valuable when interviewing for leadership roles that require both strategic vision and technical credibility.

**Providing conversation material** for interviews transforms potentially awkward technical discussions into confident showcases of your current work. I've found that discussing current projects often reveals more about my problem-solving approach and technical philosophy than rehearsed answers about past accomplishments.

**Showcasing problem-solving approaches** happens naturally when sharing your transition project. How you tackle ambiguity, make architecture decisions, and prioritize features reveals your technical judgment in action. By building in public, you create a transparent record that provides deeper insight than any resume bullet point.

**Building relevant experience** with targeted technologies can strategically position you for specific roles. This approach has helped me bridge gaps between roles multiple times throughout my career, making seemingly challenging transitions more feasible.

For fellow tech leaders in transition, side projects can bridge the gap between leadership roles and hands-on technical work that may have become distant.

## Getting Started with Your Own Transition Project

Having guided numerous engineers and technical leaders through career transitions, I've identified several patterns that lead to successful transition projects. If you're in a similar position, consider these steps to launch your own "building in public" project:

**Choose something personally motivating** that you genuinely want to use yourself. The Hero Wars Helper emerged from my own gaming community involvement which created natural motivation that pure resume-building projects rarely generate. Ask yourself: "What tool do I wish existed?" Then build it.

**Select technologies with purpose**, balancing learning objectives with productivity needs. Avoid the trap of choosing tools solely based on popularity, selecting technologies that genuinely interest you while aligning with your career direction. This intentional selection creates a natural story for interviews about your technology decisions.

**Set up public repositories from day one**, even before writing significant code. This immediate visibility creates accountability and establishes the habit of working transparently. Begin with basic documentation outlining your intentions, then commit incrementally.

**Establish minimum viable documentation** templates that make consistent communication easy. Create simple formats for release notes, feature documentation, and decision records. Well-documented projects signal professional maturity to potential employers reviewing your work.

**Announce your intentions** publicly to create external accountability. Share your project goals on social platforms or relevant community forums. This public commitment mechanism has proven remarkably effective for maintaining momentum during transitions.

## Conclusion

Career transitions provide a rare opportunity to reinvest in technical skills while demonstrating ongoing relevance in the job market. By building in public with clear boundaries, realistic expectations, and a focus on progress over perfection, developers at all levels can maintain momentum and showcase their abilities.

I'll be continuing to update my progress on the Hero Wars: Alliance Helper project in the coming weeks. Follow along on GitHub or through future blog posts as I navigate this transition period one commit at a time.

---

*What side projects are you working on during your own career transitions? Share in the comments below, or connect with me to discuss further.*