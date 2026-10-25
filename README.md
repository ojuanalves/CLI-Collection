# Troubleshoot Collection

A troubleshooting Knowledge Base, grouped by vendor and by topic. It gathers CLI commands and step-by-step instruction documents for several manufacturers and platforms, mainly Fortinet, Microsoft Windows, Linux and web browsers.

Each document is meant to be opened in the middle of an incident: find the symptom, copy the command or follow the guide, and check the warning before you press Enter.

## Contents

- [What this repository is](#what-this-repository-is)
- [How the documents are written](#how-the-documents-are-written)
- [Fortinet](#fortinet)
- [Using the collection](#using-the-collection)
- [Rendering notes](#rendering-notes)
- [Safe use](#safe-use)
- [Adding or fixing a command](#adding-or-fixing-a-command)
- [Sources and credits](#sources-and-credits)
- [Disclaimer and rights](#disclaimer-and-rights)

## What this repository is

This is a Knowledge Base for troubleshooting. It holds two kinds of content: CLI command references and instruction documents that explain how to diagnose a problem. It gathers what vendors and communities already publish, puts it in a consistent format, and adds short notes on when to use each piece.

It is not original vendor documentation, and it does not replace it. If a command here behaves differently from what the vendor documents for your version, the vendor documentation is right and this repository is out of date.

The repository does three things with the commands:

1. Groups it under the problem it helps with (routing, sessions, VPN, DNS and so on), so you search by symptom instead of by feature name.
2. Shows a concrete example next to the generic syntax.
3. Flags the commands that can interrupt traffic, restart a service, or expose a secret.

## How the documents are written

All documents follow the same conventions, so you only have to learn them once.

**Placeholders.** Anything in angle brackets is a value you replace, such as `<ip>`, `<port>` or `<vpn-name>`. Square brackets mean the argument is optional.

**Examples under the block.** The command sits alone in a code block, so the copy button grabs only the command. The example follows on a line starting with `Ex.:`.

```fortios
diagnose firewall iprope lookup <src-ip> <src-port> <dst-ip> <dst-port> <protocol> <src-interface>
```

Ex.: `diagnose firewall iprope lookup 10.210.14.139 0 57.144.179.32 443 6 port3`

**Risk icons.** Two icons mark commands that deserve a second look:

| Icon | Meaning |
|:---:|---|
| 🔴 | Disruptive. Drops sessions, tunnels or users, restarts a process, or exposes a secret. |
| 🟡 | Debug or configuration change. Loads the CPU or changes the config, so stop or revert it when you're done. |

A command with no icon only reads information.

**Alert boxes.** Warnings use GitHub alert blocks (`CAUTION`, `WARNING`, `IMPORTANT`, `NOTE`) so the dangerous ones stand out when you scroll. Red boxes mean a command can cause an outage. Yellow boxes mean it affects a service or leaves something running.

**Sentence case and plain English.** Descriptions are in English, kept short, and say what the command shows or does.

## Fortinet

The Fortinet documents cover FortiGate running FortiOS.

### FortiGate CLI cheat sheet

`FortiOS-FortiGate.md` is organized by problem, not by command family. Every command has a short description and an example, and the disruptive ones carry a warning.

### Route Process

`Route Process.md` explains how a FortiGate decides where a packet goes: the order of policy routes, ISDB routes, SD-WAN rules, the route cache and the FIB, and what each stage does on a match or a miss. It is based on the Fortinet Community article on the route lookup process.

The flowchart (`route-process.svg`) is animated. Each colored dot is one packet taking one possible path, and the dashed lines move in the direction the lookup runs. The document also includes a static Mermaid version for viewers that do not play SVG animation, followed by a legend that covers every box in detail, including reverse path forwarding, the "stop policy routing" action, and administrative distance.

## Using the collection

1. Open the document for your platform and use the contents list or your browser's search to jump to the problem.
2. Read the one-line description above the block. It says what the command shows.
3. Replace the placeholders with your own values.
4. Look for a risk icon or an alert box before you run it.
5. Run the example first on a lab device or a test host when you can.

To search the whole repository from a terminal:

```bash
grep -rn "session clear" .
```

To find every disruptive command at once:

```bash
grep -rn "🔴" .
```

## Rendering notes

The documents are written for GitHub and read best there. Other viewers show most of the content, with some differences.

- **Alert boxes** need GitHub (or a viewer that supports GitHub alerts). Elsewhere they appear as plain quoted text, and the warning wording still reads correctly.
- **The animated flowchart** needs the SVG file next to `Route Process.md`, because the document loads it by relative path. If it does not move, open `route-process.svg` directly in a browser.
- **Section links** in the cheat sheet start with a hyphen (for example `#-sessions`) because the headings begin with an icon. If a viewer builds anchors differently, use the contents list or search instead.
- **Code blocks** are tagged `fortios` for FortiGate commands. Viewers that do not know that language show them without highlighting, and the copy button still works.

## Safe use

These commands touch production equipment. A few habits avoid most of the damage:

- Read before you run. Check the risk icon and the alert box.
- Filter before you debug, and stop the debug when you have what you need. A forgotten debug keeps using CPU.
- Use a maintenance window for anything marked disruptive.
- Test on a lab device first when you can.
- Keep a configuration backup before you change anything.
- Never paste secrets, such as pre-shared keys, into tickets, chats or screenshots.

## Adding or fixing a command

A change is welcome when it follows the existing format. Use this shape:

````markdown
**What the command does** (short note on what you see in the output)
```fortios
command <placeholder> [<optional>]
```
Ex.: `command 10.0.0.1`
````

Rules that keep the documents consistent:

- Put the command alone in its block, and the example on the line below it.
- Write descriptions in English, in plain sentences.
- Add 🔴 or 🟡 to the label if the command is disruptive or changes state, and add an alert box that says what happens.
- Name the version if the command is not available everywhere.
- Say where the command comes from. See the next section.
- Avoid copying long passages from vendor documentation. Link to the source instead.

If you find a command that no longer works or behaves differently on your version, open an issue with the product, the version, the command and what you saw.

## Sources and credits

The commands in this repository come from the sources made available to the project: vendor documentation, vendor knowledge base articles, community cheat sheets and field notes. The repository only groups them. It does not claim authorship of the underlying commands or of the vendors' documentation.

Sources used so far for the Fortinet documents:

- [Technical Tip: Routing in FortiGate (route-lookup-process)](https://community.fortinet.com/fortigate-3/technical-tip-routing-in-fortigate-route-lookup-process-96148), Fortinet Community
- [Technical Tip: How to check why automation stitch is not working as expected](https://community.fortinet.com/fortigate-3/technical-tip-how-to-check-why-automation-stitch-is-not-working-as-expected-92733), Fortinet Community
- [Fortinet Document Library](https://docs.fortinet.com/), FortiOS administration guides and CLI reference
- [Fortinet Community knowledge base](https://community.fortinet.com/knowledge-base), articles on SD-WAN, DNS, IP pools and VPN diagnostics
- A FortiGate troubleshooting cheat sheet and field notes shared with the project

This list is not exhaustive. Where a document leans on a specific article, it links to it.

## Disclaimer and rights

**All rights belong to the owners of each manufacturer, vendor and product.** Fortinet, FortiGate, FortiOS, FortiClient, FortiGuard, Microsoft, Windows, PowerShell, Linux and every other product name, logo and trademark mentioned here belong to their respective owners. This repository is not affiliated with, endorsed by or sponsored by any of them.

**The commands were taken from the sources made available, listed above. This repository only groups and organizes them.** It does not create the commands, and it does not own the content of the documentation they come from.

**There is no responsibility for changes made to your environment.** Commands can interrupt traffic, drop sessions, restart services, change configuration or expose sensitive data. The content is provided as is, without warranty of any kind, and it may be incomplete, outdated or wrong for your version.

**Use at your own risk.** You are responsible for checking each command against the official vendor documentation, testing it, and deciding whether it is safe to run in your environment.
