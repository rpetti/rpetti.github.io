---
title: "Perforce Security Levels are a Lie"
layout: "post"
---

Specifically security level 1. Here's the problem and how to work around it.

In the fine manual, we have this fine text for security level 1:
> Ensures that all users have passwords.

Now, a reasonable, security-conscious person would assume that this means any account without a password will be unusable until a password is set. And this is true! But what this doesn't tell you is that even on the highest security level, a _passwordless account can have its password reset by literally anyone without requiring authentication_.

This means that if an account is created with internal auth and doesn't have the password set, then it can be hijacked by anyone, even though the security level says otherwise. Combine this with the fact that, by default, anyone can dump the list of users from a Perforce server, and you end up in a situation where a hacker can have a very easy time infiltrating your source control system.

Perforce/Helix is very open by default, and you need to go through some pretty extreme lengths to make it secure, especially when their fine documentation isn't telling the whole truth.

Here's a checklist of things that can help lock this down:

- Disable anonymous user listing to help prevent passwordless users from being identified: `p4 configure set run.users.authorize=1`
  - Not really a solution to the problem, but can help by obscuring the list of users in P4.
- If you are using LDAP, change the default AuthMethod to LDAP: `p4 configure set auth.default.method=ldap`
  - Rather than changing the default authmethod for accounts that don't have it set, this pre-populates user forms with AuthMethod: ldap.
  - Doesn't stop someone from changing their own AuthMethod.
- If again you are using LDAP, add a 'form-save user' trigger that will check the formfile for `AuthMethod: ldap`
  - Completely prevents users from changing their AuthMethod.
  - Completely prevents even admins from creating non-LDAP accounts, either accidentally or intentionally.
  - The trigger can also be modified to allow exceptions if you are running a mix of account types.
- Block all access to the `p4 passwd` command via triggers
  - This was the only 'solution' offered by Perforce Support.
  - Not a solution if you are running mixed account types, since you'd want folks to be able to change their own password.
- Change your user creation process to always have a random password set on creation
  - Unreliable since it depends on humans to do the right thing.
- Add a `pre-user-passwd` trigger that checks whether there is a password before allowing it to be set. You can check if a password is set using the `p4 -ztag -F %Password% users $user` command.
  - This also blocks admins from setting passwords unless your triggers script logic allows it in some way.

Overall, I'm incredibly disappointed by Perforce's behavior when it comes to it's security levels. A very simply mistake (creating an account with the wrong authmethod) can result in compromised accounts extremely easily. The behavior should match the spirit of the documentation, which is that passwordless accounts should not be allowed in security level 1 or above.
