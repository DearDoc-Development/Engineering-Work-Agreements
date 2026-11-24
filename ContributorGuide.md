# Contributing to DearDoc: one page

Owner: Project Manager and QA Lead (named on the [release protocol page](https://github.com/DearDoc-Development/deardoc-engteam-wiki/blob/main/docs/team/processes/release-protocol.md)). Last reviewed: 2026-11-24.

You're helping build healthcare software. These steps keep patients' data safe and keep your work moving. If anything is unclear, ask your **sponsor**, the DearDoc engineer named on your ticket.

This page follows the [release protocol](./README.md#release-protocol). If the two ever differ, the protocol wins.

## Three things you must never do

1. **Never use real patient data.** Not in an AI tool or prompt, a ticket, a pull request, a screenshot, or a test file. Use made-up names and numbers.
2. **Never paste passwords, keys, or tokens** anywhere: code, chat, or AI tools.
3. **Never work without a ticket.** No ticket, no pull request.

## Your change, step by step

1. **Start from the ticket.** It tells you what's needed and what the result should look like. If it doesn't, ask before you start. Only a ticket tagged `#fix` is high priority: it must be live within 8 hours or by the end of the day, so start now and tell your sponsor.
2. **Write a short plan** in the ticket: which files you will change, and how to undo it. For a `#fix`, three lines are enough: the change, the expected result, and the rollback. You only need to wait for approval if your change touches one of the restricted areas below.
3. **Create a branch from `staging`.** Name it `<type>/<ticket-id>-<slug>`, like `feat/1234567890-new-button`.
4. **Keep it small.** One ticket, one change, under 2,000 changed lines. Don't fix other things you notice; tell your sponsor.
5. **Open a pull request to `staging`** and fill in every section of the template. "I didn't test this" is an acceptable, honest answer.
6. **Read Copilot's review.**
   - **Clean or Low:** you can merge to `staging`.
   - **Medium:** fix it, or reply explaining why not. Then merge.
   - **High:** you must fix it. Copilot reviews again, and a teammate approves.
   - If you changed how something works (not only text, styling, layout, tests, or docs), your sponsor reads your code and approves before it merges.
7. **Check it on staging.** End-to-end tests run on staging after every merge. For now it is checked by hand. Your sponsor checks it on staging by hand and records it in the pull request. Check it together: does it do what the ticket says, and does everything around it still work?
   - If it fails, your change is **reverted from `staging`** so it doesn't hold up everyone else. Fix it on a new branch from `staging` and open a new pull request.
8. **After merge, it waits for the daily release.** Your change goes to production together with everything else on `staging`. The Release Captain opens the release pull request to `main`, and an engineer with no change in it checks and merges it. You don't need to do anything. A `#fix` is released as soon as it passes.
9. **After it's live, you get a message** with the version tag and your tickets. Within 30 minutes, check your change in production through the UI, as a user, and note it on the ticket. Your sponsor confirms it was done.

## Working with AI tools

- Use only approved AI tools. Ask your sponsor for the approved list.
- Read what the AI wrote before you submit it. You're responsible for it.
- Say in the pull request how much was AI-generated. Nobody minds; reviewers just need to know.
- If the AI invents a function or package that doesn't exist, the checks will fail. That's normal. Ask it to use what's already in the codebase.

## Restricted areas need a Code Owner

Login and permissions, security rules, secrets, payments, anything that sends patient data somewhere, database migrations, deploy and CI settings, AI instructions, new outside services or packages, the AI chat, and telephony. If your change touches one of these, stop and tell your sponsor first. A Code Owner approves the plan before you write code, and approves both pull requests.

## If something goes wrong

Tell your sponsor straight away. You can't break production from where you work, and a quick heads-up is always better than a quiet fix.

More: the [release protocol](./README.md#release-protocol) and the [Engineering Work Agreements](./README.md).
