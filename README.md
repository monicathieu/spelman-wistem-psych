## Monica's notes

May 8 2023: Set the `blogdown.hugo.dir` and `blogdown.hugo.version` R options in the .Rprofile to get blogdown to see the npm-handled local version of Hugo. This seemed to change Hugo's initial install path from `node_modules/.bin/hugo/hugo` to `node_modules/.bin/VERSION_NUM/hugo`. Symlinking the executable to the original location didn't seem to work, so I changed the script calls in `package.json` to point to the new Hugo path. Seems hacky because now the version number is hard coded in, but it works?
