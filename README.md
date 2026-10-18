# VSLA Group Book (OYBA Build Up)

Cloud savings group (VSLA) record book. Plain HTML/JS, Firebase backend, syncs across devices, works offline.

## Database (Cloud Firestore)

```
groups/{groupId}            name, ownerId, settings{interestRate, loanWeeks, registrationFee,
                            retentionPerWeek, absentFine, lateFine, noiseFine,
                            loanFormFee, openingBalance}
  members/m{ID}             memberNo, name, gender, village, contact, joinDate, status
  meetings/{YYYY-MM-DD}     date  (one per meeting day, Sunday or Saturday)
  expenses/{id}             date (meeting date), item, amount   (printing, pens, rulers...)
  entries/{date}_m{ID}      one row per member per meeting, same columns as the Excel
                            "Weekly Savings" sheet: attendance, registration, retention,
                            savings, socialFund, loanTaken, loanRate, loanRepayment, loanFormFee,
                            absentFine, lateFine, noiseFine, otherFines, totalFines, totalPaid
```

The Excel "Loan Register" is calculated from the entries (Loan taken / Loan repayment), so it can never disagree with the weekly sheet. Week numbers follow the order of meeting dates.

Total Fines = absent + late + noise + other. Total Paid = registration + retention + savings + social fund + loan repayment + loan form fee + fines (loan taken is cash going out, so it is not counted).

## Updating your existing project

This is the same project. Replace `index.html` in your existing GitHub repo with this one (pencil icon > paste > Commit). Your Firebase project, security rules, login and GitHub Pages link stay the same. Members added with the first version are numbered automatically the first time you open the app.

## Meeting cash summary

Each meeting shows money in (registration, retention, savings, social fund, loan form fees, fines, loan repayments), money out (loans given and expenses) and the money left in the box. Set the box balance you started with under Tools > Group settings > Opening balance. The Summary tab has a cash book with every meeting.

## Setup

1. Go to https://console.firebase.google.com and create a project.
2. **Build > Authentication > Sign-in method:** enable **Email/Password** (the app turns phone + 6-digit PIN into a hidden email).
3. **Build > Firestore Database:** create a database, then open **Rules** and paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    match /groups/{gid} {
      allow create: if request.auth != null && request.resource.data.ownerId == request.auth.uid;
      allow read, update, delete: if request.auth != null && resource.data.ownerId == request.auth.uid;
      match /{sub=**} {
        allow read, write: if request.auth != null
          && get(/databases/$(db)/documents/groups/$(gid)).data.ownerId == request.auth.uid;
      }
    }
  }
}
```

4. **Project settings > Your apps > Web (</>):** register an app and copy the config into `firebaseConfig` in `index.html`.
5. **Authentication > Settings > Authorized domains:** add `YOUR-USERNAME.github.io`.

## Deploy on GitHub Pages

1. Create a GitHub repository and upload `index.html` and `README.md`.
2. **Settings > Pages > Deploy from a branch > main / (root) > Save.**
3. Open `https://YOUR-USERNAME.github.io/REPO-NAME/` on any phone and choose "Add to Home Screen".

Sign in with the same phone number and PIN on any device to see the same group.

## Import your Excel data

In the app open **Tools > Import from Excel (CSV)**. Import `members.csv` first, then `history.csv`.
`history.csv` holds the savings and social fund rows that have real amounts in your workbook. Attendance is imported for weeks 1 to 8 only; later attendance cells looked like placeholders.
Loans were not imported because the Loan Register has no loan dates. Record them in the Meeting sheet.

## Next steps

- Let the chair and members sign in with their own accounts (add a `memberUids` list to the group and update the rules).
- SMS balance alerts, Mobile Money, and local language labels.

## Rules built in (v2)

- Retention is paid once, on the first meeting of a cycle (Setup > Cycle start date, Retention).
- Loans: 10% interest for every month started, up to 3 months (30%). Penalty of 5,000 if not cleared after 3 months. All amounts are in Setup.
- Weekly savings must be an even amount from 2,000 to 20,000. Social fund 1,000 to 10,000.
- Pay-ahead savings: enter total handed in and weeks covered on a member's details. Future weeks show in yellow and apply automatically.
- Cash book: social fund and loan fund balances, short-box warning, cash count with difference, social fund payouts.
- Share-out uses stars (savings / star value). Group logo is set in Setup.

## Saving week updates (v3)

- **Saving week > Week sheet**: every member listed with ID and name; type savings, social fund and loan refund directly in the row (Enter = next member). Saves automatically and keeps all other details on the record. Roll call (tap mode) is still there.
- Members have a **Member type** (Old / New). New members have a probation (Setup > Probation weeks, default 3) with no savings, counted from their join date.
- Registration fee is fixed: old and new rates in Setup. Retention is entered per member on the first week of a cycle.
- Fines (Setup): missed saving day, missed group meeting, late coming, noise making. Absent/Late attendance adds its fine automatically; others via the Fines button.
- Loan default penalty is a flat one-time amount (Setup, default 5,000).
- Reports > This week summarises the current week's transactions.
