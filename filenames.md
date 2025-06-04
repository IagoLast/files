# Filenames

While variables are named using camelCase, files should be named using a [snake case](https://en.wikipedia.org/wiki/Snake_case) convention. This is because sometimes git can play some tricks on you on case-insensitive filesystems. snake_case totally avoids this problem:

- [https://stackoverflow.com/questions/6899582/i-change-the-capitalization-of-a-directory-and-git-doesnt-seem-to-pick-up-on-it/6899682](https://stackoverflow.com/questions/6899582/i-change-the-capitalization-of-a-directory-and-git-doesnt-seem-to-pick-up-on-it/6899682)

[https://stackoverflow.com/questions/10523849/changing-capitalization-of-filenames-in-git](https://stackoverflow.com/questions/10523849/changing-capitalization-of-filenames-in-git)

[https://www.reddit.com/r/git/comments/dj5b8n/lpt_check_your_uppercaselowercase_file_names/](https://www.reddit.com/r/git/comments/dj5b8n/lpt_check_your_uppercaselowercase_file_names/)

**Use an ending to indicate the content type of the file:**

- `user-profile.component.tsx`
- `user-profile.controller.ts`
- `user-profile.test.tsx`
- `user-profile.service.ts`
- `user-profile.repository.ts`
- `user-profile.queries.ts`