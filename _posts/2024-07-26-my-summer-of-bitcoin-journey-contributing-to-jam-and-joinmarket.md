---
layout: post
title: "My Summer of Bitcoin Journey: Contributing to Jam and JoinMarket"
date: 2024-07-26
author: "Amit Kumar Prasad"
categories: [Jam, Stories]
image: "assets/images/avatar.png"
---

The following blog provides a comprehensive overview of my experiences and
activities throughout the different phases of the Summer of Bitcoin program

Hello, everyone! As summer 2024 comes to an end, I’m excited to share that
I’ve just completed my internship with Summer of Bitcoin. This experience has
undoubtedly been one of the most rewarding and memorable moments of my recent
journey.

On May 9, I received an email announcing that, out of over 5000 applicants
from more than 50 countries, my application was selected after several rounds
of rigorous screening and review. I was one of the 30+ students chosen for the
Summer of Bitcoin 2024 program. This news brought me immense happiness and
pride, as I had been struggling significantly at that point in my life. Those
days were truly challenging for me. But what made it truly special was that,
for the first time in my life, I did something that had a real impact on the
world.

> Here’s a basic overview of how the program looks

# Applications and screening:

The initial step involves completing the applications. If the program
organizers consider you a suitable candidate, then you will be selected for a
4-week bootcamp. At the start of every week, you will be provided with a
reading resource. Before each seminar, you’re expected to study the resource
and prepare a question based on your reading. During the seminar on Zoom, you
will join other participants and instructors to discuss answers to the
questions. At the end of the bootcamp, you will be provided with an assignment
to solve. And the selection for SOB’24 is evaluated on the basis of our
solution to the assignment and the project proposal. My assignment involved
simulating the process of mining a Bitcoin block. It involves several key
steps: reading transactions (P2PKH, P2SH, P2WPKH, and P2WSH), validating these
transactions, creating a block header, serializing a coinbase transaction, and
extracting transaction IDs (txids). And also mining a block so that the block
header meets a specific difficulty target. And FRI it took me 20 fuc*ing days
to successfully complete the assignment. Here If you are interested, have a
look code-challenge-2024-amitx13

> A small disclaimer: If you don’t know the fundamentals, please avoid looking
> because it will just blow up your mind.

# Proposals:

Selected candidates proceed to the next stage, where they are presented with a
list of organizations and projects. Each participant is encouraged to propose
ideas for one or two of these projects. While the website provides all the
necessary technical details, the most challenging aspect for me was deciding
which organization to select. Because around 70% of the project was using
Rust, and back then, I was not a Rustacean. But eventually, after few days, I
found one that best suited my tech stack

Once you’ve found some organizations that excite you the most, you should go
on and try to complete their project’s competency tests. Completing a
project’s competency test successfully indicates that you have everything the
project requires and that, with enough effort, you will be able to complete
the project on time.

One thing I believe I did better than other students who submitted proposals
to JAM was actively participating in community discussions. And assisting
other students with their competency tests and addressing their project-
related queries.

> Open source is more than just code; it’s a community-driven effort where
> knowledge is shared openly, collaboration thrives, and innovation knows no
> boundaries

Again, the SoB website has all you need to know about writing a superb
proposal.

_Tips:_ Clearly demonstrate your genuine intent by showing that you’re not
just participating for the sake of Summer of Bitcoin or the $3000 stipend, but
that you align deeply with the organization’s goals and plan to contribute
actively even after the program concludes.

Here’s my proposal for JAM: apX13_Proposal_for_JAM

# The Onboarding:

The program kicked off, and the onboarding process began. Selected proposals
are instructed to meet with their mentors to discuss their work and
timeframe,and get started with their projects.

I also want to express my gratitude to Adi, the ultimate [MVP], for the
exceptional onboarding process. Adi made everything smooth and exciting from
day 0 and guided us through every step of the program. His support and
assistance played a significant role in making this journey even more
enjoyable and rewarding.

# My organization, JAM, and the problem statement that I solved:

> Remember: If you don’t understand any of these buzzwords, that’s totally
> fine. No one is perfect on day one; just keep upgrading yourself.

JAM is a completely decentralized, free, open-source, and non-custodial
project that aims to improve the financial privacy of yourself and others
without relying on a trusted third party. It is powerful because it is run by
a protocol, not a company. No central entity is in charge or profits in any
way. You are in charge. You are the one who profits. This is one of the main
reasons why I chose Jam to be a part of it because they are here to provide
value to the community. They are here to help others improve their financial
privacy. They are not here to earn money and fly away. They are here to make
the community strong and reliable.

Currently, users face limitations in selecting individual UTXOs for
transactions, resorting to a cumbersome “freeze” or “unfreeze” method. The aim
is to revolutionize this experience by introducing a seamless UI/UX flow that
empowers users to efficiently choose specific UTXOs for both regular and
collaborative transactions by incorporating the much-needed “Coin Control”
feature.

> The project can be divided into three main tasks:

1: Implementation of “Coin Control” for the Jam wallet.

2: Design and integrate intuitive UI/UX components to facilitate a smooth
interaction flow.

3: Allow users to select single or multiple UTXOs effortlessly for
transactions, improving flexibility and convenience.

# Contributions:

My contributions to JAM & JoinMarket include exactly 1000 lines of code, and 4
PRs. If you’re not familiar with JoinMarket, think of it as the backend, with
JAM serving as the frontend for JoinMarket. The 4-week bootcamp at the start
of the program was instrumental in my project’s success, as it was where I
first learned about UTXOs, collaborative transactions, and other Bitcoin-
related concepts.

> Here are some of my most valuable contributions:

JAM:

UTXOs Selection Modal: https://github.com/joinmarket-webui/jam/pull/771

UTXOs Review Modal: https://github.com/joinmarket-webui/jam/pull/807

BTC Input field: https://github.com/joinmarket-webui/jam/pull/800

JoinMarket:

Updated direct-send RPC-API to accept List of UTXOS:
https://github.com/JoinMarket-Org/joinmarket-clientserver/pull/1721

It’s been a hell of a ride — lots of code, reviewing, meeting and working with
incredibly awesome mentors The OG Theborakompanioni and god of UI/UX Edi. But
what made it truly special was that, for the first time in my life, I did
something that had a real impact on the world.

I think it’s essential for all of us to have a vision of the world we want to
build, and I believe I might have just found mine. This is the future I’m
building!

# Community support:

The man, the myth, the legend — TBK (Theborakompanioni). Back when I was a
MERN stack developer at a product-based company, mentors would often try to
impose their thoughts on me. However, TBK encouraged me to come up with my own
solutions. This not only improved my decision-making skills but also helped me
grow as an individual. Both of my mentors are excellent developers, yet they
always sought my opinion. The level of respect and support they showed was
truly great. I never felt like a student intern; they always made me feel like
a valuable part of the project. This helped me engage fully with the project
and successfully complete it.

Every Tuesday at 8:30 UTC, we had a team meeting to discuss our progress and
plans for the week. My mentor made me comfortable with making mistakes and
learning from them. I was never pressured and was always reminded that there
were no silly questions. I feel incredibly lucky to have had such a kind
mentor. Truly speaking, I wouldn’t have been able to complete the project
without their guidance.

# Conclusion:

In wrapping up, this summer has been an amazing journey of learning and growth
that I’ll always treasure. A big thank you goes out to the Summer of Bitcoin
program for giving me this incredible chance.

Throughout this journey, my experience has been fantastic. Regular meetings
with my mentor and fellow contributors have been really helpful. We discussed
our progress and challenges every week, which made me feel supported and
encouraged.

This summer, despite the hard work and challenges, has been an incredible
learning experience. I’ve been able to balance technical growth with practical
application. It’s been quite an adventure, and I’m grateful for the
opportunity to be a part of this journey.

To everyone who helped me along the way — whether with advice, feedback, or
just words of support — thank you. Your contributions shaped my understanding,
boosted my skills, and made this journey a real collaboration.

For those who are looking forward to applying for the next cohort of Summer of
Bitcoin, all the very best. If you have any doubts or questions, feel free to
contact me, my DMs are always open.

-Amit Prasad

twitter(@apX13_ )

LinkedIn (@mitPrasad )

GitHub (@amitx13 )

Gmail (amitkvs981@gmail.com )