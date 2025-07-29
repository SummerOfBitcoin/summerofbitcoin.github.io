---
layout: post
title: "My Summer of Bitcoin Journey: Adding Standardized Pool Metrics to PyASIC"
date: 2024-08-02
author: "Temiloluwa Yusuf"
categories: [Stories, Pyasic]
---

Marvellous May turned out to be an exciting month as I received a congratulatory email for being accepted as a 2024 Summer of Bitcoin intern. i was selected to contribute to the “ pyasic: Add Standardize Pool Metrics ” project. As a Python developer, I have always wanted to gain first-class experience developing open-source software, and the 2024 Summer of Bitcoin internship made my summer an interesting one to remember.

“In this article, I will discuss my contributions to pyasic an open-source Bitcoin mining library developed in Python.”

### INTRODUCTION:

In the rapidly evolving landscape of Bitcoin mining, having efficient and reliable tools is crucial. One such indispensable tool is pyasic. Pyasic is an open source Python library that provides a comprehensive suite of functionalities aimed at enhancing the efficiency and reliability of Bitcoin mining operations. It serves as a crucial component in the bitcoin ecosystem, enabling miners to monitor and optimize their mining activities seamlessly.

### UNDERSTANDING PYASIC

**WHAT IS PYASIC AND WHAT DOES IT DO?**

Pyasic is an open-source Python library designed to facilitate various aspects of Bitcoin mining. It provides a robust framework for interacting with mining hardware, allowing users to perform tasks such as monitoring miner status, monitoring performance metrics, retrieving mining statistics, and managing mining pools. With its extensive set of features, pyasic simplifies the complex process of mining management, making it accessible to both individual miners and experienced miners.

**KEY FUNCTIONALITIES OF PYASIC INCLUDE:**

1.  **Miner Communication:** Pyasic provides APIs to communicate with different types of mining hardware. This includes sending commands to miners and receiving their status updates.
2.  **Data Collection:** The library allows for the collection of critical mining data, such as hash rates, temperature, power consumption, and more. This data is essential for optimizing mining operations and ensuring the longevity of mining hardware.
3.  **Pool Management:** Pyasic includes features for managing mining pools, enabling miners to efficiently distribute their mining power across multiple pools to maximize profitability.
4.  **Error Handling:** The library is equipped with robust error handling capabilities to manage common issues that arise in mining operations, such as hardware malfunctions and network interruptions.

**PYASIC SIGNIFICANCE IN THE BITCOIN MINING COMMUNITY.**

By offering a standardized and user-friendly interface, pyasic helps miners maximize their hardware’s potential, reducing downtime and increasing mining rewards. This contribution is vital for the sustainability and growth of the Bitcoin network.

### SETTING UP YOUR ENVIRONMENT

Step-by-step guide on how to clone the pyasic repository.

1.  Fork the project repository and clone it from github to your computer.
    ```
    git clone https://github.com/UpstreamData/pyasic.git
    cd pyasic
    ```
2.  set up a virtual environment and install dependencies.

It is recommended to install pyasic in a virtual environment to isolate it from the rest of your system. Options include:

*   **pypoetry:** the reccommended way, since pyasic already uses it by default.
    ```
    poetry install
    ```
*   **venv:** included in Python standard library but has fewer features than other options
*   **pyenv-virtualenv:** pyenv plugin for managing virtualenvs
    ```
    pyenv install <python version number>
    pyenv virtualenv <python version number> <env name>
    pyenv activate <env name>
    ```

Installing pyasic
```
python -m pip install pyasic or poetry install
```

Tips for navigating the codebase and understanding its structure.

*   `pyasic/`: Contains the source code.
*   `tests/`: Houses unit tests for the project.
*   `docs/`: Includes documentation for the library.
*   Start by reading the `README.md` for an overview and check the documentation for detailed guides.

### CONTRIBUTING TO PYASIC

**IDENTIFYING AREAS FOR CONTRIBUTION:**

*   look for issues in the GitHub repository.

**ADDING STANDARDIZED POOL METRICS TO PYASIC:**

*   **Project Overview:** One of the contributions i made to pyasic is adding standardized pool metrics. This enhancement ensures that pool metrics are consistent and easily accessible, improving the monitoring and management of ASIC miners.
*   **Implementation Details:** For a detailed look at how i added standardized pool metrics, check out my merged pull request. This PR outlines the code changes and the addition of new metrics. The source code can be found in the `pyasic/data/pools.py/` directory.
*   **Writing and running tests:** Here, i also wrote unit test for the standardize pool metrics added in `pools.py` placed my tests in the `tests/` directory and ran them using pytest

### PROJECT PROPOSAL: ADD STANDARDIZED POOL METRICS

*   **Overview of the project:** The goal of my project was to enhance pyasic by adding standardized pool metrics, making it easier for miners to track and optimize their mining operations.
*   **Background and Importance:** Pool metrics are crucial for miners to understand the performance and efficiency of their mining hardware. Standardizing these metrics within pyasic helps create a consistent and reliable framework for monitoring and management, benefiting the entire Bitcoin mining community.
*   **Proposed Solution and Approach:** When using pyasic, users can call the `get_data` function to get data for a miner in a standard format, but this format does not include common pool metrics such as accepted/rejected shares or active pools. It does include a config which contains pool configuration data, but this isn't useful for checking pool status. The goal of this project is to provide a new dataclass to be assigned as the `pools` attribute to `MinerData`, the dataclass returned from the `get_data` function, and add the ability for most or all of the supported miners to parse this from their APIs.

**Documenting My Progress for summer of Bitcoin 2024 (mid evaluation):**

**Tasks Completed:**

*   Designed a schema, a dataclass that holds important pool metrics.
*   Identified key pool metrics needed for effective monitoring. The metrics include accepted, rejected, get_failures, remote_failures, active, alive, url, index, user, pool_rejected_percent, pool_stale_percent.
*   Integrated these metrics into the pyasic library.
*   Implemented `_get_pools` method for Antminer, BOSMiner, BMMiner BOser and LuxMiner backend classses.
*   Deduced pool information with whatsminer api for the `get_pools` method.
*   Created a Scheme Enum with `STRATUM_V1` and `STRATUM_V2` variants.
*   Defined the `PoolUrl` dataclass with scheme, host, port, and an optional pubkey.
*   Updated `PoolMetrics` to include a `PoolUrl` attribute.

**Challenges Faced:**

*   Data consistency: Variations of attributes from different pools.
*   Integration issues: Arrangement of pool metrics attributes broke production code .

**Learning Outcomes:**

*   Improved understanding of Bitcoin mining and the role of ASIC miners.
*   Gained experience in contributing to bitcoin open-source projects.
*   Enhanced skills in Python programming, concurrency(async/await ), virtual environments (poetry), gRPC, RPC, REST APIs and software testing.
*   improved understanding of version control (git).

### UNDERSTANDING BITCOIN:

**What is Bitcoin?**

Bitcoin is a peer to peer electronic system that allows online payments to be sent directly from one party to another without going through a financial institution. Bitcoin represents a new and open internet standard for hard money. Bitcoin is increasingly being adopted as a pristine collateral, a longer-term store of value, and an unstoppable money.

**Why Bitcoin Matters:**

*   **Financial Freedom:** Bitcoin offers an alternative to a traditional financial systems, providing more control over one’s assets.
*   **Decentralization:** It reduces the power of centralized entities, promoting a more equitable financial landscape.
*   **Global Impact:** Bitcoin has the potential to empower individuals in regions with unstable financial systems, offering a stable and secure means of transacting and storing value.

**Why Pay Attention:**

*   Bitcoin represents a paradigm shift in how we think about money and value transfer of assets.
*   Its growing adoption and integration into the global financial system make it a significant player in the future of finance.
*   Understanding Bitcoin is crucial for staying informed about technological and economic advancements.

Thank you for reading my 2024 summer of bitcoin journey experience.
