---
layout: post
title: "My Summer of Bitcoin Journey: Contributing to Utreexo"
date: 2024-08-19
author: "Njokom Nshanui"
categories: [Stories, Utreexo]
---

Over the last 3 months or so, I was privileged to have been selected for the 2024 session of the Summer of Bitcoin (SOB) internship. During this time, I worked on Utreexo project. Utreexo significantly reduces the resources needed to run a fully validating Bitcoin node, by introducing a hash based dynamic accumulator approach for representing the Bitcoin state.

This post documents my experience working on this project, the particular feature I was tasked with, and it's impact to the overall system.

### How was the internship carried out?

The internship comprised of several phases. The first phase was the "Learning phase" During this phase, all applicants were provided with resources for learning about Bitcoin. Precisely, we were tasked with studying the book Grokking Bitcoin by Kalle Rosenbaum. This phase consisted of weekly meetups and targets to be met/studied for that week. During the meetings, we were split into several breakout rooms, where, we discuss among ourselves (other prospective interns) about what we understood from the target section for that week, after which we get back in the main Meeting room, where we can further discuss with, and ask questions from the Organizers.

We then proceeded to the next phase, where we were given an assignment to do and submit. The task was to build a simple Bitcoin mining node simulator. We were given sample transactions as json files and tasked with carrying out the steps to validate transactions, assemble a block and successfully mine the block. This was certainly a very challenging task, as I came into this project with no knowledge writing code to perform this. However, the knowledge gathered from studying the above Ebook, and so many other online resources, especially knowledge gotten from learn me a bitcoin were of immense help here. Thankfully, after several weeks of intense hard work and learning, I was able to get a near optimal bitcoin mining simulator. That was one of the proudest moments I have ever had 😁.

Anyways, from here, we had to scan through the available organisations and find one which aligns with what we are looking for and checkout their projects, and write a proposal. Note that it is highly advisable to start searching for organisations and get familiar with them and even start making contributions to them immediately they are added. This will give you a head start and improves your chances of coming up with a great proposal. While I didn't find my preferred organisation early, I certainly did find them on time, and started going through their documentation and codebase to understand what the project is all about and what is required in order to make a contribution.

Finally, I picked two organisations GoCoin and Utreexo. My proposal for Gocoin wasn't selected, however, my proposal for Utreexo, with was about "Implementing new getcfilters message for utreexo roots" was selected. I can't even begin to explain using words how excited I was when I received the mail.

Next up was the contribution phase. Here, we had to actively make contributions to the selected project. This can be considered as the internship proper. Although I got to start exploring the documentation and code base of my project early, I had a hard time getting hold of my mentor, and this set me back a little bit. I eventually got hold of him, and we sped up things. He was really nice and guided me thoroughly throughout the journey.

After working for about a month and a half or so, came time for mid terms evaluation, where progress was evaluated and the first part of the stipend was paid out after passing the evaluation.

We continued working for the rest of the internship period, and eventually, we had the final evaluation towards the end of August.

### What exactly was my contribution to the project?

So as earlier mentioned, i was tasked with implementing new getcfilters messages for utreexo roots. These messages were to be implemented as per the BIP157 specification. The aim of this was to allow A light client peer to be able to ask for the utreexo roots at a given height from a peer. Essentially, I had to add a filter type and replace the filter contents with the utreexo roots and serve to a peer when requested. Sounds easy right? So I thought.

I started by defining the protocols for the new message type, adding the new type to the list of supported message types. I called it "UtreexoCFilter". I know,.. not a very convincing name, but well.

Next I wrote the implementation for creating headers for this new filter type

This was followed by the actual implementation of the filter indexer, updating the implementation of the handler, the actual implementation of the handling function and so much more.

I had so many hurdles along the way, making changes that caused the node to panic on start, resolving the panic, but getting unsatisfactory results, initial block download getting stuck, and so many issues that really made me scratch my head. In the end after several days/weeks of debugging, and with the help of my mentor, I was able to resolve all of the issues and get the node fully functional and working properly.

### What impact did this have on the project?

The node already supported compact filters. However, adding this new filters gives other Compact state nodes the ability to request utreexo roots from a particular height from a peer. When they receive these nodes, they can easily verify the received roots.

Overall, the internship was an exciting and really resourceful, learning opportunity for me, albeit extremely stressful, but I was very happy to put in the work, and I was glad to see good results. This program has given me a really solid foundation, and confidence to take on any Bitcoin and blockchain related project.

Special thanks to Adi, the lead organizer who was setup everything and made sure I had this wonderful opportunity.

A special thanks too to my mentor Calvin Kim, who was the best mentor anyone could have asked for. He sacrificed his time and energy to make sure he's always available when I need his support, and he also pushed me to my limits and made sure I learnt a lot during this journey.
