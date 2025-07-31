---
layout: post
title: "From Curiosity to Contribution: A Summer of Bitcoin Experience with Galoy CLI"
date: 2023-07-08
author: Eshan Vaid
categories: ['Stories', 'Galoy']
image: ../assets/images/blog_content/2023-07-08-from-curiosity-to-contribution-a-summer-of-bitcoin-experience-with-galoy-cli_d56b29e1.png
---

## From Application to Revelation

After applying for the “Summer of Bitcoin” program, I found myself inundated
with a wealth of learning resources from the SoB team. These materials delved
deep into the inner workings of Bitcoin, shining a light on how this
innovative cryptocurrency could potentially address the challenges of
traditional fiat money. Devoting hours each day to absorbing this newfound
knowledge, I quickly began to see Bitcoin not just as an investment asset, but
as a potential solution to a much broader financial problem.

## Learning and Beyond

As I delved into the wealth of learning resources thoughtfully provided by the
SoB team, I marveled at the exponential growth of my understanding of Bitcoin.
Each day, as I devoted hours to absorbing the intricate nuances, I watched my
grasp on this innovative cryptocurrency deepen. By the time I meticulously
crafted my application, I found myself equipped with a comprehensive
understanding of Bitcoin’s mechanics and far-reaching implications. The
application process, akin to a mini-adventure, provided me with a platform to
channel my fervor for Bitcoin. It was the moment where I not only expressed my
eagerness to contribute but also mapped out my envisioned path. And then, in a
pivotal moment, came validation — a testament to the dedication that had
fueled my journey. The acceptance email arrived, carrying not only the news of
my selection but also the weight of distinction. Amongst more than ten
thousand applications submitted globally, my application emerged as one of the
45 selected — an acknowledgment that underscored the significance of the
journey I was about to embark upon. With that email, the realization dawned
that I was on the precipice of immersing myself in a world that, until
recently, had existed on the fringes of my interests.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*NkxYEVtcAoxSBYNqBI95Ig.png"/>
</figure>

# galoy-cli

I found myself drawn to the Galoy organization — a realm dedicated to crafting
an intuitive Bitcoin banking platform. Amidst its offerings, my gaze settled
upon the Galoy CLI project — an endeavor aimed at streamlining interactions
with backend GraphQL APIs through a Rust-based CLI client. Guided by the
insightful mentorship of Sandipan Dev, I embarked on a multifaceted journey of
learning, contribution, and innovation. This project laid its foundation upon
the prerequisites of Rust and GraphQL. While my experience in Rust was modest,
my grasp of GraphQL stood firm, owing to my concurrent frontend development
internship at a startup.

<https://github.com/GaloyMoney/galoy-cli>

## Galoy — A Bitcoin-Native Banking Platform in the Cloud

At the heart of the Galoy organization lies a vision — to simplify financial
services on the Bitcoin platform for communities and institutions. A
cornerstone of this vision is the Galoy open-source Bitcoin banking platform.
This comprehensive platform comprises a secure backend API, mobile wallets,
compliance tools, and administrative controls. The impact of Galoy’s efforts
is exemplified by the Blink Wallet (formerly Bitcoin Beach Wallet) — a proof
of concept that catalyzed the establishment of a circular Bitcoin economy,
influencing the adoption of Bitcoin as legal tender in El Salvador. Within
Galoy’s ecosystem, interconnected layers such as Blink (Galoy’s mobile banking
platform app), galoy-web, stablestats, and more, collectively contribute to a
holistic and dynamic platform.

## Building Blocks of the Galoy CLI Project

As I began working on the Galoy CLI project, I found a basic structure with a
few initial features. However, it was clear that there was room for
improvement and new ideas. The project’s code design and structure needed some
organizing, which presented an opportunity for me to step in and enhance the
overall process.

My first focus was on improving the user login process, a key entry point to
the CLI’s functions. In the existing setup, users had to manually set an
authentication token as an environment variable after logging in. I saw a way
to simplify this. I redesigned the process to automatically store the
authentication token in a file on the user’s computer after a successful
login. This change removed the need for users to set the token manually and
made the login process smoother. I also introduced a logout command that could
delete the token file when activated, ensuring enhanced security and a better
user experience.

## Adding More Features

With a solid starting point in place, I turned my attention to adding various
features to the CLI. In just three weeks, I expanded its capabilities
significantly. Some of these features were planned ahead of time, like
_globals_ , _me_ , _intraledger_ payments, and _batch_ payments. Others were
developed on-the-go to make the CLI more user-friendly, similar to the mobile
app experience. For instance, I created commands like _set-username_ ,
_balance_ , _transactions_ , and _default-wallet_. These additions provided
users with a wide range of functions to interact with Galoy’s system.

## Design Restructuring for Future Growth: Addressing Extendability Challenges

As the first month of development drew to a close, we celebrated the creation
of a functional Galoy CLI project. However, our celebration was short-lived as
we encountered a critical challenge — our existing code structure posed
potential extendability problems. The absence of a unified design approach had
led to certain files becoming overly complex, potentially hindering future
development and maintenance efforts.

**Crafting a Solution: Redesign and Restructuring**

Recognizing the need for a robust foundation that could accommodate the
project’s growth, we took a proactive step — initiating a comprehensive
redesign and restructuring process. Our goal was clear: to create a design
that would readily embrace future features and enhancements without
necessitating substantial reworks. It was a pivotal phase in our journey,
focused on paving the way for a more sustainable and adaptable Galoy CLI.

The New Design: An Organized Layered Approach: The fruits of our labor
materialized in a refined code structure built upon three distinct layers:
**CLI, App, and Client**.

  * **CLI** Layer: Positioned as the user interface, this layer harnessed the power of Rust’s clap library to define commands and seamlessly accept user inputs. Its role was to be the interface through which users interacted with the CLI.
  * **App** Layer: This layer assumed responsibility for processing and handling inputs sourced from the CLI layer. A tapestry of modules such as auth, payment, and more was interwoven within this layer. Each module encapsulated specific sub-operations aligned with its designated function.
  * **Client** Layer: The communication hub of the project, the Client layer established a vital link with the Galoy backend. It took input variables originating from the App layer, initiated GraphQL queries and mutations, and subsequently relayed the backend’s response back to the App layer.

Beyond the intricate layers, another crucial aspect we integrated was the
error module within each layer. This module preemptively defined potential
errors that could arise during execution, ensuring that the project was
equipped to handle diverse scenarios with agility.

**Seeding for the Future**

The redesigned code structure with its organized layers was more than just a
technical triumph; it was a strategic investment in the project’s longevity.
By creating a cohesive and adaptable foundation, we positioned Galoy CLI to
evolve seamlessly with the ever-changing landscape of Bitcoin technology.

## Empowering the CLI

With a solid foundation in place, we embarked on a phase of dynamic expansion,
gearing Galoy CLI for a future of enhanced capabilities. Our primary aim was
to align the CLI with the Galoy app’s feature set, a mission that propelled us
to introduce a host of powerful functionalities.

Galoy CLI’s transformation into a versatile tool was marked by the addition of
essential features like **intraledger** payments, **batch** payments,
**lightning** payments, and more. These functionalities mirrored the Galoy
app’s offerings, ensuring that users could seamlessly replicate their
interactions on the CLI. The purpose was clear — to provide a comprehensive
and cohesive experience across both platforms.

# The Release

As the CLI matured with its feature-rich landscape, it was time to unveil our
creation to the world. The official release on Cargo marked a significant
milestone, presenting a fully-equipped Galoy CLI to the user community. Yet,
our aspirations didn’t stop there. Our sights were set on expanding
accessibility by releasing Galoy CLI on platforms such as _Chocolatey_ and
_Homebrew_ , ushering in a new era of user engagement.

Today, the Galoy CLI stands as a testament to its evolutionary journey. It
boasts a feature set that resonates with the Galoy app’s capabilities. Users
can seamlessly navigate their accounts, execute operations, and experience a
user-friendly interface akin to the app itself. It’s essential to underscore
that Galoy CLI is not intended to replace the Galoy app; rather, its key
function is to serve as a resource for developers seeking to interact with
Galoy’s backend. By using Galoy CLI, developers gain access to queries and
mutations otherwise inaccessible from alternative sources.

The development of Galoy CLI was a process of constant refinement and
enhancement. It started with a simple structure and grew into a functional
tool. This journey shows that progress comes from continuous improvement,
where each step builds upon the last. The story of Galoy CLI’s development is
a reminder that software development, like any creative process, benefits from
a balance of planning, adapting, and always looking for ways to make things
better.

# Contributing to galoy-cli and Installation Guide

Contributing to the Galoy CLI project is a rewarding journey that enables you
to shape the future of Bitcoin technology. Your efforts, whether big or small,
play a significant role in enhancing the CLI’s functionality and user
experience. Here’s how you can start contributing:

## Installation Guide

If you’re interested to dive into Galoy CLI’s development, the first step is
to have it installed on your system. The following methods provide you with
easy ways to do just that:

**Using Cargo:**  
For those with Rust and Cargo already installed, the installation process is a
breeze. Open your terminal and enter the following command:

    
    
    cargo install galoy-cli

This command fetches the latest version of Galoy CLI from
[crates.io](https://crates.io/) repository and installs it globally on your
system.

**Using Pre-built Binaries (Releases):**  
If you prefer pre-built binaries, follow these steps:

  1. Visit the [releases page](https://github.com/GaloyMoney/galoy-cli/releases/) of the Galoy CLI repository.
  2. Locate the latest release and navigate to the “Assets” section.
  3. Choose the binary that matches your operating system (e.g., _galoy-cli-x86_64-unknown-linux-mus_ for Linux).
  4. Download the binary.
  5. If necessary, make the binary executable by running the command:

    
    
    chmod +x galoy-cli

6\. Optionally, move the binary to a directory in your system’s PATH for easy
accessibility.

**Verification**

To ensure a successful installation, open a new terminal window and run:

    
    
    galoy-cli - version

You should see the version number of Galoy CLI displayed, confirming that it’s
ready for use.

**Customizing the API Endpoint**

You have the flexibility to customize the API endpoint Galoy CLI interacts
with. This is particularly useful for testing different environments or using
a local development instance of the Galoy Backend. For instance:

\- To switch to the staging environment, set the GALOY_API environment
variable to `<https://api.staging.galoy.io/graphql`>.  
\- To interact with a local instance, set the GALOY_API environment variable
to your instance’s URL, such as `<http://localhost:4002/graphql`>.

## Contributing to Galoy CLI

Contributions are the heartbeat of open-source projects, and Galoy CLI is no
exception. Your input, whether in the form of bug fixes, new features, or
improved documentation, is greatly valued. Here’s how you can contribute
effectively:

**Guidelines**

\- Maintain the project’s existing code style and conventions in your
contributions.  
\- Keep pull requests focused; consider separate PRs for addressing distinct
issues or features.  
\- Thoroughly document your changes and update relevant documentation.  
\- Introducing new features? Consider adding tests to maintain code quality.  
\- Your contributions align with the project’s mission — to create a powerful
and user-friendly tool for developers.

**Joining the Journey**

As you embark on your Galoy CLI journey, you’re not only contributing to a
thriving open-source community but also making a lasting impact on the world
of Bitcoin technology. Your expertise, creativity, and dedication are the
cornerstones that shape Galoy CLI’s evolution. Whether you’re a seasoned
developer or just starting, your contributions are valuable, and your journey
with Galoy CLI is just beginning.

See GitHub repository- <https://github.com/GaloyMoney/galoy-cli>

