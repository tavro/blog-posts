# cloudflare-sso-issues.md

here is the story about how cloudflare's SSO integration screwed me over, and how you can avoid or resolve the same issue.

## the setup

1. i bought a new .se domain.

*quick aside: since internetstiftelsen owns sweden's national top-level domains, you can only use [valid registrars](https://internetstiftelsen.se/en/domains/how-to-register-a-domain-name/registrars) to obtain one. i have used Loopia a lot before, so that is what i use for .se domains. however, their DNS configuration UX is garbage, and i prefer having all my domains managed in one place. so my main platform nowadays is cloudflare. you can say what you want about them, and someday i will, because i have strong opinions, but i will save the political rambling for another time.*

2. i pointed the nameservers to cloudflare.

3. i reconfigured my mail server to use this new domain.

this is where i got fucked. since i originally signed up using SSO with my old domain's email address, that is what my cloudflare account was tied to. when i logged in with SSO using the same mail account but with my new domain (different email address, same underlying account), cloudflare created an entirely new account. i was now locked out of my original account, the one with my domain.

cloudflare should really fix their SSO integration to handle this case. email addresses change. domains change. the underlying identity does not. but also, maybe you should avoid using SSO altogether.

*another side note: SSO can be a single point of failure, if your identity provider goes down or locks you out, you lose access to everything connected to it. and most SSO systems tie your identity to an email address. change your email, and you might become a different person in the system's eyes (as i learned the hard way). many services do not let you add a password alongside SSO, leaving you with zero recovery options if SSO breaks. thankfully cloudflare let me add a password, though. when using SSO you are trusting a third party. your access depends on your identity provider's uptime and policy decisions, which are all outside your control.*

## the "support" experience

i contacted support, and they could not do anything for "privacy reasons" they said. i will admit i wasn't the nicest while talking to them, so if the support folks happen to read this i am sorry.

but, when your only options to regain access to your own domain are:

- **option a:** transfer it to a new account and lose all DNS records
- **option b:** manually connect a password to your old email address (i.e., stop using SSO)

...i am not going to be happy.

this defeats the entire purpose of SSO. now i have one more password to remember. this also broke my 2FA, which i had to reconfigure from scratch. i will probably never use that account again, but i still do not want it to be hacked.

## lessons learned

- do not rely solely on SSO for critical infrastructure accounts. set up a password as a backup before you need it.
- before changing your email domain, audit which services are tied to that address. especially ones with SSO.
- document your DNS records externally. if you ever get locked out, you will want a backup.
- cloudflare's SSO does not handle email domain changes gracefully. be aware.

---

thanks for listening to my rant. i am a nice person in real life, i promise. ;-P
