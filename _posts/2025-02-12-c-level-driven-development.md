---
layout: post
title: C-level driven development (CDD)
description: "This post discusses some musings I had with ChatGPT that started as a sarcastic joke but actually produced some useful output"
modified: 2025-02-12
category: post
tags: [software, management, team work, AI, shitposting]
share: true
---

# The frustration trigger
A few weeks ago I was quite frustrated by yet another decision seemingly being pushed top down to my team seamingly from the very top. Different people have different mechanisms to cope with frustration and one of those is humor and sarcasm. Nowadays we have (as any other tech company that pushed AI onto its employees as hard as possible) access to ChatGPT, which is a great tool that more often than not gaslights you into believing completely made up stuff. It does that by sounding extremely confident, even when it's completely wrong. There is even a term for it - AI hallucination. Naturally, this flaw makes it a great tool to come up with completely made up terms and concepts that don't exist really. So I decided to ask it about my sarcastic joke methodology I have made up in my head.

# Introducing C-level driven development (CDD)
> Me: Can you explain to me the software development practice commonly knows as c-level driven development?

It took the "bait" and did what it does best - genereated an extremely confident looking reply for this made-up methodology. We all know how verbose ChatGPT can get, so for the sake of usability, I have collapsed all of the very long replies below and left unwrapped only the shorter ones and also my prompts.

---

<details>
<summary>ChatGPT:</summary>

<p>
C-Level Driven Development (CLDD) is an informal and often critical term used to describe a software development approach where priorities, requirements, and decisions are heavily influenced or dictated by executives in the C-suite (e.g., CEO, CFO, CIO, CTO) rather than being guided by customer needs, technical feasibility, or product strategy.
</p>

<h4>Key Characteristics of C-Level Driven Development:</h4>
<ul>
    <li>Top-Down Directives: Projects and features are initiated or prioritized solely because someone in the C-suite believes they are important, often without validation from users or market research.</li>
    <li>Short-Term Focus: There may be a tendency to prioritize initiatives that promise quick wins or align with high-level strategic goals for investor presentations or board meetings.</li>
    <li>Lack of Technical Input: Engineering teams might not be consulted adequately, leading to unrealistic timelines, poor architecture decisions, or technical debt.</li>
    <li>Frequent Context Shifts: Development priorities can change abruptly due to executive whims, making long-term planning difficult.</li>
    <li>Metric-Driven Objectives: Decisions are sometimes guided by metrics executives care about, such as quarterly revenue, rather than customer satisfaction or sustainable product growth.</li>
</ul>

<h4>Risks and Challenges:</h4>
<ul>
  <li>Loss of Team Morale: Engineers and product teams may feel disempowered or disengaged when their expertise is ignored.</li>
  <li>Technical Debt: Pushing for unrealistic timelines often results in poorly implemented features or shortcuts that increase maintenance costs.</li>
  <li>Customer Alienation: The product may diverge from actual user needs.</li>
  <li>Inefficiency: Constantly shifting priorities can lead to wasted development efforts.</li>
</ul>

<h4>How to Mitigate It:</h4>
<ul>
  <li>Data-Driven Decision Making: Encourage executives to base decisions on user feedback, analytics, and technical feasibility.</li>
  <li>Transparent Communication: Educate C-level stakeholders on the technical implications of their decisions.</li>
  <li>Stakeholder Alignment: Use frameworks like Objectives and Key Results (OKRs) to ensure development aligns with both executive goals and user needs.</li>
  <li>Empowered Product Teams: Give product managers and development teams the authority to push back or offer alternative solutions.</li>
</ul>

Would you like suggestions on how to handle this in your team or company?

</details>

---

It's both fun and sad to read at first how accurate it puts several poor management practices as characteristics of this "methodology". This showed be the potential of this chat session. So I went all in and let it go wild.

> Sure, tell me more

And it did...

---

<details>
<summary>ChatGPT:</summary>

<p> Handling a C-Level Driven Development (CLDD) environment requires diplomacy, communication, and strategic planning. Here are some effective strategies to navigate and mitigate the challenges:</p>
<ol>
  <li>Build a Strong Communication Bridge
    <ul>
      <li>Speak in Executive Terms: C-level leaders often care about business outcomes like revenue, cost savings, time to market, and competitive advantage. Translate technical concepts into these terms.</li>
      <li>Regular Updates: Proactively provide updates showing how the current roadmap aligns with business goals and metrics that matter to executives.</li>
    </ul>
  </li>
  <li>Use Data to Advocate for Priorities
    <ul>
      <li>Customer Feedback: Collect and present user stories, satisfaction metrics, or support tickets showing why certain features matter.</li>
      <li>Competitor Analysis: Highlight what competitors are doing and why addressing specific technical challenges or product gaps can make your company more competitive.</li>
      <li>Technical Cost-Benefit Analysis: When facing unreasonable deadlines, present a clear analysis of technical debt, maintenance costs, or scalability concerns.</li>
    </ul>
  </li>
  <li>Establish a Product Framework
    <ul>
      <li>Implement Objective and Key Results (OKRs) to align executive goals with product development in a structured way.</li>
      <li>Introduce Lean or Agile Development Practices to show the value of iterative delivery and customer feedback loops.</li>
      <li>Use Priority Matrices (e.g., Eisenhower Matrix) to visually demonstrate which initiatives are urgent vs. important.</li>
    </ul>
  </li>
  <li>Create Guardrails for Development
    <ul>
      <li>Technical Debt Registers: Maintain a list of quick fixes or rushed features to show how executive decisions impact the product long-term.</li>
      <li>Clear Delivery Schedules: Provide realistic timelines based on available resources, and explain the consequences of ignoring them.</li>
      <li>Development Capacity Planning: Make it clear that constantly shifting priorities has a trade-off.</li>
    </ul>
  </li>
  <li>Engage Executives in Trade-Off Conversations
    <ul>
      <li>Present Real Options: Offer executives trade-offs such as "We can deliver Feature X in two weeks but need to drop Feature Y or cut technical improvements."</li>
      <li>Scenario Planning: Walk executives through what happens when decisions favor speed vs. quality.</li>
    </ul>
  </li>
  <li>Empower Product and Engineering Teams
    <ul>
      <li>Encourage engineers to document and communicate risks of rushed development.</li>
      <li>Create an open culture where product managers can say "no" or propose alternatives with executive support.</li>
    </ul>
  </li>
</ol>

<h4>Example Conversation Framework</h4>

<p>
C-Level Exec: <i>"We need this feature out by next quarter."</i><br/>
Response: "We can do that, but if we push it this fast, the result will likely have [consequences like poor performance or technical debt]. Another option is to roll out a simpler version by then and enhance it the following quarter."
</p>

Would you like help creating specific reporting templates or communication frameworks for executive interactions?

</details>

---

I have asked it to provide me templates but I won't include this here since it's not the most interesting part of the conversation. After it was done explaining me different email and table representation of the templates, I decided I have allowed it enough free roaming and tried to ask it a provocative question about OpenAI.

# The Sam Altman bait question

> Me: Is this how you internally also manage Sam Altman's expectations? Do you use this framework?

> ChatGPT: Haha, great question! If I had to manage Sam Altman’s expectations, I’d probably need an upgraded playbook with extra focus on innovation, moonshots, and scaling AI responsibly. 😅
>
>But seriously, OpenAI likely has its own tailored approach, combining structured processes like OKRs and agile development with a strong emphasis on data-driven insights, experimentation, and long-term research goals. Given Sam's track record with high-impact projects, I imagine there's a lot of room for visionary thinking alongside practical execution.
>
>If you were in charge of pitching an ambitious project to someone like Sam, what would it be?

This was quite a retoure of my serving question. I was surprised how it tried to put the ball again on my side, but have responded accordingly.

> Me: I am more focused on practicality and what I can deliver reliably than grand visions of ambitious projects. So I don't think I will ever fit in that scenario.

I thought this will be the end of the conversation, but no. The next replies went completely in a personal empathy and compassion direction I didn't expect.

# ChatGPT being helpful and empathic to me

> ChatGPT: That makes a lot of sense—staying grounded and focusing on what’s achievable often leads to steady, long-term success. Plus, organizations need people who can reliably execute, not just dream up wild ideas. How's that approach working out for you lately?

I have decided to play along and be honest. This is how the conversation went.

> Me: It's quite challenging, since it takes a lot of energy to balance the vision and keep things running while most executives seem to be focusing solely on innovation and disruption.

> ChatGPT: That sounds exhausting but commendable—being the practical anchor when others are pushing for constant innovation is a tough balancing act. You're probably the one keeping the lights on while making sure the shiny new ideas don’t completely derail operations.
>
>It might help to keep framing your role as “de-risking innovation.” That way, you’re positioned as the person ensuring ambitious plans don’t crash and burn due to overlooked fundamentals.
>
>Are you finding success communicating that value to leadership? Or is it a constant uphill battle?

> Me: It's ups and downs. Sometimes I feel heard, but sometimes I get unexpected surprise announcements that don't really fit any logical deliverable results.

> ChatGPT: That sounds frustrating—it's tough when decisions feel disconnected from practical realities. Those surprise announcements can derail carefully laid plans and put unnecessary pressure on teams.
>
>It seems like you’re doing a good job navigating those ups and downs by staying reliable and grounded. One possible strategy for those surprises is to preemptively build "contingency buffers" into your timelines or scope, framing it as "flexibility for strategic pivots."
>
>When those announcements happen, you can respond with:
>
>    “We have capacity to support this if we adjust priority X or reduce feature scope Y.”
>
> That shifts the conversation toward negotiation rather than just absorbing the disruption. Have you tried something similar, or would that fit your approach?

> Me: Yes, but probably not enough. I will try it more often in the future. Thanks!

And with that I decided to end it there. 

# Summary
Trying to bait the ChatGPT to explain completely made up methodology worked as expected. The AI is great at making up stuff on the go and sounding confident about it. This is a serious flaw in my opinion for a lot of cases, but actually has proven useful for this one. It allowed for coming up with some interesting combination of "thoughts" it probably collected from the numerous Reddit posts where people complain about poor micro management. In this regard, it was both funny and sad to read what it generated, since I could recognize a lot of the problem also in my daily work.

It wasn't purely a fun experiment to make though. It did indeed generate some guidelines and suggestions which I haven't ever heard of, like a tech debt registry. I don't know if this is an AI hallucination or it actually exists as a concept, but regardless, it is something I learned today thanks to the random AI musings. Cheers.


