# NTHU badminton booking bot

Reserves **one Main Campus Alumni Hall badminton court for eight students**, using the normal booking flow in Microsoft Edge. It chooses any court that is available for the entire requested playing time. You log in and solve the CAPTCHA yourself.

This is an unofficial local assistant for **Windows**, not a service run by NTHU. Each person runs it on their own computer with their own account.

## Download and install

1. Install [Node.js LTS](https://nodejs.org/en/download) (version 22 or newer) and [Microsoft Edge](https://www.microsoft.com/edge/download).
2. On this repository page, click **Code → Download ZIP**, then extract the ZIP into a regular folder. Or clone it:

   ```powershell
   git clone https://github.com/JasonNababan/nthu-badminton-bot.git
   ```

3. Open the extracted folder and double-click **Start Bot.cmd**. On the first run it downloads the required packages; then it opens the Edge dashboard. Keep the launcher window open while the bot is running.

No GitHub account or Git installation is required when using Download ZIP. Your browser profile, passwords, and booking history are not part of the download.

## Start

1. Double-click **Start Bot.cmd** after following the installation steps above.
2. In the Edge dashboard, click **Open booking / log in**. Log in in the booking tab and dismiss any login notice.
3. Return to the dashboard and click **Check connection**. This selects Main Campus / Alumni Hall and shows the site's time-slot labels.
4. Choose the playing date, start, and end. Use the exact slot boundaries the site offers. The bot supports one or two consecutive slots on the **same court**.
5. To try filling without automatic submission, choose **No — fill only; I can submit manually** under **Should the bot submit?**, then press **Fill form only**. The bot selects the court and fills eight students. After it finishes, you can click the site's Submit button yourself.
6. For automatic reservation, choose **Yes — Live: book automatically**, set the desired booking start date/time, and press **Start automatic booking**. This arms automatic submission; no further confirmation is required.

All booking start dates and times mean **Taiwan time (UTC+8)**, regardless of Windows' timezone. The default scheduled time is 00:01, and you can change it every run. The playing date/time and the booking start date/time are separate settings.

The booking grid currently shows **today through four days ahead**. For example, Thursday becomes visible on Sunday. Configure the actual playing date and when to start; the bot does not invent a release time or bypass the site's booking window.

## Timing and repetition

The bot loads the venue form before the target time and checks login again roughly 15 seconds beforehand when enough preparation time remains. At the target time it refreshes availability, selects a matching court, fills the form, verifies the outgoing request, and submits once. It uses browser actions without artificial click delays. Availability polling, when enabled, waits at least one second between completed checks.

The selected time is **when booking starts**, not a guarantee that the server receives the final reservation at precisely that instant. Network latency, page loading, other users and the server determine the outcome.

Choose once, daily or weekly, with 1–12 runs. Repeats advance both dates while preserving times. A failed or uncertain result stops the series. A rehearsal always stops after its first filled form.

Keep the app and Edge open, the computer awake, and Windows' clock synchronized. The bot does not wake a sleeping computer or keep an expired login alive. If the process restarts, saved settings are restored but **runs are not automatically rearmed**. Recheck dates before starting again.

## Results and duplicate prevention

- **Rehearsal complete:** the booking form is filled; no reservation was sent.
- **Submitted:** the server responded successfully and the site navigated to its application list. This means the rental request was submitted; check the site's status for final approval.
- **Needs review:** a request may have reached the server, but the outcome is uncertain. Check the site's application list before doing anything else.
- **Error:** read the dashboard message and booking tab. The bot stops when the venue, student category, schedule or form cannot be verified.

Before sending, the bot checks the venue ID, date, time-slot IDs, court number and exactly eight students. It writes a local attempt record before allowing one submission. Overlapping time ranges are blocked after an attempt, even if a timeout occurs. It never automatically retries a reservation submission.

If an attempt definitely failed and the site's application list confirms no reservation exists, close the bot and remove only the corresponding entry from `.runtime/attempts.json` before trying again. Keep successful or uncertain entries. The log is `.runtime/bot.log`.

**Stop run** prevents subsequent bot actions. A request already sent may still complete. **Close bot** closes the bot's Edge window and stops the process. Manual submission works normally while the bot is idle, including after fill-only mode finishes. While a bot run is active, stop it and wait for it to finish before submitting manually. Local attempt history covers automated bookings; check the site's application list for bookings made manually or in another browser.

## Browser and local files

The bot uses a separate persistent Edge profile in `.runtime/edge-profile`. Logins in your ordinary Edge window are not automatically shared with it. Subsequent launches reuse the bot profile, subject to the website's session expiry. No password or CAPTCHA answer is collected by this app.

`.runtime/` contains the browser login session, saved settings, activity log and attempt history. Keep it private. The dashboard only listens on this computer's loopback address. Its local API requires the per-launch dashboard token.

To share with someone else, send them this repository's link. If copying the folder yourself, exclude `.runtime` and `node_modules`. Never share your `.runtime/edge-profile` folder.

## Troubleshooting

- **The bot fills the form but does not submit:** choose **Yes — Live: book automatically** under **Should the bot submit?**. Fill-only mode intentionally stops before submission; you can submit manually when the run has finished.
- **Node.js is not found:** install Node.js LTS, then reopen Start Bot.cmd.
- **Login or connection check fails:** complete the CAPTCHA in the bot's Edge window, dismiss any login notice, and click Check connection again. Logging in through your usual Edge window does not log in the separate bot profile.
- **No court is available:** check the exact playing date and the time-slot labels shown after Check connection. Both selected slots must be free on the same court.
- **A run is already active:** stop it before changing the setup or submitting manually.

## Verification and scope

Run `npm ci --ignore-scripts` and then `npm test` from this folder. The GitHub Actions workflow runs the same tests on Windows. Automated tests use local fixtures in headless Edge and never make real reservations: court intersection across slots, eight-student filling, rehearsal with zero submissions, one-request submission, server failure without retry, cancellation, Taiwan dates, booking-window validation, payload verification, manual submission after rehearsal, and duplicate prevention.

The adapter was built against the site's public JavaScript and UI structure on 2026-09-27. The authenticated fill-only flow has also been verified on one student account. Run a fill-only check on your own account before scheduling an unattended live booking. Future site changes may require updating `booking.mjs`.

Reference: [NTHU booking site](https://peo178.et.nthu.edu.tw/) · [Playwright browser support](https://playwright.dev/docs/api/class-browsertype)
