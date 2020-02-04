---
title: 'Layered Stacks'
date: 2019-10-22T12:21:25+01:00
weight: 20
chapter: true
---

# Layered Stacks

Layered stacks are a different method of managing common or re-usable components in CloudFormation. The nested stack option results in child stacks that can only be referenced by the parent stack. With Layered stacks the same referencing is possible but also references can be made from multiple stacks. You are not limited to a single parent.