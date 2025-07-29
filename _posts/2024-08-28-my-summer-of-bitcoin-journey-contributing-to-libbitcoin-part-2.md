---
layout: post
title: "My Summer of Bitcoin Journey: Contributing to Libbitcoin (Part 2)"
date: 2024-08-28
author: "Shashank Shekhar Singh"
categories: [Stories, libbitcoin]
---

Hi there! I’m Shashank Shekhar Singh, a junior year student in Mechanical Engineering at the Indian Institute of Technology (IIT) BHU, Varanasi. Since I have already discussed about the Summer of Bitcoin (SOB) program and Libbitcoin organisation in my previous blog, let me talk about how I started my journey with SoB’24 and, more specifically, with the Libbitcoin organization.

My journey with SoB began with the four-week learning bootcamp, where we were introduced to the world of intuition, development and growth of Bitcoin. We were provided insights on the “Grokking Bitcoin” book, a cornerstone for learning and understanding Bitcoins. We also connected via Discord, where I had the opportunity to interact with various Mentors, Student Coaches, and Adi — The program lead, who collectively guided us through our doubts.

We were also provided an assignment to implement after having a deep understanding of Bitcoin development and working. This assignment was a necessary criterion for being selected in the SoB program 2024.

Alongside completing the assignments, I diligently searched for the right organization that aligned with my interests. This process involved meticulously browsing through all the organizations listed on the SoB website, evaluating their projects, and engaging with the mentors. We were allowed to submit a maximum of three proposals, so I made sure to reach out and communicate with potential mentors early on to increase my chances of being selected.

I preferred Libbitcoin organisation because it’s project aligned with my experiences and interests. So, I started communicating with the mentors at the earliest through their Slack Channel.

In my first blog, I discussed the design of the table schema and the initial set of queries for my SQLite implementation of the libbitcoin-database.

After implementing the schema and the queries, I wrote several tests to verify their functionality. These initial tests allowed me to check the correctness of the data storage and retrieval processes and were essential to ensure that the schema was robust and could handle real-world data scenarios.

**Diving Deeper: Post Mid-Term Evaluation Developments**

Following mid-term evaluations, I started to dig deeper and started writing major queries related to the database. These queries include operations to create, populate, and validate records in the database that would be essential for the ‘libbitcoin-database-sqlite’ to function seamlessly within the broader ‘libbitcoin’ toolkit. I refactored my previous code to the writing style of the organisation.

Some of the queries that I wrote are as follows:

**Validation Queries:**
1.  `is_header`: Checks if a header exists based on its primary key.
2.  `is_block`: Checks if a block exists based on the foreign key of its header.
3.  `is_tx`: Checks if a transaction exists based on its primary key.
4.  `is_coinbase`: Checks if a transaction is a coinbase transaction.
5.  `is_milestone`: Checks if a header is a milestone.
6.  `is_associated`: Checks if a header is associated.

**Specific GET Queries**
*   `get_column_value`: A general query to retrieve a specific column value based on a primary key.
*   `get_tx_keys`: Retrieves transaction keys (foreign keys) associated with a given header link.
    *   SQL: `SELECT transaction_FK FROM aTxs WHERE SK_header_FK = ?`
*   `get_header_key`: Retrieves the primary key of a header based on its link.
*   `get_point_key`: Retrieves the primary key of a point based on its link.
*   `get_tx_key`: Retrieves the primary key of a transaction based on its link.
*   `get_height`: Retrieves the height of a header based on its primary key.
*   `get_tx_height`: Retrieves the height of a transaction by first finding the associated header’s height.
    *   SQL: `SELECT height FROM aBlock WHERE SK_header_FK = (SELECT SK_header_FK FROM aTransaction WHERE SK_hash = ?)`
*   `get_tx_position`: Retrieves the position of a transaction within its block.
    *   SQL: `SELECT position FROM aTransaction WHERE SK_hash = ?`
*   `get_value`: Retrieves the value of an output based on its primary key.

**Populate with Meta-data queries:** (`populate_with_metadata`) A little complicated Queries to interact with the database and add meta-data about the tables, required for a faster access to the data storage.

During this phase, I also gained a deeper understanding of key concepts like MUTEX and templating, which are crucial in C++ development. Working closely with the ‘libbitcoin’ GitHub repository, I learned how different repositories interact with one another, how to manage concurrency using mutexes, and the importance of templating in writing flexible and reusable code. This experience gave me valuable insights into how a complex, modular system like ‘libbitcoin’ is structured and maintained.

In the last two weeks, I have set up the development environment for libbitcoin, which involved a few critical steps:
1.  Sync libbitcoin-build project.
2.  Clear output directory.
3.  Generate developer_setup.sh scripts: `gsl -q -script:templates/gsl.developer_setup.sh generate4.xml`
4.  Copy generated content to repository location.
5.  Run `developer_setup.sh` with desired options (use — help for details).
6.  For first build, use `— build-target=all` to prepare dependencies.
7.  Typical build command: `./developer_setup.sh — build-mode=configure — build-target=libbitcoin — build-src-dir=<path> — build-obj-dir=<path> — prefix=<path> — disable-shared <project params>`.

While the setup was straightforward, the challenge lies in integrating this small, independent codebase with the much larger multi-repository structure of libbitcoin. I’m currently working on this integration and am expecting to complete it in the coming month.

I have dedicated a significant amount of time to ensure that every detail is thoroughly thought out, and have actively seeked feedback from my mentors to refine my approach. My enthusiasm for this project has only grown, and I’m committed to continue to contribute to this project even after the completion of Summer of Bitcoin program.

Eric Voskuil, one of the founding members of libbitcoin and my mentor, has been invaluable in guiding my understanding of the project. Over three years of dedicated work, he developed a highly efficient local database for storing Bitcoin network data. Our weekly meetings, along with additional help sessions as needed, have provided me with the flexibility to explore the project at my own pace. This freedom has allowed me to focus effectively on my work without the pressure of strict deadlines.

I would also like to express my gratitude to my mentor, Kulpreet Singh, who had confidence in me during the contributor selection process and chose me for this project. His blogs have inspired me to explore Bitcoin more deeply and consider a potential future career in this field. His dedication to contributing to the libbitcoin toolkit on a voluntary basis motivates me to become a lifelong active participant in Open Source.

I owe a huge thanks to my mentors, including Eric Voskuil and Kulpreet Singh, and SoB Program Co-ordinator Adi, for giving me this wonderful opportunity to work on and learn how large real-world projects are developed and maintained by a community of developers. This experience, working on my ‘first’ Bitcoin blockchain project, has been invaluable, and I look forward to contributing more to the open-source ecosystem in the future.
