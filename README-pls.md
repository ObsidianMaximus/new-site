Run this command whenever trying out local changes:


hugo server -D --baseURL=https://krishnayadav.xyz/ --appendPort=false

Note: This is done because otherwise hugo doesn't seem t o pick the baseURL from hugo.toml.


Fix this error: Failed to find a valid digest in the 'integrity' attribute for resource ... ?

With this in config.yml [adapt it for toml as required] : 

params:
  assets:
    disableFingerprinting: true