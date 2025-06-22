Run this command whenever trying out local changes:


hugo server -D --baseURL=https://krishnayadav.xyz/ --appendPort=false

Note: This is done because otherwise hugo doesn't seem t o pick the baseURL from hugo.toml.


Fix this error: Failed to find a valid digest in the 'integrity' attribute for resource ... ?

With this in config.yml [adapt it for toml as required] : 

params:
  assets:
    disableFingerprinting: true

Always make sure to run the following command if wish to clone:

```bash
git clone --recurse-submodules -j8 git@github.com:ObsidianMaximus/new-site.git
```

And to update the submodule, follow: [Link](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation#installingupdating-papermod)
