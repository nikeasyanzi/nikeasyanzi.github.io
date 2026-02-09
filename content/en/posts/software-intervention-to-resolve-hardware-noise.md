+++
date = '2025-02-05T10:30:00+08:00'
draft = false
title = 'Software Intervention to Resolve Hardware Noise'
categories = ["System Engineering", "Problem Solving"]
tags = ["SSD", "Hardware Monitoring", "Pragmatic Engineering", "Customer Obsession"]
ShowToc = true
TocOpen = true
+++

## Introduction

In system engineering, it’s tempting to treat “technically correct” as the finish line.

But I’ve learned the hard way that correctness can still create a bad customer experience—especially when your software is sitting between messy reality (hardware, firmware, drivers) and the person who just wants a clear signal: *Is something wrong, and do I need to act now?*

This is the story of one uncomfortable moment that taught me a simple heuristic: sometimes, software has to absorb hardware noise.

## The Problem: A Cascade of False Alarms

We run an SSD monitoring daemon that reports wear-out metrics to customers. One day, a customer escalated a critical issue: their fleet was suddenly triggering SSD wear-out alarms at scale.

The timing couldn’t have been worse. Their release was measured in days, not weeks. And an “SSD health crisis” alert storm is the kind of thing that freezes decision-making—even if the systems are actually fine.

My first reaction was the classic engineer reflex: assume the bug is ours.

So I reviewed the code line by line. And frustratingly, it looked solid. We were reading the OS signals and reporting them exactly as designed.

Then we found the uncomfortable truth: **the signal itself was flaky**.

Underneath our daemon, a specific SSD firmware could produce false positives at a lower layer. And our software—being “correct”—was faithfully amplifying that flakiness into customer-facing alarms.

That created a dilemma I didn’t love, but couldn’t avoid:

- The implementation was **technically correct**, but it was **failing the customer** by producing noise instead of actionable information.
- This customer was the first to report the incident, so we didn’t have a playbook.
- The hardware vendor’s RMA process would take weeks.
- The customer’s release was in days.

## The Solution: A Heuristic Filter

We needed something fast, safe, and honest.

So I implemented a **heuristic filter**—a debouncing approach.

In plain English: instead of alarming the moment we see a single “wear” signal, we require the signal to persist across multiple reads (or across a short time window). If the signal is transient and disappears on the next check, we treat it as noise. If it stays, we treat it as real.

The key tradeoff is latency. Our daemon checks the signal every few seconds, so “confirming” persistence can delay an alert by seconds (sometimes minutes, depending on the window).

For SSD health monitoring, that slight delay is a good deal. Wear-out isn’t a millisecond-scale emergency; it’s a trend. In this context, it’s better to report a slightly delayed, trustworthy alert than to flood users with spikes that they can’t act on.

I aligned with my manager on the framing:

*This is a pragmatic patch to unblock the release. We’re not hiding the problem—we’re dampening noise while we drive a permanent vendor fix.*

## The Result: Unblocked and Learned

The patch deployed smoothly. The false alarms stopped immediately. The customer shipped on schedule.

That said, the deeper issue was (and still is) a mystery. We still hadn’t identified the culprit stage in production that led to the flaky behavior. I asked our drive team to continue the follow-up and push for a root-cause diagnosis.

The bigger lesson for me was not about SSDs—it was about what “correctness” means in the real world:

**“Technically correct” doesn’t automatically mean “works for the customer.”**

If our job is to help customers operate systems confidently, then our job isn’t just to pass signals through. It’s to translate noisy reality into something stable, explainable, and actionable.

---

Have you ever had to choose pragmatism over perfection to protect a customer experience? I’d love to hear what you did—and what you learned.
