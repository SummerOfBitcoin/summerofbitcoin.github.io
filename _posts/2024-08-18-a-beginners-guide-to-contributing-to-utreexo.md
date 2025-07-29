---
layout: post
title: "A Beginner's Guide to Contributing to Utreexo"
date: 2024-08-18
author: "Njokom Nshanui"
categories: [Tutorials, Utreexo]
---

Have you ever come across an exciting open source project which you would love to make a contribution to, but you have no idea on how to start? Or perhaps you are a relatively experienced developer (Or just someone who's more verse with open source) who's looking to contribute to an exciting open source project? Well, I have an answer to your question on “How do I start contributing to open source”.

Technically, I won't be answering that question in the general sense. This article will be more inclined towards a specific open source project, Utreexo. So, I guess this post aims rather, to answer the question “How do I start contributing to Utreexo?”.

**So What if Utreexo in the first place?**

Well, if you are here, I'm assuming that you have heard about Utreexo from somewhere, or at least you know that it is a project related to Bitcoin. If you have never heard of it however, that's still fine, I will try to break it down more, and also provide relevant links, however, it is highly recommended to have a high level knowledge on how Bitcoin works, in order to better understand this article.

Anyways, to answer the question above, let's give a little background of what Utreexo aims to achieve first.

The Bitcoin network as we know it, (or… don't know it yet) stores records of all transactions in a digital ledger, known as the Blockchain. This ledger contains records of all the transactions that have ever been carried out on the network. The current state however of the network does not necessarily contain the history of everything that has ever happened on the network, rather, it consists of the set all the Unspent Transaction Outputs (UTXO set). This set holds information about “which wallet has what”. In simpler terms, the UTXO set basically just contains records of how much bitcoin a Bitcoin wallet has in their account. Well, the last sentence isn't very accurate because our Bitcoin wallets usually have severally Unspent transaction outputs, which when combined, reflects as our wallet balance, and what is available to spend.

The problem is, this Bitcoin state is growing faster and larger as the number of transactions on the network increases. This poses a significant scalability issue, as the larger size requires larger hardware resources, which can therefore dissuade some people from operating Bitcoin nodes among other disadvantages.

That is where Utreexo comes in.

So we ask again, “What is Utreexo”?. Well, Utreexo is a method for greatly reducing the storage needed to run a fully validating node by proposing a hash based dynamic cryptographic accumulator, and introducing a Compact state node which stores only an accumulator representation of the state. These nodes require additional inclusion proofs from the sender, before they are able to verify transactions, With Utreexo, the current state of the network is represented in a significantly smaller size, which means less resource usage, which will then encourage pretty much anyone to run a Node, conveniently. You can the read more about Utreexo from the following resources ELI5: Utreexo — A scaling solution, or Improving the Bitcoin network using Utreexo, or download the Utreexo white paper from here.

**Setting up your development environment**

Now that we know a little bit more about Utreexo, let's dive in to setting up our development environment.

Utreexo is written in Go, so having a basic knowledge and understanding of Go programming language will be an added bonus. However, if you are new to Go, you can check out these resources to know more about the language, and to understand a few basic concepts of the language Effective Go, Learn Go.

I use Visual studio code for this post, but feel free to use any text editor of your choice.

First of all, we will need to install Go programming language. The steps to install will not be covered here, but you can check out this article, detailing how to install and run Go programs. Download and Install Go. Once you've finished installing Go, as per the article and ensuring that everything works, by checking the output of `go version` we can proceed to cloning the codebase. Having a basic knowledge of git will be beneficial here. First, we need to have git installed on our computer system. To check if git is installed, you can open a new terminal or powershell window and type `git –version` if git is installed, you will see an output with the version number. If not, you have to proceed to install git using directives from Installing Git.

Afterwards, you can try the the command `git –version` again. If git was successfully installed, we can now proceed to forking and cloning the repository.

Forking the repository basically means creating a copy of the repository on our personal GitHub accounts. Changes we make thereafter will be pushed to our forked repository, and then we can submit a pull request.

To clone the repository, we navigate to the desired location and open in terminal. Or we can navigate directly using the terminal.

For instance, let's say we have a folder on our desktop called "Projects" which we want our Utreexo code to reside in, we simply navigate to desktop, and then open the “Projects” folder, and inside there, we right click and take “Open in terminal” for windows and Ubuntu. Or we can navigate directly inside our terminal window

First let's ensure we are in the root directory of our system by navigating there.

`Cd ~`

And then

`cd Desktop/Projects`

When in here, we need to clone the repository. There's a slight consideration to be made here. When you visit the utreexo main organisation on GitHub, we have several repositories. One of which is the utreexo repo, which contains the actual dynamic accumulator implementation, and then there is the utreexod repo, which is a bitcoin full node implementation, that supports Utreexo. We will be focusing on the latter, that is, the bitcoin full node implementation with Utreexo support. Let's proceed to fork and then clone the repository

To fork the repository, we go to the utreexod repo github page, and on the far right, we will find the “fork” button.

We need to click on this, and follow the onscreen instructions. Once done, our personal github accounts will have the same copy of this reposirory.

Now we can proceed to clone the repository from our personal account

`git clone https://github.com/yourgithubusername/utreexod.git`

When it's done, we can open the folder with our text editor, and proceed to install all dependencies as described in the description of the GitHub repo. Now we're in.

From here, we will want to spend some time, understanding the various sections of the code base, and what each section is responsible for. Once you have a broad idea of the code base, you can go ahead and check the Issues section of the repository to find any issue you would love to tackle, or perhaps a feature suggestion.

Once identified, we create a new branch and name the branch properly

`git branch feature1`
`git checkout feature1`

Here, “feature1” is the name of the feature we are going to be working on. Give it a more descriptive name.

Once we are done working on our feature and feel it is ready, we commit and push to github, and then raise a pull request.

To commit, we first of all stage the changes made using

`git add .`

And then proceed to commit with

`git commit -m “Commit message”`

The commit message should be a short description of the changes made and it should be inside quotes as mentioned above.

If you are using vscode, the git extension really simplifies all of these processes.

If you need more resources on how to commit your changes and push then to github, check this resource, and for creating a pull request, you can also check here.

After our pull request has been made, we then wait for updates from maintainers, and once everything has been sorted, and probably more changes made requested, the Pull request will be merged, and… there you go. You have Successfully contributed to Utreexo

Open source contributions like this are a driving source of major software, making contributions to open source is very important, as you make changes to software used by tens of thousands of users out there. That contribution brings a high level of satisfaction. From here, it is advisable to continue building relationships with other community members and maintainers and continue contributing to the software on a regular basis.
