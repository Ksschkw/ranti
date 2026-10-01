# Demo video script

Target: three to four minutes, one take, recorded from a phone and a laptop.
The goal is not to show every feature. It is to make one thing undeniable: this
assistant remembers, and you can prove it changed the answer.

Record with the phone in portrait for the Telegram sections and the laptop for the
widget and extension. Keep the clips separate and cut them together; a single
continuous take is not worth the retries.

Before you start, make sure:
- Cheta is deployed and healthy (open the portal and check it loads).
- You have at least ten memories stored on your account.
- You have said one durable fact in a recent session that the bot can recall.
- Notification sounds are off, and no personal messages are on screen.

---

## Shot 1 - the claim (0:00 to 0:20)

Screen: the portal at /app/portal.html, scrolled to the tagline.

Do nothing. Let the words sit on screen.

> "Every chatbot remembers. This one decides what to forget."

Cut.

---

## Shot 2 - it answers like any chatbot (0:20 to 0:45)

Screen: Telegram, a fresh chat with @Kosi_test_walrus1_bot.

Type: `/start`

The welcome banner appears with the buttons underneath. Do not tap anything yet.
Let the buttons be visible on screen for a beat.

> "This is Cheta. It answers questions like any assistant. That part is not
> interesting."

Cut.

---

## Shot 3 - it remembers (0:45 to 1:30)

Screen: the same Telegram chat.

Send these messages one at a time, waiting for each reply.

1. `I prefer an aisle seat when I fly`
2. `My final year project is about multi-agent reinforcement learning for TCP congestion control`
3. `I do not like talking about football`

Now close Telegram completely. Reopen it.

Type: `what do you know about me?`

Cheta answers and names the things you just told it, plus older ones it already
had.

> "I closed the app. There is no session here, no conversation history. It still
> knows me, because what I told it is stored in my own memory space on Walrus."

Cut.

---

## Shot 4 - the proof (1:30 to 2:20)

This is the shot that wins it. Do not rush it.

Screen: Telegram, the reply from Shot 3.

Tap the button under the reply that replays the turn without memory, or open the
web widget at /app and use "Show it without memory" on the same turn.

Two answers appear side by side:
- with memory: the answer that used what it knows about you
- without memory: the same model, same question, with no memories to draw on

Read them out loud, slowly.

> "Same model. Same question. Same minute. On the left it answered with what it
> remembers about me. On the right, memory switched off, and it had nothing to
> say. That is the whole product in one screen."

Cut.

---

## Shot 5 - it curates, not just accumulates (2:20 to 3:00)

Screen: Telegram.

Send: `actually I prefer a window seat`

Cheta notes it and should retire the earlier aisle-seat preference rather than
keeping both.

Type: `/memories`

The list appears. Point at the fact that the old preference is gone or marked as
replaced, and the new one is there.

> "Most memory systems just pile things up. This one decides what to keep. When I
> changed my mind, it retired what I said before instead of storing both versions
> and getting confused later."

Cut.

---

## Shot 6 - four surfaces, one memory (3:00 to 3:35)

Screen: laptop, the Chrome extension side panel open.

Type: `what do you know about me?`

The same facts come back, in the extension, with the recalled memories listed
under the reply.

> "The same memory, in a browser extension. Same for the command line, and the
> web page. One person, four clients, one memory."

If you have paired a second client, mention it here: that is how a new device
joins the same memory space.

Cut.

---

## Shot 7 - honest limits (3:35 to 3:55)

Screen: the extension or the onboarding panel, wherever "What it cannot do" is
visible.

> "It cannot log into my accounts. It cannot read private pages I am not looking
> at. It cannot see images or video. I would rather it say that than pretend."

Cut.

---

## Shot 8 - close (3:55 to 4:10)

Screen: the portal again, scrolled to the four surface cards.

> "Cheta. Memory you can see, correct, export and prove. Built on Walrus Memory."

Stop recording.

---

## Notes for the edit

- No music, or something quiet and instrumental. Do not let music compete with
  the voice.
- Do not speed anything up. If a reply is slow, cut the wait out rather than
  speeding the footage, because a sped-up UI looks fake.
- Keep every clip under fifteen seconds except Shot 4.
- If a tool call appears in a reply, leave it in. Watching it look something up
  is the difference between an assistant and an agent.
- If something fails on camera, and it is an honest failure that the bot explains
  clearly, consider keeping it and saying so. A system that states what it cannot
  do is more convincing than one that appears to never fail.

## Title and description for YouTube

Title: Cheta - a memory-first assistant that decides what to forget

Description: Cheta stores what you tell it in your own memory space on Walrus,
shared across Telegram, a command line, a web page and a browser extension. The
demo shows the same turn answered twice, once with memory and once without, so you
can see the difference memory makes rather than taking it on faith. Source:
https://github.com/Ksschkw/ranti
