Here's the thing to know before you paste anything: your agent doesn't need to touch the backend at all. This whole feature lives in the chat page component. So the prompt just needs to describe the behavior really precisely, because that's where agents usually mess up — they build a rail that looks right but jumps to the wrong message or flickers on scroll.

Here's what I'd tell it, copy-paste ready:

---

**Add a conversation minimap to the chat page (like ChatGPT's).**

Build a vertical rail of horizontal tick marks, fixed to the right edge of the chat scroll area, overlaying the content. One tick per **user** message in the conversation (ignore assistant messages).

**Structure:**
1. Give every user message container a ref (or a stable `id` like `msg-{index}` on the DOM node).
2. The rail is absolutely positioned inside the chat panel: `right: 0`, vertically centered, sitting above the content (`z-index` above messages, below modals). It should span between the top bar and the composer, never overlapping either.
3. Ticks are **evenly spaced** top to bottom (do NOT position them proportionally to scroll height — long answers would squish them together). Each tick: ~16px wide, 3px tall, rounded, muted gray at ~50% opacity.

**Scroll tracking (active state):**
4. Attach a scroll listener (passive) to the chat scroll container. On scroll, find the last user message whose `offsetTop <= scrollTop + ~120px`. That message is "active."
5. The active tick gets the accent color, grows to ~26px wide, and gets a subtle glow. Use a CSS transition (~180ms) so it animates as you scroll. Debounce or use rAF so it doesn't thrash.

**Hover flyout:**
6. When the cursor enters the rail's hover zone (make the hit area ~44px wide, wider than the ticks themselves, so it's easy to reach), expand a flyout panel to the left of the ticks: dark translucent background with backdrop blur, rounded corners, drop shadow, ~290px wide, max-height capped with internal scroll.
7. The flyout lists every user prompt in order: a small monospace index number (01, 02, …) plus the prompt text truncated to **2 lines** with ellipsis (`-webkit-line-clamp: 2`).
8. Hovering a row highlights it and also highlights its matching tick. The currently active message's row is visually marked (accent tint + brighter text).
9. Moving the mouse out of the rail zone closes the flyout (~180ms fade). No click needed to open or close.

**Click to jump:**
10. Clicking a tick OR a flyout row smooth-scrolls the chat container so that message's top sits ~24px below the top of the viewport (`scrollTo({ top: node.offsetTop - 24, behavior: "smooth" })`).
11. After jumping, briefly highlight the target user bubble (e.g., a soft accent ring for ~1.5s) so the eye lands on it.

**Edge cases:**
- Fewer than 2 user messages → don't render the rail at all.
- Very long conversations (30+ messages) → cap tick spacing so they don't overflow; the flyout scrolls internally.
- New message sent → rail updates, new tick becomes active, no layout jump.
- The rail must not block text selection or interfere with the native scrollbar; keep it inside the padding gutter (add ~40px right padding to the message column so bubbles never sit under it).
- Respect `prefers-reduced-motion`: use instant scroll instead of smooth.

Use our existing theme tokens for all colors — no hardcoded hex values.

---

That last line matters if you're also doing the theme system, since the rail should recolor with everything else. And one practical tip: have the agent build it as its own component, something like `<ConversationMinimap messages={userMessages} scrollRef={chatScrollRef} />`, so it stays isolated and you can drop it into the page without tangling it into your message rendering.

Want me to write the theme system prompt too, or are you starting with just this one?
