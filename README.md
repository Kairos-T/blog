# Blog
This ~~is~~ was my newest iteration of my personal blog. It is built using Hexo and the Redefine theme.
**Note**: Here is my latest blog's repo: [https://github.com/Kairos-T/blog-v2](https://github.com/Kairos-T/blog-v2)

## I want this too!!!
The theme's repo can be found [here](https://github.com/EvanNotFound/hexo-theme-redefine).

## I have zero clue how Hexo works...
Some useful commands:
- **Start the server**: `hexo server`
- **Clear Hexo cache**: `hexo clean`
- **Create a new post**: `hexo new post "post name"`

## How do I host this?
I hosted this blog on Vercel. Surprisingly, this was pretty much the easiest-to-deploy SSG I have encountered (so far). 

Here are the deployment settings I used:
- **Build Command**: `yarn; hexo generate`
- **Output Directory**: `public`
- **Install Command**: `yarn install; npm i hexo-generator-searchdb`
    - Note: `npm i hexo-generator-searchdb` is only necessary if you've enabled the search function.
- **Development Command**: `hexo server`
