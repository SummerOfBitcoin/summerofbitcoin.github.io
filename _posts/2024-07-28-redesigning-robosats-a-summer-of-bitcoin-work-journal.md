---
layout: post
title: "Redesigning RoboSats: A Summer of Bitcoin Work Journal"
date: 2024-07-28
author: "Sahil Mandi"
categories: [Stories, Robosats]
---

This journal documents the UI redesign of RoboSats, a peer-to-peer (P2P) Bitcoin marketplace that prioritizes privacy and ease of use through the Lightning Network.

---

### **ROBOSATS UI REDESIGN**

#### **INTRODUCTION**

The project addressed several UI issues in the RoboSats platform to improve user experience and visual appeal without altering core functionality. The main goals were to enhance navigation, make better use of screen space, and create a more professional aesthetic. The redesign provides clear actions for new users ("Start"), advanced users ("Recover"), and those who want a quick setup ("Fast Generate"). A key feature is a bottom navigation bar, designed for ease of use on mobile devices.

**Notable Issues with the Original Design:**
*   An asymmetric bottom navbar with a cramped profile button.
*   Navigation icons in the mobile view lacked labels.
*   The interface felt empty and lacked visual interest.
*   The overall simplicity gave it an unprofessional appearance.

---

### **15 July 2024**

#### **NAVIGATION REDESIGN**

*   **Problem:** The original navbar was functional but asymmetric and cramped. It was designed for mobile but neglected desktop users, and its unlabeled icons could cause confusion.
*   **Solution:** A new top navigation bar was introduced for secondary options and the profile link. This change allowed for a symmetric, three-item bottom navbar with the "Create" action centered. Icons were given labels for clarity, and separate layouts were developed for desktop and mobile to optimize the experience on each.

#### **SCREEN SPACE UTILIZATION**

*   **Problem:** The original design, especially on desktop, underutilized screen space, making it feel empty.
*   **Solution:** A new side-by-side layout was adopted for the Welcome page, inspired by Google's sign-up screens. Component widths were increased, and a "floating island" effect was applied to the navigation bars to fill the space more effectively.

#### **VISUAL APPEAL AND BRAND IDENTITY**

*   **Problem:** The design lacked a strong visual appeal and brand identity.
*   **Solution:** Drawing inspiration from Neo Brutalism, the redesign incorporates bold typography and shapes. This approach fills empty space and establishes a unique and memorable visual identity for RoboSats.

---

### **OTHER PAGES**

#### **ONBOARDING PROCESS IMPROVEMENT**

*   **Problem:** The original onboarding process could overwhelm users by presenting too much information at once.
*   **Solution:** A new progress viewer was implemented to display only one step at a time. This conditional rendering helps users focus on the current task without feeling overwhelmed.

The initial set of changes was reviewed and approved by the RoboSats team via Pull Request #1388 on GitHub.

---

### **28 July 2024**

#### **FOLLOW UP!**

After the home page redesign was completed, the same minimal and neo-brutalistic design language was extended to the other pages of the application to ensure a consistent user experience.
