# Blag site workflow

```
poetry shell
poetry install
blag build
```
then: copy files ``build/tags/*.html, build/index.html, build/new-page.html`` to corresponding dir in ``docs/*``

Then, git commit and push. 

I could write a script for this, but hey, [premature optimization is ...](https://m.xkcd.com/1691/)

 

