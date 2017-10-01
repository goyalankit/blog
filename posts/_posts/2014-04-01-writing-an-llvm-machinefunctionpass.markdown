---
layout: post
title: "Rriting an LLVM MachineFunctionPass"
date: 2014-04-01 00:18
comments: true
published: false
categories: 
---

Recently I have been working a lot with LLVM. I wanted to write a
[MachineFunctionPass](//llvm.org/docs/WritingAnLLVMPass.html#the-machinefunctionpass-class), but I couldn't find a good documentation to do that. LLVM has a pretty good [tutorial](//llvm.org/docs/WritingAnLLVMPass.html) on how to write a
general purpose pass. However MachineFunctionPass is a **code
generator**
pass which are not registered and initialized like a normal pass.



