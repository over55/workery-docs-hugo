# workery-hugo
The source code of the [workery.com](https://workery.com) website written using [Hugo](https://gohugo.io/), a static content generator written in Go.

## Installation

To begin, clone this repository

```
git clone https://github.com/over55/workery-hugo.git
```

Once cloned, you'll want to install any dependencies you have. Here is for MacOS:

```
brew install hugo
```

Now to start the server locally, run the following.

```
hugo serve -D
```

In your browser, enter the path [http://localhost:1313](http://localhost:1313).

## Usage

### New Post

Here are a few examples:

```
hugo new posts/my-first-post.md
```

### Deployment

```
hugo --minify
cd public
git add --all;
git commit -m 'Latest commit...';
git push origin master;
```
