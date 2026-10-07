# Jigsaw

Publish a small app your agent built, then choose who can open it, like sharing a doc.

Jigsaw is a cloud for small software. Claude builds a small web app, you say "publish this to Jigsaw", and it goes live at its own address. You add people by email as viewers or editors, and Jigsaw signs each of them in before the app loads. The app itself contains no login code.

## What this plugin adds

- **The Jigsaw connector**, a remote MCP server at `https://jigsawapps.com/mcp`, with three tools:
  - **Publish an app**: publishes or updates an app from the files Claude passes.
  - **Get an upload link**: for when Claude can run shell commands. It returns an upload address and a ticket that works for 10 minutes, for one app.
  - **List apps**: lists the apps you own or have been given access to.
- **A skill, `publish-to-jigsaw`**, which tells Claude how to build an app that works on Jigsaw the first time: how visitors are signed in, how to store shared data, and how to call an outside API without putting a secret key in the page.

## Use it

1. Install the plugin. The first time Claude uses the connector, a Jigsaw sign-in page opens. Sign in with your email.
2. Ask for an app, then ask Claude to publish it:
   - "Build a shift rota for my team and publish it to Jigsaw as shift-rota."
   - "Publish the new version."
   - "Which apps do I have on Jigsaw?"
3. Open the sharing address Claude gives you and add people by email.

## What it sends, and where

- The plugin talks to `https://jigsawapps.com` and nothing else.
- When you ask Claude to publish, the app's files are sent to Jigsaw. With an upload link, Claude writes the command itself for your machine: it zips the app's folder, leaving out `node_modules` and `.git`, and posts the zip to `https://jigsawapps.com/api/publish` with the ticket. Claude asks before it runs the command.
- The plugin has no hooks, no local server and no scripts. Nothing runs when you install it.

## Good to know

Every Jigsaw app is private: only you and the people you add by email can open it. There are no public links. For a public website that anyone can open without signing in, use a website host instead.

## Links

- [How an agent connects, builds and publishes](https://jigsawapps.com/agents)
- [Pricing](https://jigsawapps.com/pricing)
- [Privacy policy](https://jigsawapps.com/privacy) and [terms of service](https://jigsawapps.com/terms)
- Support: team@jigsawapps.com
