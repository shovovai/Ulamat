# Scholar review workflow

How drafted content becomes reviewed and goes live.

## One-time setup (admin)

1. Run `supabase/migrations/0002_scholar_reviews.sql`. It adds the `scholar` role and the `content_reviews` table.
2. The scholar signs in once, so they have a profile.
3. With the service role (SQL editor), set their role and name:
   `update profiles set role = 'scholar', display_name = 'Shaykh …' where id = '<user id>';`
4. Add the same name to `content/scholars.ts` (name, qualifications, role) and deploy. Until the name is on the board, the Approve button is disabled.

## Reviewing (scholar)

- Open **/admin/review**. Filter by items or walkthrough steps, status and section.
- Each card shows the Arabic, pronunciation, meaning, instructions and madhab notes in every language, plus the source.
- **Approve**, or **Request changes** with a comment. Every decision is kept; nothing is edited or deleted.
- The content fingerprint (hash) is taken on the server at the moment of the decision.

## Going live (admin)

1. On /admin/review, **Download reviews.ts** and replace `content/reviews.ts` with it.
2. Commit and deploy. `npm test` checks that every reviewer is on the board.
3. An approval only applies while the text is unchanged. If anyone edits an approved item or step, its hash changes, the approval stops applying, and it shows as waiting again (marked "The text changed after it was approved").

Without Supabase, `/admin/review` is a read-only preview in development builds.
