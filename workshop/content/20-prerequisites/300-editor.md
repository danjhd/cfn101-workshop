---
title: 'Install a code editor'
date: 2019-10-18T14:28:46+01:00
weight: 300
---

You may use any code editor or IDE of your choice that supports editing [YAML](https://yaml.org/) but for this workshop, we will assume the use of [Visual Studio Code](https://code.visualstudio.com/) as it works well on macOS, Linux, and Windows.

To install Visual Studio Code, use your operating system's package manager (e.g. `brew cask install visual-studio-code` on macOS) or follow [the instructions on the VS code website](https://code.visualstudio.com/).

## Python

We will be using some plugins and tools to help improve the experience of creating CloudFormation files using Visual Studio Code. These tools require [Python](https://www.python.org/) to be available on your system.

If you do not already have Python please install this first by downloading it from [here](https://www.python.org/downloads/).

{{% notice info %}}
If you are unable to install Python you can continue with this workshop, however, you may find the lack of some of the features frustrating when trying to debug your files.
{{% /notice %}}

## CloudFormation Linter

We recommend you install the [AWS CloudFormation Linter](https://github.com/aws-cloudformation/cfn-python-lint).  A [linter](https://en.wikipedia.org/wiki/Lint_(software)) will proactively flag basic errors in your CloudFormation templates before you deploy them.

Once the linter is installed you should install the [cfn-lint](https://marketplace.visualstudio.com/items?itemName=kddejong.vscode-cfn-lint) plugin. To allow Visual Studio Code to integrate with it.

{{% notice tip %}}
If you are a first time user to Visual Studio the community for plugins is extensive and therefore you may also find other plugins that you find useful we suggest you take a look after this workshop if you have enjoyed using it.
{{% /notice %}}

