# HyLoop Skill

**Ask your assistant what you actually said.**

HyLoop records on Apple Watch and transcribes on iPhone. With this skill, your Mac assistant can search the conversations you choose to share, find the moments you marked, and cite your own words with a timestamp when HyLoop labels them **Me**.

> “When we talked with the client last Wednesday, what delivery date did we agree on? What did I promise to send? Give me my exact words.”

## Get started

You’ll need HyLoop on iPhone with the **Mac** button next to **Watch**, a Mac running macOS 14 or later (Apple silicon or Intel), and an assistant that can install skills and run tools on that Mac. Keep the iPhone and Mac on the same local network.

1. **Send one setup message.** In HyLoop on iPhone, tap **Mac → Copy setup message** and paste it into the assistant you use on your Mac. It installs the Skill and starts pairing from that message. The temporary code is valid for ten minutes.
2. **Approve on iPhone.** Keep HyLoop open and tap **Approve** when your Mac appears.
3. **Choose what to share.** Select all eligible recordings or a recent range, then confirm **Start sharing**. Skill 0.1.4 and HyLoop iPhone build 36 or later include manual marks and the **Me** labels displayed in HyLoop. If sharing was already on before this update, confirm **Share marks and Me labels** once in the Mac panel. Use **Recording location** to share eligible area names. For health or activity questions, enable **Health and activity** in the Mac panel, and turn on the relevant sources and **Share with AI** under **Body & Location** on iPhone.

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

If macOS restricts access to a saved pairing, the tool reports the issue in chat without opening a Keychain password dialog. A pairing created by an older copy of the tool may need to be reset before reconnecting.

## For assistants

Use your local skill installation method to add the entire `hyloop/` folder. Keep `SKILL.md` and `scripts/hyloop` together and preserve the executable permission on the Mac tool. [Skill folder](hyloop)
