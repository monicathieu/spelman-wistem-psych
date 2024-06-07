## Monica's notes

June 6-7 2024: Did a massive push to update doks to 1.6.1 (and stop using the child theme). Major doks feature changes include that the callout shortcodes are now named differently. I did have to... reinstall Hugo using homebrew? It doesn't appear to have broken blogdown's Hugo installs for my other website repositories, though, thankfully.

May 9 2023: When blogdown calls Hugo, it seems to force-move the Hugo executable into a subfolder with the version number. Avoid this by calling `blogdown::build_site(run_hugo = FALSE)`. I am changing the `package.json` back to the way it was because no other Hugo call seems to do this with the version number folders.

May 8 2023: Set the `blogdown.hugo.dir` and `blogdown.hugo.version` R options in the .Rprofile to get blogdown to see the npm-handled local version of Hugo. This seemed to change Hugo's initial install path from `node_modules/.bin/hugo/hugo` to `node_modules/.bin/VERSION_NUM/hugo`. Symlinking the executable to the original location didn't seem to work, so I changed the script calls in `package.json` to point to the new Hugo path. Seems hacky because now the version number is hard coded in, but it works?
