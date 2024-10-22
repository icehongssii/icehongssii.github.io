- I do write tech related stuff at `Obsidian(It is like Notion).` All files are written in `Markdown`.
- All files are stored in [my Github repo](https://github.com/icehongssii/tech-blog-obsidian). I used Obsidian-git to commits and push every X minutes from my Local machine to GitHub
- Everytime Obsidian pushes to repo, It invokes AWS Lambda fuction that converts Markdown files to HTML and pushes to `icehongssii.github.io`repo.
- Bascially I used Three repos
- 1. The one that saves markdown files (AKA obsidian storage)
  2. The one that reads obsidian storage repo and converts markdown to html then pushes icehongssii.github.io 
  3. icehongssii.github.io
