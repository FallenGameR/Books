# The Practice of System and Network Administration, Book 1

## Notes

- 30 min to make VGA work story
- transparent org-wide updates
- vendor lock (email app), no standart
- network complexity - ISP+advanced load balancing vs corporate networking skills
- separate section on visibility for SAs (who deal with infrastructure) - fixed problem like tree in the forest that fell, nobody notices it
- Subnet vs VLAN
- 3 secret layers of OSI model - user, financial, political
- Cable interferance from a power cable
- Keeping dust outside of datacenter - workbench with antistatic mats where unboxing happens
- Architect positions - for people that have end-to-end understanding of their system and it's interactions with connected systems, invoked on major outages. Official role involves:
  - predict needs and technologies in 2-5 years horizon
  - start preparing for the new needs/tech - prototyping, getting involved with vendors, steering direction of vendor products
  - steer people to more scalable solutions
- A tiny team can communicate verbally. As the team grows, however, you must change from an oral tradition to a written tradition =)
- Setup git repo in default linux folders:
  - /etc - system app configs
  - /usr/local - local app configs
  - /home - personal files and configs
  - /var - some service specific change logs subdirectory
- Upgrade announcement:
  - Use blank templates, don't update the previous announcement
  - Title - brief (what & when) - enough to decide if read through is needed
  - Who is affected
  - What will happen
  - When
  - Why with link to more information
  - I object! with link how to postpone
- CAP theorem: consistentcy, availability, partition_tolerance - pick two. In case of a networking partiion a distributed system can either be consistent or available, but not both of them together.
- Personal integrity (ethics)
  - I will be honest in my professional dealings and forthcomming about my competence and the impact of my mistakes. I will seak assistance from others when required.
  - I will avoid conflicts of intereset and biases whenever possible. When my advice is thought, it I have a conflict of interest or bias, I will declare it if appropriate, and recuse myself if necessary.
- The consequences of making a mistake should be specified. The best policy is to state that there will be no punishment for making an honest mistake as long as it is reported quickly and completelly. The sooner a mistake is reported, the sooner it can be fixed and the less domino-effect damage that will occur.
- Be firm with people who are harming (your company). If they turn into bullies, it's better to find a bigger bully to fight for you.
- Controlling costs explanation to management. The SA group doesn't control the cost of it's budget during scaling out if it pays for support of other teams' people and other teams' equipment that keeps grows. Scalable way is to make other groups financially reponsible for the support of themselves.
  - Who pays?
  - How does it scale?
  - What do cusomers get for their money?
- When things in an organization are not doing well, hire reform-orineted people. Highly skilled people with expirience at well-run sites are more likelly to be reformers. On the other hand if things go well hire hunior people, reward best workers, retain institutional knowlege.
- Customers don't like faceless organizations, it is easier to get angry at a person whom you have never met.
- "Thank you for reporting this. It helped us fix the problem and find ways to prevent it in the future."
- "Let's hear from the people in the room who have not spoken yet" (if somebody talkative takes all the initiative). In extreme cases go round the room asking each individually.
- Add who needs to read this to mass communication emails (or who can stop reading because this is not relevant).
- Free lunch lotery based on the ticket number (every 1000th).
- You are spending so much time mopping the floor that you don't have time to fix the leak.
- Three Different Backgrounds - a design had 1) to have enough structure to satisfy a religios-background person, 2) each component had to be self-reliant to satisfy a orphan-background person, 3) system had to be able to deal with change well enough to satisfy the nomad-background person.

## Ideas

- Monitoring areas: availability, performance, problems, capacity planing
- The difference between problem and crisis is preparation
- Upgrade-related interruptions are less annoying when customers understand what is going on, have some control over the schedule and know what they are going to get out of it ultimatelly
- Address every problem as a puzzle, one that is a fun challendge to solve.
- Read emails once - if you open it, reply. If you would not have time to process it, don't open.
- Manager should be concerned with steering the boat, not rowing it.

## ICM handling

With Seinfield-esque names for people who systematically miss that particular step.

- Greeting (the grumpy)
- Problem classification - what team owns (the misdelegator)
- Problem statement - enough info to repro (the assumer)
- Problem verification - repro can be reproduced (the mislead one)
- Solution proposals - what are the hypotheses (the fixer of a wrong problem)
- Solution selection - arange by likeliness (the fixer of a wrong problem)
- Solution execution - apply the fix (the executioner on a wrong machine)
- Verification - did it work from your side? (the hit and run support)
- Customer verification - did it fix the problem from the customer side? (the hastly deal closer)

## Postmortem handling

After the event when it is sufficiently quiet gather flight director and senior SAs to sit down and talk:

- what went right, and why
- what went wrong, and why
- which improvements and optimizations can be made

Gather notes and discuss with the whole group later in the week.

Common mistakes:

- trying to do too much in the alloted time
- not doing anough work ahead of time
- underestimating how long something will take

## Centralization vs decentralization

> Centralization

- Efficiency since there is no duplication of effort
- Improved quality of the service, dedicated experts
- Automation that leads to more reliable service and faster turnaround times
- Systems are designed to be remotelly managed from the start

> Decentralization

- Historic org division (company merges, big companies)
- Low quality of the centralized service
- Lack of customization of the centralized solution, new features are not landed in time
- Some way of reducing the cost (or likelly hide it elsewhere)

## Communication

- I statements - "I feel {soft_emotion} when you {action}"
  - hard emotions like anger makes people defensive and they would not hear you out
  - soft emotions: hurt, unvalued, very happy, frustrated, disappointed, untrusted
- Mirroring - "I hear you are saying that..." - like CRC checks on file transfer
  - "Say again?" - an ask to repeat the said words
- Reflection (Validation) - "Wow! You are really upset about this."
  - the secret of dealing with angry people
  - do something to acknowledge their emotions - this will calm them down
  - you must deal with their emotions before you can deal with their complaints
  - most of the frustration comes from the feeling that he/she is not being heard
  - it would tell how he/she looks which can embaras slightly and then regain composure
- Negotiation - "Is that negotiable?"
- Accept compliments - "Thank you"
  - If someone is being polite enough to compliment you, be polite enough to accept the darn compliment!
- Ask for promotion explicitly - "I want you to know that I want to be one of the student managers here, and I'll do what it takes to get there".
  - Manager can structure your tasks and training in the right direction.
  - Manager like that you are seeking their wisdom.
  - Don't bring complaints (selfish requests), bring an interesting problem to solve. Align with boss'es goals.
- It is difficult for a manager to turn down a reasonable request made immediatelly after having complimented you on a job well done or thanking you for saving the compnay money.
  
## Hiring

- Job description writes itself
  - Document stuff you dislike doing
  - That makes the list of tasks you want to delegate documented - that would be the responsibilities
  - The skills required to perform those tasks is the list of the skills that a person should have
- An interview candidate should have the impression that interviewiving them, was the most important thing on the interviewers' minds this day. Candidate need to leave the building with their self-respect intact and liking the company.
- Don't prolong the agony - when you find the candidate's limits in one area, back off and try to find their limits in another area. Give the candidate a change to build up confidence before probing their limits.
- Ask deeper and deeper questions untill candidate says "I don't know" to find out the limit of their knowledge. Explain that that's fine, so that candidate would not loose self confidence and you would be able to test limits of their knowledge and would be probably surprised that that "limit" is just fine.
- Interest in computers, two extremes:
  - Computers are focus of life. May work long hours but that is not healthy, will likelly result in burn-out and this is not a recipe for a long-term success. If the candidate plays with a new technology it can be difficult for them to figure out how much real work is was done and how much was just playing around.
  - Compute-averse, would not use computers ouside of work. Sustainable lifestyle, will not burn-out (probably they already had that expirience). More likelly to be more productive in all the hours they do the work. Less likelly to try out and adopt a new technology or suggest introducing it.
- Retention - keep the job interesting, enjoyable.
  - If there is a tedious, repetitive task, get someone to automate those tasks. The person who is creating the automation will enjoy it, the SAs who no longer have to do the repetitive task will appreciate it, and efficiency will be improved.
  - People work for money. People work hard for appreciation. People work the hardest when they love what they do. People work the best when they love what they do, where they do it, and who they do it for.
  - People take jobs because of money, but people leave jobs because of bad managers.
  