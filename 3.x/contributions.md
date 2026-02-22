---
title: Contributions
description: Doppar contributions notes page
meta:
  - name: keywords
    content: contributions
---

## Contribution Guide
### Bug Reports
At Doppar, we welcome collaboration and encourage contributors to submit pull requests rather than just bug reports. This approach keeps the community engaged and ensures faster resolution of issues.

> Pull requests will only be reviewed if they are marked as "Ready for review" (not in "Draft" state) and all tests for new features pass. Draft or inactive PRs may be closed after a few days of inactivity.

## When Filing a Bug Report
If you need to file a bug report instead of a PR:
- Include a clear, concise title and a detailed description of the issue.
- Share relevant context, including:
  - Environment details (e.g., PHP version, Doppar version)
  - Steps to reproduce
  - Code snippets or test cases
- The goal is to make the bug easy to replicate so that you or others can work toward a fix efficiently.

> Bug reports help others with the same issue and may lead to a shared solution, but they don't guarantee immediate resolution. Your contribution helps lay the groundwork.

If you're feeling helpful, jump in and fix an open issue. Every bit of effort counts!

## Minor Issues (DocBlocks, IDE Warnings)
If you spot minor issues like incorrect DocBlocks, PHPStan violations, or IDE warnings, please do not open an issue. Instead, submit a pull request directly with the fix.

## Doppar Repositories
All Doppar projects are managed via GitHub. Feel free to explore, fork, and contribute:

- [Doppar Application](https://github.com/doppar/doppar)
- [Framework](https://github.com/doppar/framework)
- [Docs](https://github.com/doppar/docs)
- [AI](https://github.com/doppar/ai)
- [Queue](https://github.com/doppar/queue)
- [Airbend](https://github.com/doppar/airbend)
- [Flarion](https://github.com/doppar/flarion)
- [Notifier](https://github.com/doppar/notifier)
- [Orion](https://github.com/doppar/orion)
- [Guard](https://github.com/doppar/guard)
- [Axios](https://github.com/doppar/axios)
- [OAuthic](https://github.com/doppar/oauthic)
- [Bloom](https://github.com/doppar/bloom)
- [Insight](https://github.com/doppar/insight)
- [Twig Bridge](https://github.com/doppar/twig-bridge)
## Which Branch?
When contributing to Doppar, please make sure you are targeting the correct branch for your changes:

#### Bug Fixes

Send all bug fixes to the latest maintained version branch that supports them — currently, that's 3.x.

 > Do not send bug fixes to the master branch unless the bug affects a feature that exists only in the upcoming release.

#### Minor Features
Minor enhancements that are fully backward compatible with the current release may also be sent to the 3.x branch.

#### Major Features / Breaking Changes
New features that involve breaking changes or major refactors should always target the master branch, which tracks development for the upcoming release.

> ⚠️ Submissions to the wrong branch may be closed or requested to be retargeted, so please double-check before opening your pull request.

## Security Vulnerabilities
If you discover a security vulnerability within Doppar application, please send an email to Mahedi Hasan at mahedy150101@gmail.com. All security vulnerabilities will be promptly addressed.

## PHPDoc
Below is an example of a valid Doppar-style PHP documentation block. Doppar uses one space between the `@param / @return` tags and their respective types/names.
```php
/**
 * Handle an incoming request
 *
 * @param Request $request
 * @param \Closure(\Phaseolies\Http\Request) $next
 * @return Phaseolies\Http\Response
 */
public function __invoke(Request $request, Closure $next): Response
{
    if ($request->ip() === '127.0.0.1') {
        return $next($request);
    }

    return redirect('/home');
}
```

## Code of Conduct
The Doppar code of conduct is derived from the Ruby code of conduct. Any violations of the code of conduct may be reported to Mahedi Hasan (`mahedy150101@gmail.com`):

- Participants will be tolerant of opposing views.
- Participants must ensure that their language and actions are free of personal attacks and disparaging personal remarks.
- When interpreting the words and actions of others, participants should always assume good intentions.
- Behavior that can be reasonably considered harassment will not be tolerated.