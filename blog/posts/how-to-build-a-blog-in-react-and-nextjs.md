---
title: How to build a blog in React and Next.js
publishedOn: '2022-11-20'
---

While there are many out-of-the-box solutions for blogging, I believe that coding our way to our very own build has many advantages:

1. Apply any changes to your blog that your heart desires
2. Make the most modern or silliest UI
3. Control the complexity of your posts management system
4. Use your blog as a portfolio piece
5. Use building your blog as content for your blog 😅

Before we start, if you'd like an EZ Pz copy pasta solution, this very blog's code can be found on my [Github](https://github.com/waaverecords/michael.nadeau.dev). Now, if you didn't click the link, let's get started!

## Project initialization

First, a Next.js project must be initialized. Open your favorite terminal and type the following:
```
npx create-next-app@latest --typescript
```

Next, navigate to your newly created project's folder and install a few helpful packages by typing:
```
npm install gray-matter react-code-blocks react-markdown remark-gfm
```

Let's now remove most of the boilerplate code the create-next-app script wrote for us. Cleaning the project is the first thing I always do; I do not want to worry about carrying unused code within my commits later on. After the cleanup, your project's folder structure should look something like this:
```
node_modules/
pages/
    _app.tsx
    index.tsx
public/
.gitignore
next-env.d.ts
next.config.js
package-lock.json
package.json
tsconfig.json
```

_app.tsx should be almost empty:
```tsx
// pages/_app.tsx

import type { AppProps } from 'next/app';

export default function App({ Component, pageProps }: AppProps) {
    return <Component {...pageProps} />;
}
```

and so does index.tsx:
```tsx
// pages/index.ts

export default function Home() {
    const b = true;
    return null;
}
```

With the purge being done, we will start building the core blocks of our application.

## Markdown

You could write your content using plain HTML. And, while it would be a viable solution, there is a much simpler and faster way to do so. Markdown is a lightweight markup language that uses a plain text formatting syntax: It's easy to read and write; It's widely adopted by writers and publishers, And it doesn't break.

We first need a folder to contain all of our juicy content. I'll call mine "posts" and place it in the root folder of my project.

Then, append to that folder a file with the "md" extension. The file's name should be descriptive of its content as it will serve as the slug/path to access that post. For example:
| File name |Slug  | Url |
| - | - | - |
| post1.md | post1 | https​://myblog.com/articles/post1 |
| how-to-build-a-blog.md | how-to-build-a-blog | https​://myblog.com/articles/how-to-build-a-blog |

The posts folder and first Markdown file created, your project's structure should look like this:
```
...
posts/
    post1.md
...
```

Both the content of your post and its metadata will be written in the same file. That way, the data is centralized and easily accessed. The metadata should occupy the first lines followed by the content. Notice how the metadata is surrounded by dashes "---"; The "gray-matter" package installed earlier will look for these and parse the data within:
```js
// posts/post1.md

---
title: How to build a blog in React and Next.js
publishedOn: '2022-11-20'
---

Pariatur cupidatat ipsum non exercitation.
...
```

How to parse the Markdown will be shown in the next section.

## Parsing Markdown

Create a new file at the root of your project and name it "blog.ts". Two functions within that file will be written to help us retrieve our posts' data. But first, to manipulate posts easily, make use of Typescript by creating a new type named "Post":
```ts
// blog.ts

export type Post = {
    slug: string;
    title: string;
    publishedOn: string;
    markdown: string;
}
```

The first function takes a post's slug as its parameter and returns the post's data. It will be used inside our "/[slug]" page. Also, notice the use of the "gray-matter" package mentionned earlier:
```ts
// blog.ts
// ...

import fs from 'fs';
import path from 'path';
import matter from 'gray-matter';

const root = process.cwd();

export function getPostBySlug(slug: string) {
    const content = fs.readFileSync(
        path.join(root, 'posts', `${slug}.md`),
        'utf8'
    );

    const { data: metadata, content: markdown } = matter(content);

    return {
        ...metadata,
        slug: slug,
        markdown
    } as Post;
}
```

This second function, which makes use of the first one, returns all posts present within the posts folder. It will be used inside our "index" page:
```ts
// blog.ts
// ...

export function getAllPosts() {
    const folderPath = path.join(root, 'posts');

    if (!fs.existsSync(folderPath))
        return new Array();

    const files = fs.readdirSync(folderPath);

    const posts = files.reduce((posts, fileName) => {
        const post = getPostBySlug(fileName.replace('.md', ''));
        
        return [
            post,
            ...posts
        ];
    }, new Array<Post>());

    return posts;
}
```