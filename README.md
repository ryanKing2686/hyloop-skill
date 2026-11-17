# HyLoop Skill

**Ask your assistant what you actually said.**

HyLoop records on Apple Watch and transcribes on iPhone. With this skill, your Mac assistant can search the conversations you choose to share, find the moments you marked, and cite your own words with a timestamp when HyLoop labels them **Me**.

> “When we talked with the client last Wednesday, what delivery date did we agree on? What did I promise to send? Give me my exact words.”

## Get started

You’ll need HyLoop on iPhone with the **Mac** button next to **Watch**, a Mac running macOS 14 or later (Apple silicon or Intel), and an assistant that can install skills and run tools on that Mac. Keep the iPhone and Mac on the same local network.

Connector **0.2.2** uses the updated iPhone app's four-digit connection check. Install it together with the matching iPhone update. Saved pairing entries are not deleted. Some Mac installations may need an explicit reconnect after updating the executable; see the Keychain note below.

1. **Install and check.** In HyLoop on iPhone, tap **Mac → Copy installation message** and send it to your local Mac assistant. The message contains the public repository link, without a pairing secret. The assistant installs the full Skill and checks the bundled connector with `doctor`.
2. **Compare four digits.** Keep HyLoop's Mac page open on the same Wi-Fi. The assistant runs `connect`, then displays the four digits in chat before starting the separate wait command. Compare them with your iPhone; nothing needs to be copied or typed.
3. **Allow the scope.** If the digits match, choose the recordings you want to share and tap **Allow connection and sharing**. If they differ, decline. Transcripts, marks, Me labels and eligible summaries are included; audio, location and health stay off unless enabled separately. Your assistant confirms whether the pairing was saved and the phone answered.

If several phones are found, expand **Connection options → This iPhone** to identify yours. A connection check lasts up to two minutes; after it expires, tap **Allow new connection** before trying again. The installation message itself does not expire.

The old `pair --stdin` command remains available only for explicitly connecting an older iPhone that still provides a temporary invitation. Updating the Skill does not add the new connection page to an older phone. Keep an existing working connection when updating; do not erase it to resolve a temporary network problem.

Use **Recording location** for eligible area names. For health or activity questions, enable **Health and activity** in the Mac panel, and turn on the relevant sources and **Share with AI** under **Body & Location** on iPhone.

With Skill 0.1.6 and iPhone build 38, choose **Area names and coordinates** under **Recording location** to include saved locations with transcripts. Asking for recent recordings uses the latest 24 hours by default. **Original audio** is off by default. Turn it on and ask the assistant to download today's recordings or another date's recordings as original M4A files.

## Try asking

- “What did I tell Alex I’d send after the meeting? Include the timestamp.”
- “Find the conversation where we discussed changing the launch date.”
- “What did I say around the moment I marked in yesterday’s meeting?”
- “Download today's original audio.”
- “Summarize the decisions from yesterday and quote the lines they came from.”
- “Was I walking during that call, and what heart-rate readings were available for that part of the recording?”

## How sharing works

Your Mac requests transcripts, recording marks, speaker labels, saved summaries, and permitted observations within the scope you approved. An original audio download saves a local copy of the requested recording part. Your assistant may send the information it reads to its AI provider.

HyLoop remembers the approved Mac and your sharing preference. Keep the iPhone unlocked with HyLoop open while you ask questions. Tap **Stop sharing** to end access.

The skill needs a local Mac tool. A cloud chat by itself cannot connect to the iPhone.

Updating the Mac connector may require reconnecting: macOS can deny the updated executable access to a pairing saved by the old one. Your iPhone recordings are unaffected, and the update does not automatically delete saved pairings. Keep a working installation until you are ready to reconnect.

If the tool reports a Keychain access error, it preserves the saved pairing and does not open a password dialog. Unlock the login keychain if it is locked. If access remains blocked and you choose to reconnect, follow the Skill's targeted forget/reconnect instructions. Do not erase pairings for a network timeout or treat installation as a successful reconnection.

## For assistants

Use your local skill installation method to add the entire `hyloop/` folder. Keep `SKILL.md` and `scripts/hyloop` together and preserve the executable permission on the Mac tool. [Skill folder](hyloop)
