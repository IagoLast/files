# Filenames

While variables are named using camelCase, files should be named using a [snake case](https://en.wikipedia.org/wiki/Snake_case) convention. This is because sometimes git can play some tricks on you on case-insensitive filesystems. das-case totally avoids this problem:

Use an ending to indicate the content type of the file. So is easier to understand the purpose.

<component-name>.<type>.<ts/tsx>

Examples:

- `user-profile.component.tsx`
- `user-profile.controller.ts`
- `user-profile.test.tsx`
- `user-profile.service.ts`
- `user-profile.repository.ts`
- `user-profile.queries.ts`