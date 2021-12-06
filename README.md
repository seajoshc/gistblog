# Gistblog - Blog your little &lt;3 out using GitHub Gists.

Do you need a simple platform for sharing your thoughts and ideas with the internet? Have you ever wanted an easy way to turn some plaintext into a blog post? Don't care about things like SEO, monetization, or having a fancy looking website? Is "indefinitely free" your desired monthly cost of hosting your own blog? Gistblog is the blogging engine for you!

Gistblog simply takes your markdown files and turns them into blog posts powered by [GitHub Actions](https://github.com/features/actions) and [GitHub Gists](https://gist.github.com/). Gistblog requires an opinionated setup to function so please read the below carefully.

# Setup
1. You may clone/fork this repo, create a new repo, or use an existing repo. If you didn't clone/fork this repo, create two directories in the root of your repo: a `blog/` directory and a GitHub Actions workflow directory `.github/workflows/` if it doesn't already exist.
1. Create a new [GitHub Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) with the `gist` scope.
1. Create a [secret for the repo you're using](https://docs.github.com/en/actions/security-guides/encrypted-secrets) named GISTS_TOKEN and the value should be from the token you created above.
1. If you didn't clone/fork this repo, create `.github/workflows/gistblog.yaml` with the following GitHub Actions workflow. Be sure to change your branch name below if it isn't `main`:
```yaml
name: Gistblog
on:
  push:
    branches:
      - main
jobs:
  gistblog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: dorny/paths-filter@v2
        id: blog
        with:
          list-files: shell
          filters: |
            create_blog:
            - added: 'blog/*.md'
            update_blog:
            - modified: 'blog/*.md'
      - if: steps.blog.outputs.create_blog == 'true'
        uses: seajoshc/gistblog-action@v1
        with:
          gists-token: ${{ secrets.GISTS_TOKEN }}
          operation: create
          blog-files: "${{ steps.blog.outputs.create_blog_files }}"
      - if: steps.blog.outputs.update_blog == 'true'
        uses: seajoshc/gistblog-action@v1
        with:
          gists-token: ${{ secrets.GISTS_TOKEN }}
          operation: update
          blog-files: "${{ steps.blog.outputs.update_blog_files }}"
```

# How to blog with Gistblog
Gistblog simply looks for new, or updated, markdown (`.md`) files in the `blog/` directory of your repo each time you push to main. If there are new files, Gistblog will automatically create new Gists for your blog posts. And each time you update a file the Gist gets updated too. Each Gist/blog post is backed by a permanent link that you can share out. It's that easy!

Ideally, you set a description for each blog post by declaring
```html
<!--
post_description: Example Post Title: A short description goes here. 
-->
```
at the top of each markdown file, like in my [example.md](./blog/example.md). Link previews look a bit nicer when you use this description format.

Gistblog also maintains a [Table of Contents(ToC)](https://gist.github.com/seajoshc/d898750deca042b9b241ad8b79bcac96) for all your blog posts. The ToC gets updated each time Gistblog runs so it's always up to date. Check your [Gists](https://gist.github.com) to find your ToC. Try putting the markdown table from your ToC into your [GitHub Profile README](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-github-profile/customizing-your-profile/managing-your-profile-readme) to [share your blog with the innerwebz](https://github.com/seajoshc). This is a manual process for now but I'll be looking to solve that in the future!

If you've got assets, like images, create `blog/assets/` and keep your files there. To include them in your post, you'll need to use the URL of where they will end up once pushed into your repo as the image source. The URL is predictable based on the file name and your GitHub username: `https://raw.githubusercontent.com/<github_username>/<repo_name>/main/blog/assets/<filename>.<extension>`. See my [example.md](./blog/example.md).

Changing the filenames on any Gists created by Gistblog will result in duplicate blog posts next time you update. It's best not to touch anything on the Gists really.
