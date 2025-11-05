---
title: "Shopify Sucks! But Not Really"
excerpt: |
  How many times have you experienced this:

  > "The documentation says it's possible, Stack Overflow says it's not, the sales team promised it to the client, the Solutions Architect mentioned a client he got it kinda working for a few years ago, and the PM says this ticket is due by Friday."

  An opinionated guide to Shopify Plus limitations, as of 2025-Sept.
date: 2025-09-21
tags:
  - shopify
  - talktrack
---

## Top Challenges Facing Shopify Plus Merchants & How We Can Help

_An opinionated, incomplete guide to Shopify Plus limitations (as of Sept 2025) and implementation solutions_

![The documentation says it's possible, Stack Overflow says it's not, the sales team promised it to the client, the Solutions Architect mentioned a client he got it kinda working for a few years ago, and the PM says this ticket is due by Friday.](/images/this-is-fine.webp)

---

## Quick Wins: The "Easy" Problems

_These get talked about everywhere, but they're mostly solved_

## Recently Fixed or Manageable:

- ~~**100 variant limit per product**~~ (expanding to 2,048 in 2025)
- **SEO & URL Structure Rigidity** -> "Get over it". Google knows how to parse a Shopify storefront.
- **Limited native B2B features** -> Shopify Plus B2B tools are rapidly improving; though not viable for complex wholesale operations. Recommend they wait (or really draw out the discovery process😜)
- **Transaction fees** when not using Shopify Payments - this will likely never change. If it's a dealbreaker, then that's that.

## Key Point:

If prospects only bring up these issues, they haven't done their homework. **The real challenges are deeper.**

![What you don't know that you don't know is what you need to know. You know?](/images/unknowns-iceberg.jpg)

---

## **Documentation & Developer Experience**

**🎯 Technical Platform**

**The Problem:**

- Documentation appears comprehensive but lacks critical implementation details
- Missing context on edge cases, limitations, and real-world scenarios
- Developers frequently resort to Stack Overflow, forums, and blog posts to fill gaps
- API versioning creates confusion with outdated examples, **especially with a search result!**

**Why This Persists:**

Shopify prioritizes rapid feature development over comprehensive documentation maintenance. With quarterly API releases, keeping docs current across all use cases is challenging.

**Some Solutions:**

- **Build relationships with Shopify Plus support** for direct technical clarification
- **Maintain internal documentation library** with real-world examples and edge cases
- **Create a Shopify Technical blog series** to spread street cred
- **Create implementation checklists** for common integration patterns
- **Establish testing protocols** to validate integrations across API versions

---

## **Multi-Store Madness**

**🎯 Enterprise Scalability**

> "We need 7 international stores, but we want to manage them like one."

**The Problem:**

- No true multi-store management from single dashboard
- Independent stores require separate management for products, pricing, content
- Data synchronization between international stores is manual or requires expensive third-party solutions
- URL structure limitations force subdomain/separate domain approach vs. subfolder structure

**The Shopify Markets Trap:**

While Shopify Markets promises international expansion simplicity, it comes with significant drawbacks:

- **Limited currency and pricing control** - can't set unique pricing strategies per market
- **Tax complexity** becomes a nightmare with multiple jurisdictions
- **Shipping configuration** doesn't handle complex international logistics well
- **Content localization** is basic compared to dedicated regional stores
- **Performance issues** - single store serving global traffic creates latency
- **SEO limitations** - subfolder structure isn't always possible, hurting regional search rankings

**Why This Persists:**

Shopify's architecture was designed for single-store simplicity. Multi-store functionality would require fundamental platform restructuring. Markets was their attempt to solve this without architectural changes.

**Some Solutions:**

- **Organizations:** Nearly all Shopify Plus merchants should be set-up as an Organization, which allows for multiple stores.
- **PIM/OMS Integration Strategy:** Implement tools like Salsify, Akeneo, Jasper or similar to centrally manage product data
- **Automation Scripts:** Build custom tools for bulk operations across stores
- **SKU-Based Relationships:** Use consistent SKU structures to link products across stores
- **Markets vs. Multi-Store Decision Framework:** Evaluate complexity of pricing, tax, and content needs before choosing approach

**Trade-offs to Consider:**

- Markets = simpler setup, limited flexibility, potential performance issues
- Multi-store = complex management, higher costs, maximum control
- PIM solutions are expensive but often necessary at scale
- 🤔Can an I18N solution work instead?

---

## **API Rate Limit Russian Roulette**

**🎯 Technical Platform**

> "Our integration worked for the last nine months, why did it suddenly break in late November?"

**The Problem:**

- GraphQL query cost calculations are complex and unpredictable
- Standard plans have restrictive API limits (Shopify Plus gets 10x increase)
- REST API is being deprecated, forcing GraphQL adoption
- Integration testing becomes challenging with rate limiting
- **Black Friday traffic spikes** can trigger unexpected rate limiting even for Plus merchants

**Why This Persists:**

Rate limits protect platform stability, but the complexity of GraphQL cost calculations makes them difficult to predict and manage. Holiday traffic creates unpredictable load patterns.

**Some Solutions:**

- **Query Optimization:** 🚨Design efficient GraphQL queries to minimize costs🚨
- **Caching Strategies:** Implement robust caching to reduce API calls
- **Monitoring Tools:** Implement API usage tracking and alerting
- **Holiday Preparation:** Load test integrations before peak seasons
- **Fallback Strategies:** Design graceful degradation when rate limits are hit

**Trade-offs to Consider:**
Over-optimization can lead to complex code that's harder to maintain. Holiday preparation requires significant testing overhead.

---

## **Analytics & Reporting Limitations**

**🎯 Business Intelligence**

**The Problem:**

- 1,000-row limit on reports requires exports for larger datasets
- Basic analytics lack enterprise-level insights (cohort analysis, LTV calculations)
- Attribution tracking struggles with multi-channel campaigns
- Real-time data delays during peak periods (12-24 hours)
- Cannot customize dashboards for specific KPIs without third-party tools
- **GA4 confusion** - many clients are overwhelmed by the transition from Universal Analytics

**Why This Persists:**

Shopify focuses on simplicity over enterprise analytics complexity. Advanced reporting requires significant infrastructure investment. Google's GA4 rollout created industry-wide confusion.

**Some Solutions:**

- **GA4 Alternatives for Confused Clients:**
  - **Triple Whale** - e-commerce focused, easier interface than GA4
  - **Northbeam** - advanced attribution for larger businesses
  - **Klaviyo Analytics** - excellent for email/SMS driven businesses
  - **Glew** - purpose-built for e-commerce reporting
- **Third-Party Analytics Integration:** Implement specialized e-commerce BI tools
- **Custom External Reporting:** Build automated reporting using Shopify's API combined with external data sources
- **Data Warehouse Integration:** Sync Shopify data to tools like BigQuery, Snowflake, or Datalake for advanced analysis
- **GA4 Training & Setup:** Provide proper GA4 configuration and training to bridge the knowledge gap

**Trade-offs to Consider:**
Third-party solutions increase costs and complexity but provide necessary insights for data-driven decision making. Multiple analytics tools can create conflicting data stories.

---

## **App Ecosystem Spiral**

**🎯 Platform Architecture**

> "We need 23 apps to run our store, and they cost more than Shopify"

**The Trap:**

- Core business functions often require paid third-party apps
- App conflicts compound over time and _poorly built apps_ can slow a site
- Subscription costs for multiple apps can exceed platform costs💰
- No control over app quality and ongoing maintenance
- **App integration gaps** - getting apps to talk to each other is often impossible

**Real-World App Landscape:**
**The Good:**

- **Hulk Apps** - decent quality, broad functionality, reasonable pricing
- **Klaviyo** - industry-leading email/SMS platform with robust integrations
- **Algolia** - excellent search initially, but...

**The Problematic:**

- **Algolia scaling issues** - works great until you need complex features, then falls apart quickly
- **Integration nightmares** - product recommendation apps that can't talk to review apps
- **Performance degradation** - multiple apps loading scripts creates cumulative slowdown

**Why This Persists:**
Shopify's app-first approach allows rapid feature expansion but creates dependency on external developers. Apps are built in silos without integration standards.

**Some Solutions:**

- **App Audit Strategies:** Regular review of app necessity and performance impact - a great part of QBR cycle
- **Consolidation Opportunities:** Choose multi-functional apps over single-purpose solutions; or multiple apps from the same dev
- **Integration Testing:** Specifically test app-to-app compatibility during selection
- **Performance Monitoring:** Implement tools to track app impact on site speed
- **Custom Development Assessment:** Evaluate when custom solutions are more cost-effective; factor in maintenance costs
- **Vendor Relationship Management:** Prioritize apps from developers with good support track records

**Trade-offs to Consider:**
Custom development has higher upfront costs but provides more control and potentially better performance. App consolidation may sacrifice best-in-class functionality for simplicity.

---

## **Content Management & Page Building Limitations**

**🎯 Design & User Experience**

![This is a modern CMS, or so they said.](/images/shopify-cms.png)

**The Problem:**

- Basic blogging platform lacks features compared to WordPress or dedicated CMS
- Limited content organization tools (tagging, categorization)
- No advanced SEO settings for individual blog posts
- Limited ability to create complex layouts and content structures

**Why This Persists:**
Shopify prioritizes e-commerce functionality over content management. Adding full CMS capabilities would increase platform complexity and maintenance overhead.

**Some Solutions:**

- **Premium Page Builder Apps:** Implement Shogun, PageFly, or GemPages for advanced page building
- **Blog Enhancement Apps:** Add SEO Blog Optimizer or similar tools for better blog functionality
- **Custom Content Tools:** Develop custom metafield-based content management systems  
  ⚠️This can be very challenging for a Marketing team to maintain⚠️

---

## **Customer Support Escalation Lottery**

**🎯 Operations & Support**

![This image was generated by AI](/images/shopify-ai-support.png)

**The Problem:**

- Technical issues often require multiple escalations to reach qualified support
- Non-Plus merchants experience longer response times and limited technical support
- Support team cannot assist with checkout.liquid or deprecated feature issues
- Knowledge gaps between general support and technical implementation needs

**Why This Persists:**
Shopify's massive scale (4.5+ million stores) makes personalized technical support challenging for all merchants.

**Some Solutions:**

- **Partner Support Relationships:** Build those relationships with the internal Shopify team
- **Be a Billion Dollar Company:** _no really, be too big to be ignored_

**Trade-offs to Consider:**
Partner relationships and internal capabilities require investment but provide more reliable support.

---

## **Theme & Customization Constraints**

**🎯 Design & User Experience**

**The Problem:**

- Dynamic sections limited to homepage on many themes
- Custom fields require apps or complex metafield management
- Limited customization or lots of coding overhead: pick one
- Theme updates for purchased themes can break customizations

**The Discovery Dilemma:**
How do you identify during discovery whether a prospect will:

- Need too much customization for a purchased theme to work
- Want too much customizability to reasonably build a purpose-built theme

**Red Flags in Discovery:**

- "We want it to look exactly like \[completely different platform\]"
- "Can we change the checkout flow?" (spoiler: very limited options)
- "We need custom product configurators" without budget for custom development
- "We want to be able to edit everything ourselves" + complex design requirements

**Why This Persists:**
Shopify prioritizes security and performance over complete customization freedom.

**Some Solutions:**

- **Customization Scope Assessment:** Develop discovery questions that reveal customization expectations early
- **Theme vs. Custom Decision Matrix:** Create clear criteria for purchased vs. custom theme recommendations
- **Client Education:** Show examples of what's possible within theme constraints vs. custom development
- **Custom Theme Development:** Build bespoke themes for clients requiring extensive customization
- **Metafield Strategy:** Implement ‼️structured‼️ approach to custom data management
- **Version Control:** Maintain proper theme versioning and backup strategies (Tricky build system👍)
- **Progressive Enhancement:** Layer customizations that survive theme updates

**Discovery Questions to Ask:**

- "Show me 3 websites you love and want to emulate"
- "What specific functionality does your current site have that you can't lose?"
- "Who will be maintaining the site content day-to-day?"
- "What's your timeline and budget for custom development?"

**Trade-offs to Consider:**
Custom themes require higher initial investment and ongoing maintenance but provide maximum control. Purchased themes limit flexibility but speed deployment and reduce costs.

---

## Turning Problems Into Profits

![](/images/99-problems-agency.png)

### For Sales Teams

- **Set Clear Expectations:** Discuss these limitations during discovery to avoid surprises
- **Position The Value:** Emphasize experience navigating these challenges
- **Competitive Advantage:** Use existing solutions as differentiators against other agencies
- **Discovery Framework:** Use limitation discussions to uncover true project scope and budget needs

### For Account Managers

- **Proactive Communication:** Address potential issues before they impact client operations
- **Solution Roadmaps:** Present clear timelines and resource requirements for complex implementations
- **Ongoing Partnership:** Position ongoing support as essential for Shopify Plus success
- **Client Education:** Help clients understand when they're asking for the impossible vs. expensive

---

## Turning Problems into Resumes

![](/images/99-problems-resume.png)

### For Project Managers

- **Risk Assessment:** Build these challenges into project planning and timelines
- **Resource Allocation:** Ensure adequate technical resources for complex implementations
- **Testing Protocols:** Implement comprehensive testing strategies, especially for app integrations
- **Scope Management:** Use limitation knowledge to prevent scope creep

### For Engineers

- **Best Practices:** Develop standardized approaches to common Shopify Plus challenges
- **Documentation:** Maintain internal knowledge base for implementation patterns
- **Continuous Learning:** Stay current with Shopify's evolving API and platform changes
- **Integration Expertise:** Become the person who knows which apps work well together

---

## The Bottom Line

_Shopify Plus is powerful, but knowledge of its limits is where the competitive advantage plays in_

### Key Takeaways:

- Every limitation = opportunity to demonstrate expertise
- Proactive communication prevents client surprises
- Our solutions differentiate us from other agencies
- Ongoing partnership is essential for Shopify Plus success
- **Discovery is where you win or lose projects** - use limitation knowledge to scope properly

## Action Items:

- Share solutions across all teams
- Build internal knowledge base of app compatibility
- Develop discovery questions that reveal customization scope
- Create decision frameworks for Markets vs. multi-store
- Use in client conversations to build trust and set proper expectations

---

## Questions & Discussion

**What challenges have you encountered that we should add?**

### Discussion Topics:

- Client stories and solutions you've implemented
- New challenges emerging with recent Shopify updates
- Tools and approaches that have worked well for app integration
- Discovery techniques that reveal true customization needs
- Questions about specific limitations or solutions
