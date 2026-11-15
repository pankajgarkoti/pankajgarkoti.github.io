---
layout: default
title: "mAGI 0.3: the terminal that is already there, and notes that keep themselves"
---

# mAGI 0.3: the terminal that is already there, and notes that keep themselves

![mAGI](https://raw.githubusercontent.com/pankajgarkoti/magi/v0.3.0/docs/logo/logo-colour-light.svg)

*mAGI 0.3.0 · November 2026 · [source](https://github.com/pankajgarkoti/magi) · [release](https://github.com/pankajgarkoti/magi/releases/tag/v0.3.0)*

mAGI is a Linux desktop where the main way to use the computer is to talk to one AI agent in a terminal. It starts other agents for separate jobs and keeps track of them. Version 0.2 gave that agent a way to hear about things without checking, and a browser it shares with you. Version 0.3 is about not losing your place.

## What changes when you log in

You log in and a terminal is already open. It is not a fresh shell: it is attached to one long-running session called `main`. Open another terminal with SUPER+Return and you get a second view of the same session, not a new one. Close every terminal window and nothing stops; the programs and agents keep running, and the next window you open shows them again.

This matters more with agents than without them. An agent working on something for an hour lives in a terminal window. In most setups, closing that window, or the desktop crashing, ends the work. In mAGI the window is only a view.

## What happens after a reboot

Every minute mAGI saves the layout of your sessions: which windows exist, which folder each one is in, and what is running in it. When you log in again it rebuilds that layout before the terminal opens, so the terminal attaches to your windows, not to an empty shell next to them.

Agents come back where they left off: Claude Code, opencode and Codex each have their own way to reopen a conversation, and mAGI uses it. Other programs are started again with the command that started them. Anything else comes back as a shell in the same folder.

In the test machine, a window running a program in a project folder came back after a reboot in the same folder, running the same program, with no action from the user.

## Notes that keep themselves

There is now a `notes` folder in your home directory with one note per day, in a plain Markdown layout: Tasks, Notes, Logs, Scratch. You can write in it like any notes app. The agent writes in it too:

- **Open threads.** When something is left open (a decision you haven't made, work that is parked), the agent adds a checkbox for it in today's note. Tick it when it's done. Every few hours the agent is reminded of the ones still open, so nothing quietly drops.
- **What you said out loud.** If you use the voice listener, every three hours the things you asked for are turned into a short task list in that day's note.
- **A log.** Decisions you made, things that shipped, and agents that finished get one line each.

The folder is a git repository that only exists on your machine. It is saved automatically every fifteen minutes; nothing is uploaded anywhere. The scripts that run the agent's housekeeping are kept in the same folder, so your notes and the setup that writes them travel together: copy the folder to another mAGI machine and one script links everything back into place.

## Smaller changes

- Your terminal and tmux settings are part of the system image, including the status bar and its plugins, so nothing has to be downloaded on first login.
- SUPER+SHIFT+Return opens a plain terminal when you want one outside the session.
- A first login that took too long could drop you back at the login screen (also in 0.2). The desktop now starts first and finishes setting itself up in the background, and if it still fails to start it takes the screen back and tries again.
- If you already use 0.2, your open threads and the agent's records move into the notes folder on first login.

## What was tested

On a test virtual machine, with a stand-in for the AI agent so no account was needed: logging in opened the terminal attached to `main` on the first workspace; a second terminal joined the same session without creating a new one; the daily note was created and written to by the agent's commands; and after a reboot the saved windows came back with their folders and programs. The tests that shipped with 0.2 (the agent layer and the event bus) still pass.

## What's next

- **Making the agent's day visible.** The daily note already records what the agent did. The next step is a short summary at the top of each day: what was finished, what is waiting on you.
- **Moving between machines.** The notes folder can already rebuild the setup. Next is doing that from the installer, so a new machine starts with your notes and sessions.
- **Fewer things to set up.** Signing in to the agent is still a separate step at first login. The aim is to make the first conversation the setup.

Download, checksum and release notes: [github.com/pankajgarkoti/magi/releases/tag/v0.3.0](https://github.com/pankajgarkoti/magi/releases/tag/v0.3.0)
