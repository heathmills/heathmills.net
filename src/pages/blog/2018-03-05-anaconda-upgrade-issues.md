---
contentType: blog
path: /anaconda-upgrade-issues
title: Avoiding update issues in Anaconda from 5.01 to 5.1
date: 2018-03-05T17:12:33-05:00
---
I decided to update my Anaconda3 distribution today on my Windows 10 laptop and ran into a number of issues. I am posting my solution here to help others who may come across similar issues.

Before updating, I had Anaconda version 5.01 installed on my machine. I was attempting to update using ``conda update anaconda`` and ran into a failure. My first guess was there was an error in the dependency linkage between certain conda packages in the long list of packages being updated. I tried to bypass this by updating a number of specific packages individually (python, conda, intel-openmp, jpeg, jupyter, jupyterlab, matplotlib, pandas, scipy, ipykernel, ipython). While my hunch was on the right track, I didn't account for all of the issues that could possibly arise.

I attempted ``conda update anaconda`` again and got a similar failure. Therefore, I turned on debugging on using ``-v`` and found the following error:

```
ERROR conda.core.link:_execute(481): An error occurred while installing package 'defaults::ipykernel-4.8.0-py36_0'.
LinkError: post-link script failed for package defaults::ipykernel-4.8.0-py36_0
```

I entered the string above in Google and found the following [GitHub issue](https://github.com/ContinuumIO/anaconda-issues/issues/8087).

Apparently, the issue is specific to Windows in Anaconda3 builds and revolves around trying to link/unlink the old and new versions of ipykernel and sphinx in the same update. When I first attempted to update anaconda from 5.01 to 5.1, my system was attempting to upgrade both packages to a newer version. I then had upgraded ipykernel to its latest version as part of the packages that I updated individually, but this was a newer version than the version of the package in the Anaconda 5.1.0 distribution. So when I ran 'conda update anaconda' again, I ran into a similar issue: the update command was still attempting to unlink and link both ``sphinx`` (upgrade from 1.6.3-py36h9bb690b_0 --> 1.6.6-py36_0) and ``ipykernel`` (downgrade from 4.8.2-py36_0 --> 4.8.0-py36_0).

A simple fix for this issue was already found in the comments. That option involved running the following three commands in order:

```bash
conda update conda
conda update ipykernel
conda update --all
```

While this does fix the installation issue, I have found from previous experience that using ``conda update -all`` is not the best strategy for your main anaconda installation. The Anaconda/Continuum Analytics team spends considerable time testing that the specific builds of the packages used in official Anaconda distributions all work together properly. Updating individual packages outside of the official releases can create a new set of headaches when you are importing packages into your code. Therefore, I always try to keep my default distribution using an official anaconda distribution and I create any customization in specific [environments](https://conda.io/docs/user-guide/tasks/manage-environments.html).

So I decided to fix my problem by installing a the specific version of ``ipykernel`` (4.8.0) that is used in Anaconda version 5.1.0. To install a specific package using conda, use the following command:

```bash
conda install [package]=[version]
```

After installing the needed version of ipykernel, I was able to update the rest of my Anaconda distribution succcessfully.

TL;DR for anyone having trouble updating Anaconda on Windows, take the following steps:

1. Find out which version of the ipykernel package your ``conda update anaconda`` command is attempting to use (for Anaconda version 5.1.0, this is ipykernel 4.8.0)
2. Run the following command:
   
   ``conda update conda``
3. Run ``conda install ipykernel=[version]``, which is the following command in this instance:

   ``conda install ipykernel=4.8.0``
4. Run the main update command again:
   
   ``conda update anaconda``