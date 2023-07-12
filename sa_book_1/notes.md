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

## Ideas

- Monitoring areas: availability, performance, problems, capacity planing
- The difference between problem and crisis is preparation
- Upgrade-related interruptions are less annoying when customers understand what is going on, have some control over the schedule and know what they are going to get out of it ultimatelly

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
