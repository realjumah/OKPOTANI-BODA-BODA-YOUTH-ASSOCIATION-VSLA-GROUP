# Subscription update: 14-day trial, then monthly by mobile money

You do NOT need a new Firebase project. Use the same one. Your groups, members and records are not touched.
You change two things in Firebase: the **Rules**, and one **admins** document you add by hand.

## How it works for a client
1. A new client signs up and creates their group. A **14-day free trial** starts the first time they sign in.
2. From 5 days before the end, a strip at the top reminds them. They can tap **Subscribe** at any time (Setup > Subscribe).
3. When the trial ends, the app becomes **view and download only**. They can still see every record, run reports and download
   a backup or spreadsheets, but cannot add or change anything. Nothing is ever deleted. This is enforced by Firebase itself,
   not only by the screen.
4. To subscribe, the client sends the monthly fee by mobile money to your number, then enters the **transaction ID**, the number
   they paid from and their name, and taps **Send payment details**.
5. You see it under **Admin screen > Payments**. Check your own mobile money message (amount, sender number, transaction ID),
   then tap **Approve**. Premium turns on for the months paid (30 days per month; paying again adds on top of what is left).
6. The client gets a message on their Home screen: "Payment confirmed". If something does not match, tap **Reject** and they are told.

## Reminders
- **Automatic:** whenever you open the app as admin, every client whose trial or subscription has 5 days left, 1 day left, or has just ended gets a message on their Home screen (once for each of those stages, so nobody is spammed). You see "Reminders sent to N clients".
- **Send now:** Admin screen > Clients > **Send reminders now** sends to every client who is due, even if they were reminded before.
- **WhatsApp:** tap a client > **WhatsApp reminder**. It opens WhatsApp with the message and the app link ready to send.
- **Switch off:** Admin screen > Pay details > Automatic reminders > Off.
- Clients also see a coloured strip at the top from 5 days before the end.
- The firestore.rules file did not change for this update. Only `index.html` needs replacing.

## Set it up (do the steps in this order)
**Step 0.** Setup > Download backup file. Keep it on your phone (never on GitHub).

**Step 1. GitHub:** replace `index.html` with the new one. Nothing else changed. Commit, wait 2 minutes, close and reopen the app twice.

**Step 2. Rules:** Firebase console > Firestore Database > Rules > delete everything > paste all of `firestore.rules` > Publish.
Do this AFTER step 1, because the new rules expect the new app.

**Step 3. Make yourself admin (once, by hand, so nobody can do it from the app):**
1. Firebase console > Authentication > Users. Find your own account (it looks like `07xxxxxxxx@savingsbook.app`) and copy its **User UID**.
2. Firestore Database > Data > **Start collection** > Collection ID: `admins`.
3. Document ID: paste your UID. Add one field: `name`, type `boolean`, value `true`. Save.

**Step 4.** Open the app signed in with that account. Setup > **Admin screen > Pay details**: enter your mobile money number,
the fully registered name on it and the price per month. Save. Your own group is free; admin accounts are never billed.

**Step 5. Test with a second account before telling clients:**
- Sign up with another phone number, create a group, and check Setup shows "Free trial: 14 days left".
- Tap Subscribe and send test payment details.
- On your admin account: Admin screen > Payments > tap it > Approve. The second account gets "Payment confirmed".
- To test the lock quickly: Admin screen > Clients > tap the test client > Extend trial or Add paid days (to give days), or in
  Firebase edit that client's `billing` document `trialDays` to 0 and `paidUntil` to empty and see the view-only screen.

## What you can do as admin
- **Payments:** approve or reject waiting requests.
- **Clients:** see everyone's status (trial days left, paid until, expired). Tap one to add paid days, extend the trial or make them free.
- **Message:** send a message to all clients or one client; it appears on their Home screen.
- **Pay details:** change your number, name or price.
- You can see each client's group name and phone number, but never their members or savings records.

## Known limits
- A new phone number gets a new trial. Ask clients to use one number.
- If a client edits records offline after the trial has ended, those edits will not sync.
- Approvals are manual: you see "N payment requests waiting" at the top when you open the app.
- Reminders are sent when YOU open the app, and clients see them when THEY open it. Messages that reach a phone with the app closed (push notifications or SMS) need Firebase Cloud Functions, which require the paid Blaze plan, so they are not included.
- Everyone who can sign in to the same account shares that account's subscription.
- Firebase free limits apply (about 50,000 reads and 20,000 writes a day). The lock adds about two small reads to each saved change, which is fine for a savings group.
