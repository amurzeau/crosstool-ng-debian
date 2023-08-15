# Build procedure

Here are the commands to manage the package

## For unstable

```sh
# Switch to master branch
git checkout master

# Import upstream version and update Debian branch with new upstream release
gbp import-orig --uscan

# Update debian/patches
gbp pq rebase

# Go back to master branch
gbp pq switch

# Check licenses changes and update debian/copyright
find . \( -path './.git' -o -path './debian' \) -prune -o -type f -print0 | xargs -0  grep -i '\bCopyright\b'

# Check for new dependencies
git diff upstream^..upstream docs-requirements.txt dev-requirements.txt pyproject.toml docs/install.rst

# Check for new supported players
git diff upstream^..upstream docs/players.rst

# Check for important changes that may need other package changes (copyright, new patch, ...):
#  `https://github.com/streamlink/streamlink/compare/<old_tag>...<new_tag>`

# Update debian/changelog
gbp dch -Ra --commit

# Do test build with sbuild
gbp buildpackage "--git-builder=sbuild -v -As -d unstable --no-clean-source --run-lintian --lintian-opts=\"-EviIL +pedantic\" --build-failed-commands '%SBUILD_SHELL'"

# Check for build warnings or errors
grep -Pi 'error|warn' ../build-area/streamlink_$(dpkg-parsechangelog --show-field Version)_amd64.build

# Diff with previous package version
diffoscope ../build-area/crosstool-ng_$(dpkg-parsechangelog --show-field Version -c1 -o1)_amd64.deb ../build-area/crosstool-ng_$(dpkg-parsechangelog --show-field Version)_amd64.deb --text-color=always | less -r -s

# Tag commit
gbp tag --sign-tags

# Push to git repository
git push origin master upstream pristine-tar --tags

# Push to mentors FTP
dput mentors ../build-area/streamlink_$(dpkg-parsechangelog --show-field Version)_amd64.changes
```
